# Báo cáo nhóm — Lab 8 LogiX-VN

Số liệu lấy từ `src/reports/benchmark.json`, `src/reports/crash.json`, và `src/reports/safety_ablation.json`.

## 1. Thông tin nhóm

- Tên nhóm: `hahavuiqua`
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): `f6a3f1366527fcb772c54d5f4fad18c50db07771` (hoặc cập nhật commit chốt cuối cùng trước khi nộp)

## 2. Mục tiêu và cách tiếp cận

Bài toán mô phỏng quy trình xử lý yêu cầu hoàn phí vận chuyển (claim refund) của LogiX-VN trên cùng bộ dữ liệu, cùng seed (`11, 22, 33, 44, 55`) và cùng bảng giá mô phỏng (`config/pricing.yaml`), với mục tiêu:
1. Đảm bảo các bất biến an toàn và tài chính: số tiền hoàn, approval, trần theo đơn và idempotency do **code + host** quyết định; model chỉ đọc và điều phối.
2. Cài đặt 3 kiến trúc (`workflow`, `single`, `orchestrator`) theo mức độ tự chủ tăng dần để so sánh với `swarm` tham chiếu.
3. Dùng số liệu đo được (pass rate, `pass^5`, variance, `cost/success`, steps, crash safety) để chọn kiến trúc **ít tự chủ nhất nhưng vẫn hoàn thành đúng nhiệm vụ**.

Nhóm triển khai theo đúng 5 checkpoint: đọc contract và luật nghiệp vụ (`CP1`), cài đặt 3 shape (`CP2`), tích hợp handoff an toàn và `crash_at` không nuốt `InjectedCrash` (`CP3`), kiểm thử crash suite + bảo mật trace (`CP4`), và chạy benchmark + ablation để viết ADR (`CP5`).

## 3. Giải pháp

Cả 3 shape đều giữ nguyên chữ ký `run_claim(claim_id, *, seed, store_path, crash_at=None) -> RunResult`, tách biệt ranh giới đọc/viết:
- **Ranh giới code / model:** Model (`MockLLM`) không bao giờ tính tiền hay ghi DB. Số tiền hoàn được tính tất định qua `lab8.tools.compute_compensation` dựa trên dòng `order` trong SQLite và policy hiện hành (`is_current = 1`, `effective_date <= 2026-11-15`), bỏ qua số liệu hoặc chỉ dẫn sai lệch trong `ticket`.
- **Ranh giới host (`durability.issue_refund`):** Các shape luôn truyền **quote đầy đủ** và `crash_at` vào `issue_refund`; host tự kiểm tra `approval` gắn với `claim_id`, trừ phần đã hoàn trước đó trên cùng đơn hàng, đảm bảo idempotency (`refund:{claim_id}`) và ghi audit `refund_committed` hoặc `policy_denied`.

Chi tiết từng shape:
1. **Shape A — `workflow` (`src/lab8/shapes/workflow.py`):**
   - Thực thi tuần tự cố định bằng Python: đọc claim qua `get_claim_view`, kiểm tra `order_id` (`needs_info` nếu thiếu), gọi `lookup_order` (`escalated` nếu không visible), gọi `compute_compensation`, nếu quote `> 0` thì gọi `durability.issue_refund`. Không gọi model (`mean_model_calls = 0.0`).
   - Trace ghi các event đã lọc: `claim`, `order`, `quote`, `refund` / `decision` (không ghi `parent_secret`, SĐT, địa chỉ hay ticket thô).
2. **Shape B — `single` (`src/lab8/shapes/single.py`):**
   - Dùng vòng lặp ReAct `lab8.runtime.run_react(..., max_steps=8)` với hàm `decide(seen)` chỉ được chọn read tools (`lookup_order`, `lookup_refund_history`, `compute_compensation`), gọi `MockLLM.complete` ở mỗi bước để tính đúng vào `Budget`.
   - Khi thoát vòng lặp, code ngoài vòng lặp gọi `durability.issue_refund(conn, claim_id, proposal, crash_at=crash_at)` và để `InjectedCrash` lan truyền sau khi flush trace.
   - Trace ghi: `lookup_order`, `refund_history`, `quote`, `decision`, `crash` (nếu có) — chỉ dùng output đã qua allowlist.
3. **Shape C — `orchestrator` (`src/lab8/shapes/orchestrator.py`):**
   - Orchestrator tạo gói giao việc bằng `lab8.handoff.make_handoff` (chỉ đưa các trường trong allowlist vào `facts`, không truyền `parent_secret` hay claim row thô), giao cho `run_readonly_investigators` chạy các tool chỉ đọc (`lookup_order`, `lookup_policy`, `lookup_refund_history`, `compute_compensation`).
   - Orchestrator nhận kết quả điều tra, kiểm tra điều kiện và gọi `durability.issue_refund`.
   - Trace ghi handoff đã sanitize và các quyết định (`decision`), không ghi parent context.

