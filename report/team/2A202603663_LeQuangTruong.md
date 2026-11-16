# Báo cáo cá nhân — 2A202603663_LeQuangTruong

## 1. Thông tin thành viên

- Họ tên: Lê Quang Trường
- MSSV: 2A202603663
- Email:26ai.truonglq@vinuni.edu.vn
- GitHub:Letruong238

## 2. Phần việc phụ trách

Vai **ledger + evaluation** (theo gợi ý chia vai nhóm 4 người trong `TEAM.md`):

- Ledger: kiểm chứng phần host đã có sẵn (`durability.py`, `policy.py`) bằng test. Không sửa các file thước đo.
- Evaluation: chạy crash suite và benchmark 4 shape, đọc số liệu, điền `decision_adr.md` và `research_card.md`, đo thêm secret leak / approval bypass và ablation giá.

## 3. Đóng góp cụ thể

- Thiết lập môi trường (Python 3.13, `.venv`), chạy `lab8.smoke` và cổng host `python -m pytest -q`.
- Thêm 7 test mới ở `src/tests/test_ledger_edge_cases.py` cho các trường hợp biên mà test có sẵn chưa phủ:
  - claim_id không tồn tại;
  - claim thiếu order_id;
  - đơn của principal khác (`outside_principal_scope`);
  - approval có số tiền 0;
  - quote bằng 0 dù approval hợp lệ;
  - `approval_id` trỏ tới approval không tồn tại;
  - hoàn tiền bị cắt về số khách đã trả (C-009: approval 500.000, đơn trả 90.000, ledger 90.000).
- Chạy benchmark 4 shape x 14 claim x 5 seed, tạo `src/reports/benchmark.json`.
- Viết `src/scripts/measure_safety_and_ablation.py` và tạo `src/reports/safety_ablation.json`:
  - đếm trực tiếp secret leak, PII leak, approval bypass trên bộ public và trên 5 kiểu approval bị làm hỏng (thu hồi, hết hạn, số tiền 0, mượn của claim khác, thiếu approval);
  - ablation một biến: giá token của model đặt về 0, giữ nguyên mọi thứ khác.
- Điền `report/team/decision_adr.md` và `report/team/research_card.md` (chọn R4). Khi ablation cho thấy giả thuyết ban đầu của nhóm (chênh cost đến từ lời gọi model) là sai, đã sửa lại cả hai file cho khớp số liệu.
- Quyết định kỹ thuật: chọn workflow làm shape nộp (nhóm đã thống nhất). Lý do: cùng pass 1.000 và pass^5 1.000 với single và orchestrator nhưng cost/success thấp nhất (27.86, so với 38.67 và 56.65).
- Commit chính (hash): (điền sau khi commit)

## 4. Bằng chứng

- Cổng host: `python -m pytest -q` ra `69 passed, 248 deselected`. Ma trận student `python -m pytest -q -m student -o addopts=` ra `248 passed`.
- Test: `src/tests/test_ledger_edge_cases.py` (7 test, đều pass).
- Crash suite: `src/reports/crash.json`, `crash_safety` = `20/20` (file do bạn cùng nhóm commit, đã chạy lại `check_submission` xác nhận).
- Benchmark: `src/reports/benchmark.json`, ví dụ run id `R-workflow-C-001-s11`. Workflow: pass 1.000, pass^5 1.000, variance 0, cost/success 27.86. Swarm: pass 0.986, pass^5 0.929, sai C-002 ở seed 55.
- Safety và ablation: `src/reports/safety_ablation.json`.
  - secret_leak, pii_leak, approval_bypass đều bằng 0 ở cả 4 shape (70 trial public mỗi shape và 1400 trial approval bị làm hỏng).
  - Khi giá token về 0, cost/success của single, orchestrator, swarm giảm 5.8%, 1.7%, 4.7%. Phần chênh cost chủ yếu đến từ số lần gọi tool.
- `python -m lab8.check_submission`: phần shapes, reports, ADR, research card đã `[ OK ]`. Còn FAIL ở commit hash và bảng đóng góp `TEAM.md` (sẽ xử lý khi commit).

Giới hạn: chưa đo được trên bộ hidden của giảng viên, nên các con số 0 chỉ đúng trên bộ public và trên các kịch bản tự tạo.

## 5. Công cụ AI đã dùng

- **Claude Code (trợ lý AI trong VS Code):**
  - đọc và giải thích dự án;
  - chạy lệnh cài đặt, test, crash suite, benchmark;
  - viết nháp 7 test, script đo safety/ablation, và nháp `decision_adr.md`, `research_card.md`.
- Cách tự kiểm tra kết quả:
  - tự chạy lại `python -m pytest -q` và `check_submission`;
  - số trong ADR đối chiếu với `benchmark.json` và `safety_ablation.json`;
  - dòng `baseline` của script ablation khớp từng số với `benchmark.json`, dùng làm kiểm tra tính đúng của script;
  - khi số liệu ablation mâu thuẫn với giả thuyết ban đầu thì sửa kết luận, không giữ giả thuyết cũ.
- Phần nguồn trong research card chỉ ghi theo tài liệu của lab, chưa đọc toàn văn paper và chưa có URL.

## 6. Tự đánh giá

- Làm được: phần ledger và evaluation chạy được, có số liệu tái lập bằng lệnh, có test, và các kết luận đều kèm giới hạn.
- Chưa làm được: chưa đo trên bộ hidden; chưa có đóng góp vào mã của ba shape (do các bạn khác viết); nguồn nghiên cứu chưa được đọc toàn văn.
- Cần cải thiện: bổ sung URL nguồn cho research card, điền commit hash sau khi commit, và rà lại bảng đóng góp trong `TEAM.md`.
