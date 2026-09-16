---
description: Read-only soi/kiểm tra một màn hoặc khu vực, liệt kê defect. KHÔNG sửa code.
---

Chạy **Bug discovery (sweep)** — chế độ **READ-ONLY** — trong `AGENTS.md`,
`.agent/FEATURE_WORKFLOW.md` (§ Bug discovery) cho khu vực/màn sau:

`$ARGUMENTS`

> ⚠️ **This command is prompt-enforced read-only**; the final `git status --short` check is
> **mandatory**. If any file other than `scan.md` changed, **stop and report the violation**.
> Không đổi command này thành builder flow — không build, không fix, không sửa code.

Bạn là người **soi lỗi**, KHÔNG phải người sửa lỗi. Tuyệt đối tuân thủ:

1. **KHÔNG sửa code** — không edit bất kỳ file nguồn nào, không refactor, không format.
2. **KHÔNG gọi subagent `builder` / `builder-strong`**.
3. **KHÔNG update** `.context/progress.json`, **KHÔNG commit / push / deploy**.
4. **KHÔNG sửa mò** — chỉ ghi nhận điều quan sát được, đánh dấu rõ chỗ chưa chắc chắn.
5. Chỉ được phép **tạo/ghi đúng một file**: `tasks/bug-<slug>/scan.md`
   (`<slug>` = slug ngắn mô tả khu vực, vd `bug-checkout-flow`).
6. Nếu khu vực mơ hồ tới mức không xác định được phạm vi → hỏi lại 1 lần rồi mới soi.

## Cách soi (chỉ đọc)

- Xác định phạm vi: màn hình / module / route / file liên quan.
- Đọc code + trace luồng; đối chiếu `docs/**`, `SPECIFICATIONS.md`, `.agent/PROJECT_PROFILE.md`.
- Với mỗi nghi vấn: xác định **tái hiện** (điều kiện, bước), **expected vs actual**,
  **root cause kèm `file:line`** (nếu chưa chắc ghi `nghi ngờ` + lý do).
- Ưu tiên chạy check read-only để có bằng chứng (đọc `.agent/PROJECT_PROFILE.md` →
  `web_typecheck_command`, `web_lint_command`, `api_typecheck_command`, `api_lint_command`,
  `test_command`; `check_commands` chỉ là alias tổng hợp nếu project đã điền).
  Không chạy lệnh ghi/xóa/mutate dữ liệu. Nếu project chưa có app code hoặc command chưa cấu hình
  → ghi `skip, no app configured`, không coi là workflow fail.

## Output — bắt buộc ghi vào `tasks/bug-<slug>/scan.md`

```markdown
# Scan: <khu vực> — <YYYY-MM-DD>

## Phạm vi đã soi
- ...

## Defects

| # | Mô tả | Tái hiện | Expected | Actual | Root cause (file:line) | Severity | File cần sửa | Ước lượng |
|---|-------|----------|----------|--------|------------------------|----------|--------------|-----------|
| 1 | ... | ... | ... | ... | `path/file.ts:42` | blocker/high/medium/low | ... | S/M/L |

## Chưa xác minh / cần thêm info
- ...
```

## Kết thúc

1. In bảng defect ở trên cho user.
2. **DỪNG — chờ user chọn defect** muốn xử lý (user sẽ chạy `/bug` cho từng defect).
3. Chạy `git status --short` và tự kiểm: **chỉ `tasks/bug-<slug>/scan.md` được xuất hiện là file mới/thay đổi**.
   Nếu có file khác biến động → **stop and report violation** ngay (bạn đã vi phạm read-only).
