# Báo cáo cá nhân — 2A202603627_MaiDangKhuong

## 1. Thông tin thành viên

- Họ tên: Mai Đăng Khương
- MSSV: 2A202603627
- Email: 26ai.khuongmd@vinuin.edu.vn
- GitHub: dangkhuong14

## 2. Phần việc phụ trách

Vai **orchestrator + handoff** (theo gợi ý chia vai nhóm 4 người trong `TEAM.md`):

- Shape C `orchestrator`: viết `run_claim` trong `src/lab8/shapes/orchestrator.py`.
- Handoff: dựng handoff bằng `lab8.handoff.make_handoff`, chỉ cho investigator tool chỉ đọc, trace ghi handoff đã sanitize.
- Thiết lập môi trường ban đầu (CP1) cho nhóm.

## 3. Đóng góp cụ thể

- Viết shape orchestrator (CP2 và CP3):
  - thiếu `order_id` trả `needs_info`; không tìm thấy đơn hoặc sai principal trả `escalated`; quote bằng 0 trả `denied` và không gọi `issue_refund`;
  - ticket luôn là dữ liệu người dùng, không được parse để lấy mã đơn hay số ngày trễ;
  - handoff có `facts={}` và bốn tool chỉ đọc (`lookup_order`, `lookup_policy`, `lookup_refund_history`, `compute_compensation`); investigator không có write registry;
  - chỉ orchestrator gọi `durability.issue_refund`, truyền **quote đầy đủ** để host tự trừ phần đã hoàn trên cùng đơn (T13, T14);
  - truyền `crash_at` vào `issue_refund`, không nuốt `InjectedCrash`. Trước khi ném tiếp, trace được ghi ra để không bị `stale_trace`.
- Thêm 6 test mới ở `src/tests/test_orchestrator_handoff.py`:
  - `make_handoff` bỏ field lạ và từ chối `issue_refund`, `append_audit`;
  - trace của cả bộ claim public đúng contract và không chứa `CANARY_SECRET_`, `parent_secret`, `raw_headers`, `Authorization`;
  - trace ghi handoff đã sanitize, không có tool ghi;
  - claim không tồn tại vẫn có trace hợp lệ và không có dòng ledger;
  - chạy lại cùng claim với seed khác vẫn một refund và một audit;
  - crash thì trace được ghi trước khi exception đi lên, resume xong vẫn một refund và một audit.
- Quyết định kỹ thuật: orchestrator dùng đúng một lần gọi model cho phần soạn chữ (`draft`), không gọi model để quyết số tiền. Tiền do `compute_compensation` tính.
- Commit chính (hash): `7e6ac4b` (set up CP1 + tạo orchestrator và handoff), merge vào main qua PR `dc91356`.

## 4. Bằng chứng

- Test student của shape: `python -m pytest -q -m student -o addopts= -k orchestrator` ra `62 passed`.
- Test bổ sung: `src/tests/test_orchestrator_handoff.py`, 6 test, đều pass.
- Cổng host: `python -m pytest -q` ra `69 passed, 248 deselected`.
- `python -m lab8.check_submission`: `shape orchestrator: format đúng, 12/12 claim public pass ở seed 11`.
- Benchmark (`src/reports/benchmark.json`, run id ví dụ `R-orchestrator-C-001-s11`): orchestrator pass 1.000, pass^5 1.000, cost/success 56.65, đắt hơn workflow (27.86) và single (38.67). Vì vậy nhóm chọn workflow, không chọn orchestrator (xem `decision_adr.md`).

Giới hạn: chưa đo trên bộ hidden của giảng viên. Test secret chỉ quét trace và `RunResult` trên bộ public.

## 5. Công cụ AI đã dùng

- **Claude Code (trợ lý AI trong VS Code):**
  - đọc dự án và hướng dẫn trong docstring của shape;
  - viết nháp `orchestrator.py` và `test_orchestrator_handoff.py`;
  - hướng dẫn tạo `.venv` bằng Python 3.13 qua `uv`, vì máy chỉ có Python 3.15 và PyYAML không cài được.
- Cách tự kiểm tra kết quả:
  - tự chạy lại test student, cổng host và `check_submission`;
  - đối chiếu tên crash point và định dạng trace trong test với `durability.py` và `trace.py`;
  - đọc lại trace thật của từng claim public để chắc không có secret.

## 6. Tự đánh giá

- Làm được: orchestrator qua đủ 62 test student, có test riêng cho handoff, trace và crash, ba điều kiện must-pass không bị vi phạm trên bộ public.
- Chưa làm được: chưa đo trên bộ hidden; shape orchestrator không được chọn nộp vì cost cao hơn workflow nên phần này chủ yếu phục vụ so sánh.
