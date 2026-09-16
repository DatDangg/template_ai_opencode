---
description: Thêm/sửa/bỏ tính năng theo Change Request workflow (classify → spec delta → phase/task → build/review/validate).
---

Chạy Change Request workflow trong `AGENTS.md` và `.agent/FEATURE_WORKFLOW.md` (§3) cho:

`$ARGUMENTS`

Quy tắc bắt buộc:
1. Classify ADDITIVE / MODIFY / REMOVE trước khi code.
2. Requirement mơ hồ → hỏi lại, không tự chọn giả định lớn.
3. Cập nhật spec delta (hoặc ghi rõ lý do không cần).
4. Chia phase/task (`tasks/feature-<slug>/phase-<N>-task-<NN>.md`) khi nhiều bước hoặc có risk.
5. Trình phase plan → chờ user duyệt mới code.
6. Mỗi task: Builder code + test → Reviewer độc lập; hết phase → Spec Validator cross-check gap.
7. **Bắt buộc update `.context/progress.json`** (schema maintenance: `features[]`, `activeWorkItem`).
8. **Bắt buộc update `docs/history/YYYY-MM.md`** trước khi báo xong.
9. **Chỉ commit/push khi Reviewer PASS** + progress/history đã cập nhật, **chỉ tới `target_branch`**
   trong `.agent/PROJECT_PROFILE.md`, và **chỉ khi** `auto_commit_after_pass: true`.
   Cấm push `forbidden_branch`, cấm `--force`/`-f`.
10. Verify commands lấy từ `.agent/PROJECT_PROFILE.md`; nếu command chưa cấu hình hoặc chưa có app code → ghi `skip, no app configured`, không tự hardcode package manager/test command.
11. Nếu đầu vào là danh sách feature → tách mỗi feature thành task riêng, chốt ưu tiên, xử lý tuần tự.
