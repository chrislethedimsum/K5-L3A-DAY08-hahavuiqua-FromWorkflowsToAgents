# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R4 · Evaluation confounders |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | EMNLP 2026, Rethinking Evaluation of MAS Efficiency (theo tài liệu lab, slide 44 và 59). Loại bằng chứng: bài nghiên cứu về cách đánh giá. URL: nhóm tự bổ sung sau khi kiểm tra nguồn |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Theo mô tả trong docs/bonus_topics.md của lab: mức tăng hiệu quả của multi-agent có thể đến từ cách thiết kế benchmark, nên phải so với baseline đơn giản và giữ cố định runtime, model, tool. Nhóm chưa đọc toàn văn paper |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Kết luận gốc nói về benchmark của paper, không về LogiX-VN. Ở lab này: MockLLM thay model thật, 14 claim public (70 trial mỗi shape), cost mô phỏng, latency unavailable |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Ghi sau khi đã có benchmark.json nên đây không phải dự đoán trước: khi đổi từ workflow sang single hoặc orchestrator, pass rate không đổi (1.000) còn cost/success tăng. Giả thuyết ban đầu của nhóm là phần chênh cost đến từ lời gọi model; ablation giá (xem bên dưới) kiểm tra giả thuyết này |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ cố định: MockLLM, tool đọc, seed 11,22,33,44,55, 14 claim public, bảng giá config/pricing.yaml, evaluator trial_pass, mỗi trial một SQLite mới. Biến thay đổi: shape (workflow, single, orchestrator, swarm). Lưu ý: đổi shape kéo theo đổi số lần gọi model và tool cùng lúc, nên đây là confounder chưa tách riêng được |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: kết luận cost chỉ do topology trong khi thật ra do model_calls. Đo bằng cost/success cạnh mean_model_calls và mean_tool_calls từng shape; pass rate và pass^5 để kiểm độ đúng; variance để kiểm độ ổn định giữa seed |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json` (chạy trong thư mục `src/`)
- Lệnh ablation đã chạy (một biến: giá token của model, đặt về 0 trong bộ nhớ, không sửa config/pricing.yaml): `python -m scripts.measure_safety_and_ablation --out reports/safety_ablation.json` (chạy trong thư mục `src/`, script nằm ở `src/scripts/measure_safety_and_ablation.py`)
- Run id / file report: `src/reports/benchmark.json` (ví dụ run id `R-workflow-C-001-s11`) và `src/reports/safety_ablation.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: **Giả thuyết "chênh cost đến từ lời gọi model" là SAI trên LogiX mô phỏng.** Pass rate bằng nhau (workflow, single, orchestrator đều 1.000, pass^5 1.000). Cost/success baseline: workflow 27.86, single 38.67, orchestrator 56.65, swarm 49.57. Khi đặt giá token về 0 (mọi thứ khác giữ nguyên): workflow 27.86, single 36.43, orchestrator 55.71, swarm 47.25, tức chỉ giảm 0% / 5.8% / 1.7% / 4.7%. Khi đặt giá tool về 0: 0.00 / 2.24 / 0.93 / 2.32, tức gần như toàn bộ cost là tiền tool. Phần chênh cost vì vậy chủ yếu đến từ số lần gọi tool (mean_tool_calls: workflow 2.79, single 3.64, orchestrator 5.57, swarm 4.66, mỗi lần 10 VND). Kết luận này phụ thuộc bảng giá mô phỏng: nếu dùng model thật với giá token cao hơn nhiều thì thứ hạng có thể đổi
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
