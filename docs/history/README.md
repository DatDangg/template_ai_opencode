# docs/history/ — Change Log (append-only)

> Mọi thay đổi **code/config/docs/schema** → append vào `YYYY-MM.md` của tháng hiện tại.
> **Không sửa/xóa entry cũ** — chỉ thêm mới (append-only) để giữ lịch sử.
> Với repo template chưa muốn ghi history thật, giữ `TEMPLATE.md`; khi thay đổi workflow thực tế vẫn nên append entry tháng hiện tại.

## Format mỗi entry

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

- Cập nhật **trước khi báo xong** một task.
- Bug: ghi rõ root cause (không chỉ triệu chứng).
- Breaking change / migration → nêu rõ + rollback.
