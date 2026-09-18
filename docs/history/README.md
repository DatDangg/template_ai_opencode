# docs/history/ — Legacy Change Log (optional/generated-only)

> Deprecated for mandatory tracking. Commit-first tracking is the source of truth for changed files,
> timestamp, SHA, and rollback point. Task files hold root cause/repro/residual risk/doc impact;
> `.context/progress.json` holds current status and reviewer report path.
> Keep this folder only as legacy, optional, or generated-only history. Agents must not block close-out
> on creating or editing `YYYY-MM.md`.

## Legacy optional format

```markdown
### YYYY-MM-DD — <feature|bug|chore> · <slug>
- **Type:** feature (ADDITIVE/MODIFY/REMOVE) | bug | chore
- **Task:** tasks/<...>
- **Root cause / Lý do:** ...
- **Thay đổi:** file chính bị đổi
- **Test:** đã thêm/chạy gì, kết quả
- **Review:** reviewer verdict (PASS/FAIL)
- **Ảnh hưởng:** migration? API? breaking change?
- **Tác giả:** <agent/user>
```

## Quy tắc

- Không bắt buộc cập nhật trước khi báo xong task.
- Nếu generator hoặc user vẫn tạo entry legacy, bug nên ghi rõ root cause và migration/breaking change nên nêu rollback.
