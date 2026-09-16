# PROJECT_PROFILE.md — Parameterize the workflow

> **Điền file này khi clone template.** Mọi workflow/subagent đọc giá trị ở đây,
> KHÔNG hardcode branch / package manager / DB / lệnh check trong generic docs.
> Nếu file còn placeholder (`<...>`) → coi như chưa cấu hình, phải hỏi user.

## Profile

```yaml
project: <tên dự án>
output_language: vi            # vi | en — ngôn ngữ cho docs/summary

# ── Git ──
target_branch: <target_branch> # branch đích để commit/push sau khi PASS review; chưa điền = hỏi user
forbidden_branch: main         # cấm push trực tiếp (opencode.jsonc hard-deny main/ref main)
branch_pattern: "feature/<slug>|bug/<slug>"
auto_commit_after_pass: false  # true = tự commit/push target_branch sau khi reviewer PASS + progress/history xong

# ── Package / source ──
package_manager: <none|pnpm|npm|yarn|bun>  # none/chưa điền = không hardcode lệnh
source_roots: []               # ví dụ: [src] hoặc [apps, packages]; [] nếu chưa có app code

# ── Verify commands (null/placeholder = skip, no app configured) ──
web_typecheck_command: null
web_lint_command: null
api_typecheck_command: null
api_lint_command: null
test_command: null
install_command: null
lint_command: null          # generic alias if web/api split does not apply
typecheck_command: null     # generic alias if web/api split does not apply
build_command: null
migration_command: null     # only used when db_tool != none and migration_required: true

# ── Database ──
db_tool: none                  # none | prisma | drizzle | other
migration_required: false      # true nếu cần migration versioned (chỉ khi db_tool != none)
staging_db: <tên DB staging>
prod_db: <tên DB production>
```

### DB / migration

- **`db_tool: none`** → project không dùng DB/ORM: **bỏ qua toàn bộ migration safety rules**
  (Phase 1 schema, migration gate ở `.agent/FEATURE_WORKFLOW.md` §6, ERD/migration checks).
- `db_tool != none` **và** `migration_required: true` → áp dụng migration gate:
  migration phải versioned + committed, không sửa migration đã apply.
- Không hardcode Prisma/Drizzle: dùng đúng `db_tool` đã khai.

## Check commands (chạy trước khi báo xong)

> Mọi người (builder/reviewer) PHẢI dùng đúng lệnh đã cấu hình ở profile, filter theo package bị đụng nếu monorepo.
> Nếu command là `null`/placeholder hoặc repo chưa có app code (`source_roots: []`) → ghi `skip, no app configured`.
> KHÔNG tự suy ra `npm test`, `pnpm lint`, Prisma, package name, hay path `apps/` khi chưa cấu hình.

```yaml
check_commands:
  install: null             # alias of install_command
  web_typecheck: null       # hoặc = web_typecheck_command
  web_lint: null            # hoặc = web_lint_command
  api_typecheck: null       # hoặc = api_typecheck_command
  api_lint: null            # hoặc = api_lint_command
  test: null                # hoặc = test_command
  build: null               # hoặc = build_command
  migration: null           # hoặc = migration_command khi migration_required=true
  docs_inventory: null
```

## Models per role

> Model KHÔNG còn đọc từ `.env.local` (biến đó không có tác dụng). Khai ở đây rồi
> **bỏ comment + copy sang frontmatter** của từng file `.opencode/agent/*.md`,
> rồi **restart opencode** (config không hot-reload).
> Quy tắc: **builder ≠ reviewer** (khác họ provider) để lộ blind spot khác nhau.

```yaml
models:
  builder:        <provider>/<model-code-chinh>
  builder_strong: <provider>/<model-manh-hon>     # chỉ dùng khi user yêu cầu rõ (§7 gate)
  reviewer:       <provider>/<model-khac-ho>
  spec_validator: <provider>/<model-ho-thu-3>     # họ thứ 3 nếu có
```

## UI rules (nếu project có UI)

```yaml
ui:
  responsive_breakpoints: [375, 768, 1280]
  max_file_lines: 300
  max_function_lines: 50
```

## Secrets

```yaml
secrets:
  source: env                    # chỉ đọc từ env; KHÔNG hardcode/commit
  required: [DATABASE_URL, JWT_SECRET]
```
