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
| hỏi / điều tra / "tại sao", "how does X work" | **Research-only** — KHÔNG edit nếu user chưa yêu cầu fix |

Không rõ intent → hỏi 1 câu ngắn để phân loại, đừng đoán.

### Phân biệt 3 command

| Command | Dùng khi | Tính chất |
|---|---|---|
| `/bug-check` | Khu vực/màn mơ hồ, "cảm giác nhiều lỗi" | **READ-ONLY** — soi, liệt kê defect vào `tasks/bug-<slug>/scan.md`, **dừng chờ user chọn**. Không sửa, không commit. |
| `/bug` | **Một bug đã biết** hoặc list bug đã xác nhận | Diagnose root cause → task → builder → reviewer → progress/history → commit/push target branch nếu PASS |
| `/feature` | Thêm/sửa/bỏ tính năng | Classify ADDITIVE/MODIFY/REMOVE → spec delta → phase/task → builder/reviewer/spec-validator → progress/history |

---

## Bug rules (bắt buộc)

1. **Không sửa triệu chứng trước khi có root cause.** (Iron Law — `skills/superpowers/systematic-debugging.md`)
2. **Thiếu info** (màn hình / bước tái hiện / expected-actual / role/vai trò) → **hỏi ngắn trước**, không tự giả định.
3. Bug **không phải sửa 1 dòng** → tạo task: `tasks/bug-<slug>/phase-<N>-task-<NN>.md`.
4. **Builder** code + test; **Reviewer** kiểm tra độc lập (subagent, `edit: deny`).
5. **Cập nhật `.context/progress.json`** (schema maintenance) sau mỗi bước đổi trạng thái bug.
6. **Cập nhật `docs/history/YYYY-MM.md`** trước khi báo xong.
7. **Chỉ commit/push khi Reviewer PASS** + progress/history đã cập nhật, và **chỉ tới
   `target_branch`** trong `.agent/PROJECT_PROFILE.md`. Reviewer FAIL → không commit/push.
8. **Danh sách bug** hoặc kết quả `/bug-check`, kể cả "fix tất cả defect" → tách từng bug/task,
   tóm tắt số lượng defect, đề xuất thứ tự, nêu bug nào gộp vì cùng root cause, rồi **DỪNG hỏi xác nhận**
   trước khi gọi Builder. Chỉ bỏ checkpoint nếu user ghi rõ `auto proceed`, `khỏi hỏi lại`,
   hoặc `tự xử lý hết không cần hỏi`.

## Feature rules (bắt buộc)

1. **Classify ADDITIVE / MODIFY / REMOVE** trước khi code.
2. Requirement mơ hồ → **hỏi lại**, không tự chọn giả định lớn.
3. Đổi behavior/scope → cập nhật **spec delta** hoặc ghi rõ lý do không cần.
4. Tạo `tasks/feature-<slug>/phase-<N>-task-<NN>.md` khi nhiều bước hoặc có risk.
5. **Builder** code + test; **Reviewer** độc lập; **Spec Validator** cross-check gap so với spec.
6. **Cập nhật `.context/progress.json`** (schema maintenance) + `docs/history/YYYY-MM.md`.
7. **Danh sách feature** → tách **mỗi feature thành task riêng**, chốt ưu tiên, xử lý **tuần tự**.
   Gộp chỉ khi cùng mục tiêu/scope (1 feature nhiều phase).

---

## Non-negotiables (mọi route)

- **KHÔNG commit / push / deploy / mở PR** trừ khi user yêu cầu rõ **hoặc** `auto_commit_after_pass: true`
  trong `.agent/PROJECT_PROFILE.md` **và** Reviewer đã PASS.
- **Chỉ push tới `target_branch`** (`.agent/PROJECT_PROFILE.md`). **Cấm push `forbidden_branch`**,
  cấm `--force` / `-f`. Gate cứng ở `opencode.jsonc` (`permission.bash`).
- **KHÔNG commit/push khi Reviewer FAIL** hoặc khi progress/history chưa cập nhật.
- **KHÔNG tự sửa khi đang review** — reviewer/spec-validator có `edit: deny`.
- **Check commands lấy từ `.agent/PROJECT_PROFILE.md`** — không hardcode `npm`.
- Nếu repo chưa có app code/API/web/test hoặc command chưa cấu hình → verify ghi `skip, no app configured`,
  không hardcode package manager/test command và không fail workflow vì thiếu app.
- Mọi thay đổi code/config/docs/schema → **append `docs/history/YYYY-MM.md`**.
- **Migration safety** chỉ áp dụng khi `db_tool != none` / `migration_required: true`
  (`.agent/PROJECT_PROFILE.md`); `db_tool: none` → bỏ qua gate migration.
- **Model mạnh (`builder-strong`) chỉ dùng khi user yêu cầu rõ** — không tự chọn theo độ khó.
- Xong việc → không tự chạy phase/task tiếp theo khi chưa qua **human checkpoint**.

## Reviewer rules (risk-based)

- Reviewer tự chọn `FAST` / `NORMAL` / `STRICT`; mặc định `NORMAL`.
- `FAST` chỉ khi scope rất hẹp, không shared/API/auth/tenant/schema, Builder đã test PASS.
- `STRICT` bắt buộc khi có risk đỏ: auth/RBAC/permission; tenant/school/org isolation; DB/schema/migration;
  data loss/bulk update; API contract/DTO/response shape dùng nhiều client; shared service/hook/component/API client/cache key/navigation;
  payment/subscription; import/export/report; cron/webhook; security/token/session/password/upload/file access;
  root cause chưa rõ; logic quan trọng thiếu test.
- Report phải có `Review level`, `Reason`, `Blast radius`, `Verify commands + result`, `Findings`, `Verdict PASS/FAIL`.
