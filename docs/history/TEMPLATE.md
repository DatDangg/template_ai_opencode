# Legacy History Entry Template (optional/generated-only)

> Deprecated for mandatory tracking. Commit-first tracking replaces manual history entries.
> Keep this template only for legacy, optional, or generated-only change logs; agents must not use
> `docs/history/YYYY-MM.md` as task close-out evidence.

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