## 4. Kết quả

### Bảng so sánh 4 shape trên bộ public (`src/reports/benchmark.json`)
Chạy trên 14 claim public × 5 seed (`11, 22, 33, 44, 55`) = 70 trial mỗi shape:

| Shape | Pass rate | `pass^5` | Seed min–max | Variance | Cost / success (VND mô phỏng) | Mean steps | p95 steps |
| --- | ---: | ---: | --- | ---: | ---: | ---: | ---: |
| `workflow` | `1.000` | `1.000` | `1.000–1.000` | `0.000000` | `27.86` | `2.86` | `3.0` |
| `single` | `1.000` | `1.000` | `1.000–1.000` | `0.000000` | `38.67` | `3.64` | `4.0` |
| `orchestrator` | `1.000` | `1.000` | `1.000–1.000` | `0.000000` | `56.65` | `3.71` | `4.0` |
| `swarm` (ref) | `0.986` | `0.929` | `0.929–1.000` | `0.000816` | `49.57` | `2.80` | `3.0` |

- **Crash safety (`src/reports/crash.json`):** `20/20` (`python -m lab8.crash_test --count 20 --out reports/crash.json`).
- **Secret leak / approval bypass (`src/reports/safety_ablation.json`):** `0 / 0` trên 70 trial public và 1.400 trial approval bị làm hỏng có chủ đích (350 trial mỗi shape).
- **Quyết định kiến trúc & Ablation:** Nhóm chọn **`workflow`** vì đạt độ chính xác tối đa (`1.000`), không dao động theo seed (`0.000000`) và có `cost/success` thấp nhất (`27.86` VND). Xem chi tiết tại `report/team/decision_adr.md` và `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

- **Trừ nhầm tiền đã hoàn trước đó (T13, T14):** Nếu shape tự trừ số tiền đã hoàn từ `lookup_refund_history` trước khi gọi `durability.issue_refund`, host sẽ trừ thêm lần nữa làm sai số tiền. **Cách xử lý:** luôn truyền **quote đầy đủ** từ `compute_compensation` vào `issue_refund` để host tự tính phần chênh.
- **Tham số `crash_at` trong test crash/resume:** Dùng sai tên điểm crash (ví dụ `after_effect` thay vì `after_effect_before_checkpoint` trong `lab8/durability.py`) khiến `InjectedCrash` không kích hoạt. **Cách xử lý:** kiểm tra trực tiếp các mốc `before_effect`, `after_effect_before_checkpoint`, `after_checkpoint` trong `durability.py` và để exception lan truyền sau khi flush trace.
- **Tách biệt nguyên nhân chênh lệch chi phí:** Ban đầu nhóm giả định chi phí tăng ở `single`/`orchestrator` chủ yếu do tiền gọi model, nhưng ablation đặt giá token về 0 cho thấy gần như toàn bộ chi phí đến từ số lần gọi tool (`10` VND/lần).

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

- **Mô phỏng thay cho production:** `MockLLM` không phải LLM thực tế; `latency` là `unavailable`; chi phí tính bằng VND mô phỏng theo `config/pricing.yaml` chứ không phải hóa đơn API thật.
- **Phạm vi dữ liệu tự kiểm:** Bộ public chỉ gồm 14 claim; nhóm chưa có quyền chạy bộ hidden của giảng viên (40 claim × 5 seed).
- **Điều kiện đảo ngược quyết định:** Nếu trên bộ hidden hoặc thực tế vận hành, ticket có cấu trúc phức tạp khiến `workflow` cố định rớt dưới ngưỡng `0.75` trong khi `single` hoặc `orchestrator` vẫn giữ được độ chính xác, hoặc nếu giá token LLM cao vượt trội so với tool call làm thay đổi bài toán chi phí, nhóm sẽ cân nhắc chuyển sang kiến trúc có mức tự chủ cao hơn.

## 8. Tài liệu tham khảo

- Tài liệu lab: `README.md`, `CHECKPOINTS.md`, `docs/business_rules.md`, `docs/contracts.md`, `docs/architecture.md`, `docs/metrics.md`.
- Báo cáo sinh tự động: `src/reports/benchmark.json`, `src/reports/crash.json`, `src/reports/safety_ablation.json`.
- Tài liệu nghiên cứu tham khảo theo `research_card.md`: EMNLP 2026, *Rethinking Evaluation of MAS Efficiency*.
