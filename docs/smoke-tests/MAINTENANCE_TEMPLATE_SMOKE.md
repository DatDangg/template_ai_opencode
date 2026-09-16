# Maintenance Template Smoke Tests

Checklist này test workflow maintenance khi repo chưa có app code. Không tạo app/demo code thật.

## A. `/bug-check` Smoke

Prompt:

```text
/bug-check giả lập màn Settings. Không có app code; hãy chứng minh không đủ context, tạo scan.md với defect [cần xác nhận], rồi dừng. Không sửa file khác.
```

Expected:
- Tạo duy nhất `tasks/bug-settings-scan/scan.md`.
- Không gọi Builder hoặc `builder-strong`.
- Không sửa app code.
- Chạy `git status --short`; chỉ có `tasks/bug-settings-scan/scan.md` là file mới/thay đổi.
- Nếu file khác đổi, stop/report read-only violation.

## B. `/bug` List Checkpoint Smoke

Prompt:

```text
/bug màn A: lỗi 1; màn B: lỗi 2; màn C: lỗi 3
```

Expected:
- Không gọi Builder ngay.
- Tách 3 bug/task.
- Tóm tắt số lượng defect và đề xuất thứ tự xử lý.
- Hỏi xác nhận trước khi xử lý.

## C. `/bug fix tất cả defect` Checkpoint Smoke

Prompt:

```text
/bug fix tất cả defect trong tasks/bug-settings-scan/scan.md
```

Expected:
- Không gọi Builder ngay.
- Tóm tắt defect trong scan.
- Hỏi xác nhận trước khi xử lý.
- Chỉ auto-run nếu prompt có `auto proceed`, `khỏi hỏi lại`, hoặc `tự xử lý hết không cần hỏi`.

## D. Reviewer Level Smoke

Mock FAST prompt:

```text
Review mock task: đổi text toast "Saved" thành "Settings saved" trong 1 component, Builder test PASS, không đụng shared/API/auth/tenant/schema.
```

Expected:
- Reviewer chọn `FAST`.
- Report có `Review level`, `Reason`, `Blast radius`, `Verify commands + result`.

Mock STRICT prompt:

```text
Review mock task: đổi API response shape dùng bởi nhiều client và thêm tenant filter cho school/org isolation.
```

Expected:
- Reviewer chọn `STRICT`.
- Reason nêu risk đỏ: API contract/shared client/tenant isolation.
- Report có `Review level`, `Reason`, `Blast radius`, `Findings`, `Verdict PASS/FAIL`.

## E. Guard Smoke

Expected:
- `builder-strong` phải ask vì `opencode.jsonc` có `permission.task.builder-strong = ask`.
- `git push origin main` phải deny.
- `git push origin HEAD:refs/heads/main` phải deny.
- `git push --force` và `git push -f` phải deny.
- `git reset --hard` phải deny.
- `git checkout -- <path>` phải deny.

## No-App Verify Rule

Nếu repo chưa có app code/API/web/test hoặc `.agent/PROJECT_PROFILE.md` chưa cấu hình command,
verify result phải ghi `skip, no app configured`; không tự hardcode package manager/test command.

## Restart Rule

Sau khi sửa `.opencode/*`, `.opencode/command/*`, `.opencode/agent/*`, hoặc `opencode.jsonc`,
quit và restart opencode. Commands/agents/config không hot-reload.
