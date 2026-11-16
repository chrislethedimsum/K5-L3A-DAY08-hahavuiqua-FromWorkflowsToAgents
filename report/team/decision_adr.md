# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash:
- Run id dẫn chứng: R-workflow-C-001-s11 (src/reports/benchmark.json)
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000-1.000 / 0.000000 (70/70 trial, 14 claim public x 5 seed)
- Cost per success (mô phỏng, VND): 27.86
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0 (workflow: 70 trial public và 350 trial approval bị làm hỏng; src/reports/safety_ablation.json; chưa đo trên bộ hidden)

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Trên cùng 14 claim public, cùng 5 seed (11,22,33,44,55), cùng bảng giá và cùng evaluator (`src/reports/benchmark.json`), ba shape của nhóm đều đạt pass rate 1.000 và pass^5 1.000, variance 0. Swarm đạt 0.986 / 0.929 vì sai C-002 ở seed 55. Vì độ đúng ngang nhau nên điểm phân biệt là chi phí mô phỏng: cost/success của workflow là 27.86, single 38.67, orchestrator 56.65, swarm 49.57. Workflow rẻ nhất và ít bước nhất trong ba shape của nhóm (mean steps 2.86), nên nhóm chọn kiến trúc ít tự chủ nhất mà vẫn làm đủ việc. Ablation giá (`src/reports/safety_ablation.json`) cho thấy phần chênh cost chủ yếu đến từ số lần gọi tool (10 VND mỗi lần): đặt giá model về 0 chỉ giảm cost/success của single, orchestrator, swarm lần lượt 36.43, 55.71, 47.25 (giảm 1.7-5.8%), còn workflow giữ 27.86 vì không gọi model. Xem `research_card.md`.

## Có đáng đổi lấy multi-agent không

Trên bộ public không có shape nào pass nhỉnh hơn workflow, nên không có lý do trả thêm cost cho single, orchestrator hay swarm. Swarm còn đắt hơn workflow (49.57 so với 27.86) và có variance khác 0 (0.000816). Quyết định sẽ đảo lại nếu trên bộ hidden (40 claim x 5 seed) workflow rớt dưới ngưỡng 0.75 trong khi single hoặc orchestrator vẫn đạt, hoặc nếu ticket thực tế mơ hồ đến mức luật cố định không đọc được và cần model hiểu ngôn ngữ. Khi đó mức tự chủ cao hơn mới đáng trả giá.

## Trace và bằng chứng

Trace của nhóm ghi những event nào? Field nào cố ý **không** ghi, và vì sao? Nếu trace nằm ngoài chỗ scanner quét được, đội security chứng minh thế nào là hệ thống không lộ secret?

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.

Bộ public chỉ có 14 claim nên pass 1.000 chưa chứng minh được kết quả trên bộ hidden (ticket nói dối, injection, homoglyph, approval mượn của claim khác, ...). Cost là chi phí mô phỏng, không phải hóa đơn API. Số secret leak và approval bypass ở trên chỉ đếm trên bộ public (70 trial mỗi shape) và trên 1400 trial có approval bị làm hỏng có chủ đích (thu hồi, hết hạn, số tiền 0, mượn của claim khác, thiếu approval), đều ra 0 (`src/reports/safety_ablation.json`). Nhóm không có quyền chạy bộ hidden của giảng viên nên chưa đo trên bộ đó.
