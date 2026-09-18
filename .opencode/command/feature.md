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
   Task phải có `Classification / Risk`: change type, scope, expected review level, blast radius,
   doc impact, decision impact.
5. Trình phase plan → chờ user duyệt mới code.
6. Mỗi task: Builder code + test → Reviewer độc lập; hết phase → Spec Validator cross-check gap.
   Nếu test/check fail, acceptance criteria chưa đạt, hoặc Reviewer/Spec Validator FAIL → **chưa xong**;
   quay lại Builder cập nhật trong cùng task cho tới khi PASS hoặc ghi `BLOCKED` với lý do thật.
7. Mỗi task/phase bắt buộc có verification summary:
   - Acceptance criteria: PASS | FAIL | BLOCKED
   - Verify commands: `<configured command>` hoặc `skip, no app configured`
   - Reviewer verdict: PASS | FAIL
   - Spec Validator verdict (khi hết phase): PASS | FAIL
8. **Không set task/feature/phase `done`** nếu verify/test/review/spec status là `FAIL`, `BLOCKED`, hoặc unknown.
9. Retry / Escalation Policy:
   - Attempt 1 fail: gọi/áp dụng Error Analyzer, xác định lại root cause, fix tối thiểu.
   - Attempt 2 fail: dừng patch triệu chứng; so với pattern code đang hoạt động và kiểm tra lại assumption.
   - Attempt 3 fail: **KHÔNG thử fix #4**. Set `BLOCKED` / `ARCHITECTURE_REVIEW_NEEDED`.
     Tạo Structural Review: data flow, ownership/scope boundary, API contract, permission/tenant/school filters,
     state/cache layer, mock/real data boundary, schema/domain mismatch.
   - Sau Structural Review: hỏi human hoặc tạo task refactor/design riêng trước khi sửa tiếp.
10. **Bắt buộc update `.context/progress.json`** (schema maintenance: `features[]`, `activeWorkItem`).
11. **Chỉ commit/push khi Reviewer PASS** + progress đã cập nhật, **chỉ tới `target_branch`**
   trong `.agent/PROJECT_PROFILE.md`, và **chỉ khi** `auto_commit_after_pass: true`.
   Cấm push `forbidden_branch`, cấm `--force`/`-f`.
12. Mỗi failed attempt phải append `.context/error-memory.md` hoặc ghi rõ vì sao không có entry.
13. Nếu update làm đổi kiến trúc/ownership/scope boundary/API contract/mock-real boundary → append `.context/decisions.md`.
14. Verify commands lấy từ `.agent/PROJECT_PROFILE.md`; nếu command chưa cấu hình hoặc chưa có app code → ghi `skip, no app configured`, không tự hardcode package manager/test command.
15. Trước khi báo xong/đóng task/phase phải chạy **Doc Impact & Reconcile** trong `AGENTS.md` + `.agent/FEATURE_WORKFLOW.md`:
    reconcile as-built docs nếu code đổi hoặc ghi rõ `no doc impact`. **Không** sửa intent docs để khớp code;
    code ≠ intent thì ghi gap register nếu có.
16. Nếu đầu vào là danh sách feature → tách mỗi feature thành task riêng, chốt ưu tiên, xử lý tuần tự.
