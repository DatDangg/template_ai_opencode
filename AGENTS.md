# AGENTS.md — AI Workflow Router (entry point)

> This is the **always-loaded entry point**. Read it before acting on any request.
> Detailed workflow: greenfield build → `AGENT.md`; maintenance (bug/feature/update) → `.agent/FEATURE_WORKFLOW.md`.
> Project-specific values (branch, package manager, check commands) → `.agent/PROJECT_PROFILE.md`.

## Precedence

`AGENTS.md` **always wins** over every file in `.agent/`. If a legacy workflow file
(`.agent/blackboard.md`, `.agent/rollback.md`, `.agent/graph.md`, …) conflicts with this
file or `.agent/FEATURE_WORKFLOW.md`, follow **this file**. Legacy files carry a
`Maintenance mode override` note at the top — honor it.

## Router — classify intent BEFORE coding

| Intent (user says…) | Route (mandatory) |
|---|---|
| "fix bug", "lỗi", "broken", regression, crash (đã biết rõ bug nào) | **Bug workflow** (§Bug) → `/bug` |
| "soi/kiểm tra màn", "cảm giác nhiều lỗi nhưng không rõ" | **Bug discovery / sweep** → `/bug-check` — READ-ONLY, KHÔNG fix |
| "thêm/sửa/bỏ/xóa tính năng", "change/update feature" | **Change Request workflow** → `.agent/FEATURE_WORKFLOW.md` → `/feature` |
| "implement feature" (spec/task đã có sẵn) | **Builder theo task** → `.opencode/agent/builder` |
| "review", "check", "soát" (một diff/task cụ thể) | **Reviewer** → `.opencode/agent/reviewer` — KHÔNG tự sửa code |
| "thêm skill", "add skill", "tạo skill", "register skill" | **Customize opencode** — tạo/cập nhật runtime skill đúng format (§Local skills) |
| hỏi / điều tra / "tại sao", "how does X work" | **Research-only** — KHÔNG edit nếu user chưa yêu cầu fix |

Không rõ intent → hỏi 1 câu ngắn để phân loại, đừng đoán.

### Phân biệt 3 command

| Command | Dùng khi | Tính chất |
|---|---|---|
| `/bug-check` | Khu vực/màn mơ hồ, "cảm giác nhiều lỗi" | **READ-ONLY** — soi, liệt kê defect vào `tasks/bug-<slug>/scan.md`, **dừng chờ user chọn**. Không sửa, không commit. |
| `/bug` | **Một bug đã biết** hoặc list bug đã xác nhận | Diagnose root cause → task → builder → reviewer → progress → commit/push target branch nếu PASS |
| `/feature` | Thêm/sửa/bỏ tính năng | Classify ADDITIVE/MODIFY/REMOVE → spec delta → phase/task → builder/reviewer/spec-validator → progress |

---

## Bug rules (bắt buộc)

1. **Không sửa triệu chứng trước khi có root cause.** (Iron Law — `skills/superpowers/systematic-debugging.md`)
2. **Thiếu info** (màn hình / bước tái hiện / expected-actual / role/vai trò) → **hỏi ngắn trước**, không tự giả định.
3. Bug **không phải sửa 1 dòng** → tạo task: `tasks/bug-<slug>/phase-<N>-task-<NN>.md`.
4. Fix-loop + `Repro Verification` theo `.agent/FEATURE_WORKFLOW.md` §2 và `/bug`: chỉ `done` khi repro PASS + Reviewer PASS.
5. Retry/Escalation: sau 3 attempt fail → status `architecture_review_needed`, dừng chờ review kiến trúc/refactor.
6. **Builder** code + test; **Reviewer** kiểm tra độc lập (không sửa source; chỉ ghi report scoped).
7. **Cập nhật `.context/progress.json`** (schema maintenance) sau mỗi bước đổi trạng thái bug.
8. **Chỉ commit/push khi Reviewer PASS** + progress đã cập nhật, và **chỉ tới
   `target_branch`** trong `.agent/PROJECT_PROFILE.md`. Reviewer FAIL → không commit/push.
9. **Danh sách bug** hoặc kết quả `/bug-check`, kể cả "fix tất cả defect" → tách từng bug/task,
   tóm tắt số lượng defect, đề xuất thứ tự, nêu bug nào gộp vì cùng root cause, rồi **DỪNG hỏi xác nhận**
   trước khi gọi Builder. Chỉ bỏ checkpoint nếu user ghi rõ `auto proceed`, `khỏi hỏi lại`,
   hoặc `tự xử lý hết không cần hỏi`.

## Feature rules (bắt buộc)

1. **Classify ADDITIVE / MODIFY / REMOVE** trước khi code.
2. Requirement mơ hồ → **hỏi lại**, không tự chọn giả định lớn.
3. Đổi behavior/scope → cập nhật **spec delta** hoặc ghi rõ lý do không cần.
4. Tạo `tasks/feature-<slug>/phase-<N>-task-<NN>.md` khi nhiều bước hoặc có risk.
5. Task phải có `Classification / Risk`, verification summary, và Retry/Escalation theo `/feature` + `.agent/FEATURE_WORKFLOW.md` §3.
6. Sau 3 attempt fail → status `architecture_review_needed`, dừng chờ review kiến trúc/refactor.
7. **Builder** code + test; **Reviewer** độc lập; **Spec Validator** cross-check gap so với spec.
8. **Cập nhật `.context/progress.json`** (schema maintenance).
9. **Danh sách feature** → tách **mỗi feature thành task riêng**, chốt ưu tiên, xử lý **tuần tự**.
   Gộp chỉ khi cùng mục tiêu/scope (1 feature nhiều phase).

### Doc Impact & Reconcile Rules

