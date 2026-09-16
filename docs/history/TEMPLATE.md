# History Entry Template

> Copy block dưới vào `docs/history/YYYY-MM.md` (tháng hiện tại), **append** — không sửa entry cũ.
> Nếu đang dùng repo như template và chưa muốn ghi history thật, vẫn giữ file template này để agent biết format.

```markdown
### YYYY-MM-DD — <feature|bug|chore> · <slug>
- **Date:** YYYY-MM-DD
- **Type:** feature (<ADDITIVE|MODIFY|REMOVE>) | bug | chore
- **Scope:** <module/màn/khu vực bị ảnh hưởng> — task: `tasks/<...>`
- **Description:** <thay đổi gì, vì sao; bug thì ghi root cause, không chỉ triệu chứng>
- **Migration:** <none | tên migration + versioned?>  (bỏ trống nếu `db_tool: none`)
- **Tests:** <test đã thêm/chạy, kết quả; bug: test tái hiện fail trước fix>
- **Commit SHA:** <sha hoặc "chưa commit">
- **Review:** <reviewer verdict PASS/FAIL> | **Progress:** `.context/progress.json` updated
- **Ảnh hưởng:** migration? API? breaking change? rollback?
- **Tác giả:** <agent/user>
```
