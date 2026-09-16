---
description: Điều tra và sửa bug đã biết; list bug phải qua checkpoint trước khi gọi Builder.
---

Chạy Bug workflow trong `AGENTS.md` và `.agent/FEATURE_WORKFLOW.md` (§2) cho bug:

`$ARGUMENTS`

Nếu đầu vào là khu vực mơ hồ / nhiều nghi vấn (chưa rõ bug nào) → **dừng**, yêu cầu chạy
`/bug-check` trước. `/bug` xử lý một bug đã biết, hoặc list bug đã được user xác nhận.

## Nếu input là list bug hoặc kết quả `/bug-check`

Áp dụng cả khi user nói "fix tất cả defect":
1. **KHÔNG gọi Builder ngay.**
2. Tách từng bug thành task riêng (`tasks/bug-<slug>/...`).
3. Tóm tắt số lượng defect, severity, root cause nghi ngờ và file cần sửa.
4. Đề xuất thứ tự xử lý; nêu rõ bug nào được gộp vì **cùng root cause**.
5. **DỪNG hỏi user xác nhận** trước khi xử lý/gọi Builder.

Chỉ bỏ checkpoint nếu prompt có đúng một trong các cụm: `auto proceed`, `khỏi hỏi lại`,
`tự xử lý hết không cần hỏi`.

Quy tắc bắt buộc:
1. Không sửa code trước khi có root cause.
2. Thiếu info (màn hình / bước tái hiện / expected-actual / role) → hỏi ngắn trước.
3. Tạo/cập nhật task nếu không phải fix 1 dòng (`tasks/bug-<slug>/...`).
4. Builder code + test; Reviewer kiểm tra độc lập (subagent, edit: deny).
5. **Bắt buộc update `.context/progress.json`** (schema maintenance tối thiểu) khi bug đổi trạng thái
   (`bugs[]`, `activeWorkItem`). `done` chỉ khi reviewer PASS.
6. **Bắt buộc update `docs/history/YYYY-MM.md`** trước khi báo xong.
7. **Chỉ commit/push khi Reviewer PASS** + progress/history đã cập nhật.
8. Commit/push **chỉ tới `target_branch`** trong `.agent/PROJECT_PROFILE.md`, và **chỉ khi**
   `auto_commit_after_pass: true`. Cấm push `forbidden_branch`, cấm `--force`/`-f`.
   Reviewer FAIL hoặc `auto_commit_after_pass: false` → **không** commit/push.
9. Nếu có code/config/docs/schema change → update progress/history; nếu chỉ triage/checkpoint chưa sửa gì thì không ghi done.
10. Verify commands lấy từ `.agent/PROJECT_PROFILE.md`; nếu command chưa cấu hình hoặc chưa có app code → ghi `skip, no app configured`, không tự hardcode package manager/test command.