Sau khi task/bug/phase PASS, xác định doc impact và reconcile **TRƯỚC** khi đóng việc:

| Thay đổi | Doc cập nhật |
|---|---|
| API contract/endpoint/response shape | `docs/API_SPEC.md` |
| Schema/model/enum | `docs/ERD.md` + regen `docs/generated/*` nếu có |
| Kiến trúc/flow/current behavior | `docs/DESIGN.md` (current-state) |
| Giải quyết gap đã ghi | đổi status gap register |
| Không đổi contract/schema/behavior tài liệu hoá | ghi rõ `no doc impact` |

Phân biệt 2 loại doc — **CẤM tự sửa intent docs cho khớp code**:
- **As-built** (`docs/API_SPEC.md`, `docs/ERD.md`, `docs/DESIGN.md` current-state, generated inventory,
  gap register status): reconcile khi code đổi, kèm evidence code.
- **Intent** (`docs/BRD.md`, `docs/PRD.md`, `docs/USER_FLOW.md` target, business rules trong
  `SPECIFICATIONS.md` — nếu file tồn tại): chỉ đổi qua Change Request + user duyệt.

Code ≠ intent → ghi gap vào gap register (nếu có, vd `docs/changes/TECHNICAL_REQUIREMENT_GAPS.md`),
**KHÔNG** hạ cấp intent cho khớp code. Fix code sai rồi sửa doc cho khớp = hợp pháp hoá bug, coi là vi phạm.

---

## Non-negotiables (mọi route)

- **KHÔNG commit / push / deploy / mở PR** trừ khi user yêu cầu rõ **hoặc** `auto_commit_after_pass: true`
  trong `.agent/PROJECT_PROFILE.md` **và** Reviewer đã PASS.
- **Chỉ push tới `target_branch`** (`.agent/PROJECT_PROFILE.md`). **Cấm push `forbidden_branch`**,
  cấm `--force` / `-f`. Gate cứng ở `opencode.jsonc` (`permission.bash`).
- **KHÔNG commit/push khi Reviewer FAIL** hoặc khi progress chưa cập nhật.
- **KHÔNG tự sửa source khi đang review** — reviewer/spec-validator chỉ được ghi report scoped.
- **Check commands lấy từ `.agent/PROJECT_PROFILE.md`** — không hardcode `npm`.
- Nếu repo chưa có app code/API/web/test hoặc command chưa cấu hình → verify ghi `skip, no app configured`,
  không hardcode package manager/test command và không fail workflow vì thiếu app.
- **Migration safety** chỉ áp dụng khi `db_tool != none` / `migration_required: true`
  (`.agent/PROJECT_PROFILE.md`); `db_tool: none` → bỏ qua gate migration.
- Khi migration gate áp dụng: migration phải versioned + committed; không sửa migration đã apply.
  Trước commit inspect migration artifact; destructive/high-risk ops (`DROP`, đổi type, `SET NOT NULL`,
  `UNIQUE/FK` trên data cũ, enum phá hoại, bulk transform/backfill) → gắn `HIGH_RISK_MIGRATION`,
  không promote production, báo rõ destructive op/table/column/data/backfill/rollback/verify staging.
- Cấm `db push`, `migrate reset`, seed/reset, clone data giữa môi trường cho staging/prod.
  Flow: dev → migration versioned → staging deploy → verify → promote đúng migration đã test lên prod.
  `staging_db` phải khác `prod_db`; không sync data staging→prod.
- **Model mạnh (`builder-strong`) chỉ dùng khi user yêu cầu rõ** — không tự chọn theo độ khó.
- Xong việc → không tự chạy phase/task tiếp theo khi chưa qua **human checkpoint**.

## Tool Loop Guard

- Không chạy lặp cùng 1 shell/search/read command y hệt quá 1 lần.
- Không thử cùng 1 giả thuyết quá 2 lần bằng biến thể gần giống.
- Command/search trả empty hoặc non-zero → ghi nhận và chuyển hướng, không retry vô hạn.
- Bash bị permission deny → **DỪNG NGAY**: không retry, không đổi biến thể, không vòng qua pipeline;
  chuyển Grep/Read hoặc ghi `Blocked`.
- Không xác minh được → ghi `Residual risk`/`Blocked`, không lặp tool.

## Local skills

- Runtime skills của template nằm trong `skills/<skill-name>/SKILL.md` và được đăng ký qua `opencode.jsonc` → `skills.paths: ["./skills"]`.
- Khi user yêu cầu **thêm skill**, phải tạo folder `skills/<lowercase-hyphen-name>/SKILL.md` với frontmatter `name` + `description`; `description` phải nêu rõ khi nào auto-trigger bằng keyword cụ thể.
- Nếu skill có nhiều tài liệu chi tiết, giữ chúng trong cùng folder và để `SKILL.md` làm wrapper trỏ tới các file đó.
- Sau mọi thay đổi skill/config/agent/command, nhắc user **restart opencode** vì config không hot-reload.

## Reviewer rules (risk-based)

- Reviewer tự chọn `FAST` / `NORMAL` / `STRICT`; mặc định `NORMAL`.
- `FAST` chỉ khi scope rất hẹp, không shared/API/auth/tenant/schema, Builder đã test PASS.
- `STRICT` bắt buộc khi có risk đỏ: auth/RBAC/permission; tenant/school/org isolation; DB/schema/migration;
  data loss/bulk update; API contract/DTO/response shape dùng nhiều client; shared service/hook/component/API client/cache key/navigation;
  payment/subscription; import/export/report; cron/webhook; security/token/session/password/upload/file access;
  root cause chưa rõ; logic quan trọng thiếu test.
- Report phải có `Review level`, `Reason`, `Blast radius`, `Verify commands + result`, `Findings`, `Verdict PASS/FAIL`.
