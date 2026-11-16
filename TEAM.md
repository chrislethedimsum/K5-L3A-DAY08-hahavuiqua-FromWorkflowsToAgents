# Team

## Thông tin nhóm

- Tên nhóm: hahavuiqua  
- Lớp: L3A  
- Repo nhóm: `K5-L3A-DAY08-hahavuiqua-FromWorkflowsToAgents`  
- URL repo, nhánh nộp, commit chốt: [chrislethedimsum/K5-L3A-DAY08-hahavuiqua-FromWorkflowsToAgents](https://github.com/chrislethedimsum/K5-L3A-DAY08-hahavuiqua-FromWorkflowsToAgents/tree/main) 

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| ----: | :---- | :---- | :---- | :---- | :---- |
| 1 | Vũ Đức Minh | 2A202603673 | 26ai.minhvd4@vinuni.edu.vn | ducminh03 | Implement shape workflow |
| 2 | Lê Xuân Phúc | 2A202603655 | 26ai.phuclx@vinuni.edu.vn | chrislethedimsum | Implement single |
| 3 | Mai Đăng Khương | 2A202603627 | 26ai.khuongmd@vinuin.edu.vn | dangkhuong14 | Implement shape orchestration |
| 4 | Lê Quang Trường | 2A202603663 | 26ai.truonglq@vinuni.edu.vn | Letruong238 | Implement Ledger  evaluation  |

Gợi ý chia vai:

- Nhóm 4 người: workflow \+ integration; single; orchestrator \+ handoff; ledger \+ evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| :---- | :---- | :---- | :---- |
| Vũ Đức Minh | Implement shape workflow | 37ffa0d63004d99cc53f00d1dea3adb2210bf8cc; 061784086b74733e1a41ee2cfebf1656c9941e1e | Kiểm tra shape Workflow có bù trừ số tiền hoàn không: đặc biệt C-014 khách hàng nói “đã hoàn phí ship” => chỉ hoàn phần tiền hàng (120k) thay vì cả phí ship (+30 = 150k). `test_workflow_shape.py` (`test_workflow_passes_full_quote_to_host`) |
| Lê Xuân Phúc | Implement single | 2560fa1bcdfc5bceb13927cd3b904443c17b7782 | tests/test\_single\_shape.py (test\_single\_ignores\_ticket\_delay\_at\_seed\_55, test\_single\_crash\_and\_resume\_is\_idempotent) |
| Mai Đăng Khương | Implement shape Orchestration | \#7e6ac4bab07c1ca0aa1ed3f1013fe0180330b444: set up CP1 \+ tạo orchestrator và handoff- \#1  | src/tests/test\_orchestrator\_handoff.py (thêm mới) |
| Lê Quang Trường | Implement Ledger  evaluation  | 2ac96510f99478c7267cb79c1007e0205f3048dd  2ac96510f99478c7267cb79c1007e0205f3048dd | src/tests/test\_ledger\_edge\_cases.py (thêm mới, 7 test)  |

## Cá nhân

Mỗi thành viên có một báo cáo cá nhân:

- [Vũ Đức Minh — 2A202603673](<report/team/2A202603673_Vũ Đức Minh.md>)
- [Lê Xuân Phúc — 2A202603655](report/team/2A202603655_LeXuanPhuc.md)
- [Mai Đăng Khương — 2A202603627](report/team/2A202603627_MaiDangKhuong.md)
- [Lê Quang Trường — 2A202603663](report/team/2A202603663_LeQuangTruong.md)
