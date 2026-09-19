---
description: Onboarding repo thật — auto-detect stack, hỏi branch/DB, ghi .agent/PROJECT_PROFILE.md, sync quyền verify command.
---

Chạy **Project Profile Setup** cho repo hiện tại. Đây là bước chạy **một lần** khi clone template
vào project thật (maintenance mode). Mục tiêu: biến placeholder trong `.agent/PROJECT_PROFILE.md`
thành giá trị thật để `/bug`, `/feature`, builder, reviewer biết chạy lệnh gì và commit/push ở đâu.

`$ARGUMENTS`

## Bước 0 — Auto-detect (không hỏi)

1. Chạy `node scripts/detect-profile.mjs` và đọc JSON trả về.
2. Nếu script không tồn tại, đọc repo thủ công: lockfile, `package.json` scripts, `source_roots`,
   `prisma/schema.prisma` hoặc `drizzle.config.*`.
3. Hiển thị gọn giá trị detect được + `warnings` (nếu có).

## Bước 1 — Hỏi bắt buộc (từng câu một, KHÔNG đoán)

1. **`target_branch`** — branch đích để commit/push (vd `staging`, `develop`). Không được là `main`
   nếu `forbidden_branch: main`.
2. **`forbidden_branch`** — branch cấm push. Mặc định `main`, chỉ cần confirm.
3. **`auto_commit_after_pass`** — `true|false`. Giải thích rõ: commit sau PASS review là **bắt buộc luôn**;
   flag này chỉ quyết định có **tự PUSH** lên `target_branch` sau PASS hay không.

## Bước 2 — Confirm giá trị detect được (user sửa nếu sai)

4. Stack + `package_manager` + `source_roots`.
5. Verify commands: `install`, `lint`, `typecheck`, `test`, `build`. Nếu repo không phải Node hoặc
   `package.json` không có script → hỏi user hoặc để `null` (workflow sẽ ghi `skip, no app configured`).

## Bước 3 — DB (chỉ khi detect/confirm `db_tool != none`)

6. Confirm `db_tool` và `migration_required`.
7. Hỏi `staging_db` và `prod_db` — ghi **tên env var**, KHÔNG ghi secret/connection string.
   Bắt buộc `staging_db != prod_db`; nếu trùng → dừng, báo user (workflow cấm dùng chung/sync data staging↔prod).
8. Hỏi `migration_command` nếu project có lệnh migrate riêng (vd `pnpm prisma migrate deploy`).

## Bước 4 — Ghi `.agent/PROJECT_PROFILE.md`

- Fill đúng các field đã chốt, **giữ nguyên cấu trúc + comment** của file.
- KHÔNG ghi secret/token/connection string; secret chỉ đọc từ env.
- Field không xác định để `null` / giữ placeholder, không bịa.

## Bước 5 — Sync quyền verify command (dry-run trước, ghi sau)

1. Chạy `node scripts/apply-verify-permissions.mjs` (**mặc định dry-run**, chưa ghi).
   Script đọc command trong profile, tự bỏ qua command không an toàn (quote/backslash, wildcard toàn bộ,
   DB-destructive, git-mutating, `--force`) và in danh sách `Bỏ qua` kèm lý do.
2. Xem output:
   - Command đủ an toàn → sẽ được auto-allow.
   - Command bị bỏ qua → **hỏi user**: có muốn tự thêm allow rule không? Nếu có, hướng dẫn sửa tay
     block `# verify-commands:start` … `# verify-commands:end` trong `.opencode/agent/reviewer.md`
     và `.opencode/agent/spec-validator.md`.
3. Chỉ khi user đồng ý với danh sách auto-allow, chạy lại `node scripts/apply-verify-permissions.mjs --write`
   để ghi file.
4. Nếu script exit code khác 0 (thiếu marker `# verify-commands:start/end`) → **dừng**, báo chưa hoàn tất,
   không tự thêm marker vào file agent.
5. Không auto-allow `migration_command` (lệnh migrate) — để user tự thêm nếu thật sự cần.
6. Pattern sinh ra là **exact** (không nối `*`) để tránh `cmd && lệnh phá hoại` đi kèm.

## Bước 6 — Tóm tắt

In ra:
- `target_branch`, `forbidden_branch`, `auto_commit_after_pass`
- package manager + verify commands đã ghi
- `db_tool`/`migration_required` + `staging_db`/`prod_db`
- allow rules đã sync
- nhắc: **quit và restart opencode** vì `.opencode/*` và config không hot-reload.

Không commit/push trong bước này. Nếu `PROJECT_PROFILE.md` còn field quan trọng chưa chốt, ghi rõ
là chưa hoàn tất thay vì báo done.
