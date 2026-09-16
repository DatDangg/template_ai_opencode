# FEATURE_WORKFLOW.md — Maintenance entry point (bug · feature · update)

> Entry point cho **mọi request sau khi project đã tồn tại** (maintenance mode).
> Greenfield build (từ BRIEF/spec tới deploy lần đầu) → `AGENT.md`.
> Luật cứng/route nhanh → `AGENTS.md`. Giá trị project → `.agent/PROJECT_PROFILE.md`.

## 0. Precedence

1. `AGENTS.md` — luôn thắng.
2. `.agent/FEATURE_WORKFLOW.md` (file này).
3. `.agent/PROJECT_PROFILE.md` — giá trị cụ thể (branch, package manager, lệnh check, model).
4. Các file `.agent/*.md` khác (greenfield) — chỉ đọc phần không bị override.
   Mọi file legacy có dòng `Maintenance mode override:` ở đầu → phần bị override KHÔNG áp dụng.

> ⚠️ Nếu file legacy ghi `git push origin main --tags` hoặc ghi `currentLayer` vào state
> → **bỏ qua**. Ở maintenance mode: cấm push thẳng `forbidden_branch`; state dùng `features[]`/`bugs[]`.

---

## 1. Router

| Intent | Route |
|---|---|
| "fix bug", "lỗi", "broken", regression, crash (bug đã biết) | **§2 Bug workflow** → `/bug` |
| "soi/kiểm tra màn", "cảm giác nhiều lỗi nhưng không rõ" | **§2b Bug discovery (sweep)** → `/bug-check` — read-only |
| "thêm/sửa/bỏ/xóa tính năng", đổi behavior | **§3 Change Request workflow** → `/feature` |
| "implement feature" (đã có spec/task) | gọi subagent `builder` theo task file |
| "review", "check", "soát" | gọi subagent `reviewer` — không tự sửa |
| hỏi / điều tra | research-only — không edit tới khi user yêu cầu fix |

Không rõ intent → hỏi 1 câu ngắn. **Không tự phân loại thành "chắc là bug nhỏ, sửa luôn".**

---

## 2. Bug workflow

```
Triage → Reproduce → Root cause → Task → Builder → Reviewer PASS
   → progress.json → History → (commit/push target_branch nếu auto_commit_after_pass)
```

### 2.1 Triage (bắt buộc trước khi sửa)
- Xác định: **màn hình/module**, **bước tái hiện**, **expected vs actual**, **role/vai trò**, **môi trường**.
- Thiếu bất kỳ mục nào → **hỏi ngắn 1 lần** (gom câu hỏi), KHÔNG tự giả định.
- Phân loại mức độ: `blocker` / `high` / `medium` / `low`.

### 2.2 Reproduce
- Dựng lại đúng điều kiện. Nếu không reproduce được → ghi nhận, hỏi thêm, **không sửa mò**.
- Chạy verify command thật từ `.agent/PROJECT_PROFILE.md` (`web_typecheck_command`, `web_lint_command`,
  `api_typecheck_command`, `api_lint_command`, `test_command`; `check_commands` chỉ là alias tổng hợp nếu project đã điền).

### 2.3 Root cause (Iron Law)
- **KHÔNG fix khi chưa có root cause.** Đọc `skills/superpowers/systematic-debugging.md`.
- Ghi root cause + evidence (log/stack/trace) vào bug task.
- ≥3 lần fix fail → nghi ngờ **kiến trúc**, dừng lại, báo human. Không thử fix #4.

### 2.4 Task
- Bug **1 dòng, rõ ràng, không risk** → có thể sửa trực tiếp (vẫn phải update progress + history).
- Còn lại → tạo `tasks/bug-<slug>/phase-<N>-task-<NN>.md` (format §5).

### 2.5 Builder → Reviewer
- Gọi subagent `builder` (hoặc `builder-strong` — xem §7) implement + test.
- Gọi subagent `reviewer` kiểm tra **độc lập** (edit: deny). FAIL → trả lại builder, **không** đóng bug.
- Regression: **phải có test tái hiện bug fail trước fix**, pass sau fix (test-first).
- **Chỉ khi reviewer PASS** mới đi tiếp bước 2.7–2.9. FAIL → không update progress là "done", không commit/push.

### 2.6 Nhánh "đầu vào là danh sách bug"
Áp dụng cả khi input đến từ `/bug-check` hoặc user nói "fix tất cả defect".

1. **KHÔNG gọi Builder ngay**.
2. **Tách mỗi bug thành task riêng** `tasks/bug-<slug>/`.
3. Tóm tắt số lượng defect, severity, root cause nghi ngờ, file cần sửa.
4. Đề xuất thứ tự xử lý (blocker/high trước), xử lý **tuần tự**.
5. Nêu rõ bug nào gộp vì **cùng root cause**; ngoài trường hợp đó không gộp nhiều bug vào 1 diff.
6. **DỪNG hỏi user xác nhận** trước khi gọi Builder.

Chỉ bỏ checkpoint nếu prompt có đúng một trong các cụm: `auto proceed`, `khỏi hỏi lại`,
`tự xử lý hết không cần hỏi`.

### 2.7 Progress (bắt buộc)
- Cập nhật `.context/progress.json` ngay khi bug đổi trạng thái:
  thêm/cập nhật entry trong `bugs[]` (`status: triaged → reproducing → root_caused → fixing → review → done|blocked`),
  set `activeWorkItem`.
- `done` chỉ khi reviewer PASS.

### 2.8 History (bắt buộc)
- Trước khi báo xong: append `docs/history/YYYY-MM.md` (bug, root cause, fix, test, file đổi).

### 2.9 Commit / push (chỉ khi PASS)
- Điều kiện **đủ**: reviewer PASS **và** `progress.json` + history đã cập nhật
  **và** `auto_commit_after_pass: true` trong `.agent/PROJECT_PROFILE.md`.
- Chỉ push tới **`target_branch`** (`.agent/PROJECT_PROFILE.md`). Tuyệt đối không push
  `forbidden_branch`; không `--force`/`-f` (đã chặn ở `opencode.jsonc`).
- Reviewer FAIL / progress-history chưa xong / `auto_commit_after_pass: false` → **không** commit/push.
- Không hardcode tên branch — luôn đọc từ profile.

### 2.10 Nhánh "bug đã biết" = 1 bug
- `/bug` chỉ xử lý **một bug đã biết**. Nếu input là khu vực mơ hồ / danh sách nghi vấn
  → chạy `/bug-check` (§2b) trước, dừng chờ user chọn.

## 2b. Bug discovery (sweep) — `/bug-check`

Chế độ **READ-ONLY** để soi một màn/khu vực mơ hồ, KHÔNG sửa gì.

```
Xác định phạm vi → Đọc code/docs → Liệt kê defect (file:line)
   → ghi tasks/bug-<slug>/scan.md → DỪNG chờ user chọn defect → (user chạy /bug)
```

Quy tắc bắt buộc:
- **KHÔNG** sửa code, **KHÔNG** gọi `builder`/`builder-strong`, **KHÔNG** update `progress.json`,
  **KHÔNG** commit/push.
- Chỉ được tạo/ghi **một file**: `tasks/bug-<slug>/scan.md`.
- Report là bảng defect: `# | Mô tả | Tái hiện | Expected | Actual | Root cause (file:line) | Severity | File cần sửa | Ước lượng`.
- Kết thúc: chạy `git status --short`; nếu có file nào khác `scan.md` biến động → cảnh báo vi phạm read-only.
- Output xong → **dừng**, chờ user chọn defect (mỗi defect xử lý bằng `/bug`).

---

## 3. Change Request workflow (feature / update)

```
Classify → Spec delta → Spec Validator → Phase/Task → Human duyệt plan
   → Loop(builder/reviewer) → Phase Review → History
```

### 3.1 Classify
- **ADDITIVE** (thêm mới) / **MODIFY** (đổi behavior) / **REMOVE** (bỏ).
- Requirement mơ hồ → hỏi lại. Không tự chọn giả định lớn.

### 3.2 Spec delta
- Ghi rõ thay đổi so với `SPECIFICATIONS.md` (thêm/sửa/xóa mục nào, API/DB/UI bị ảnh hưởng).
- Nếu thay đổi behavior/scope → **cập nhật spec** hoặc ghi rõ lý do không cần.
- Liệt kê ảnh hưởng tới phase/task đã có (regression risk).

### 3.3 Spec Validator
- Gọi subagent `spec-validator` (edit: deny) cross-check delta vs spec & docs.
- FAIL → quay lại làm rõ. PASS → chia phase/task.

### 3.4 Phase / Task
- Chia theo **Phase model** (§4), mỗi task: scope, inputs, outputs, acceptance criteria, deps.
- File: `tasks/feature-<slug>/phase-<N>-task-<NN>.md`.

### 3.5 Human duyệt plan
- Trình danh sách phase + task + thứ tự. **Chờ user duyệt** mới code.

### 3.6 Loop
- Mỗi task: builder implement+test → reviewer độc lập. FAIL → trả lại builder (max 2 vòng).
- **Không tự chạy task/phase tiếp theo** khi chưa qua checkpoint (§8).

### 3.7 Phase Review
- Sau khi cả phase PASS: `spec-validator` cross-check "đã build đúng & đủ so với spec delta".
- PASS → phase done → checkpoint. GAP → quay lại bổ sung.

### 3.8 Nhánh "đầu vào là danh sách feature"
- Tách **mỗi feature thành task/feature riêng**, chốt ưu tiên, xử lý **tuần tự**.
- Gộp chỉ khi cùng mục tiêu/scope (1 feature nhiều phase).

### 3.9 Progress & History (bắt buộc)
- Update `.context/progress.json`: thêm/cập nhật entry trong `features[]`, set `activeWorkItem`.
- Append `docs/history/YYYY-MM.md` (feature, spec delta, phase done, file đổi).
- Commit/push: xem §2.9 (chỉ khi PASS + `auto_commit_after_pass: true`, chỉ tới `target_branch`).

---

## 4. Phase model

| Phase | Nội dung | Reviewer tập trung |
|---|---|---|
| 1 | Schema / domain (migration, model, types) | migration versioned, không phá dữ liệu cũ |
| 2 | Backend / API (service, route, validation, auth) | contract, OWASP, BOLA/IDOR, lỗi |
| 3 | UI (component, screen, states, responsive) | a11y, responsive 375/768/1280, craft-floor |
| 4 | Integration (FE↔BE, auth flow, error/loading) | contract thật, không hardcode shape |
| 5 | Test / UAT (unit, integration, e2e, edge) | coverage path xấu, không false-confidence |

> Không nhảy cóc: schema xong mới API, API xong mới UI, integration xong mới UAT.
> Task có thể gộp phase nếu thật nhỏ — nhưng phải ghi rõ.

---

## 5. State & paths

### State — `.context/progress.json` (schema maintenance)
Tối thiểu file phải là:

```json
{
  "mode": "maintenance",
  "activeWorkItem": null,
  "features": [],
  "bugs": []
}
```

Khi có work item, có thể mở rộng trong `features[]` / `bugs[]`:

```json
{
  "mode": "maintenance",
  "activeWorkItem": null,
  "features": [
    { "slug": "", "title": "", "type": "ADDITIVE|MODIFY|REMOVE",
      "status": "planned|in_progress|blocked|done", "currentPhase": 0, "tasks": [] }
  ],
  "bugs": [
    { "slug": "", "title": "", "severity": "blocker|high|medium|low",
      "status": "triaged|reproducing|root_caused|fixing|review|done|blocked",
      "task": "tasks/bug-<slug>/..." }
  ]
}
```
- Tối thiểu: `mode`, `activeWorkItem`, `features`, `bugs`. Có thể thêm `lastUpdated` nếu muốn.
- **KHÔNG** dùng field greenfield (`currentLayer`, `totalLayers`, `completedTasks`, `inProgressTask`, …)
  trong maintenance mode.
- `activeWorkItem` = `{ type, slug }` của bug/feature đang làm, hoặc `null`.
- Cập nhật progress.json là bước **bắt buộc** (§2.7, §3.9).

### Task
- Feature: `tasks/feature-<slug>/phase-<N>-task-<NN>.md`
- Bug: `tasks/bug-<slug>/phase-<N>-task-<NN>.md`

### Review report
- `.context/review-reports/<feature|bug>-<slug>-phase-<N>-<review|layer-review>.md`

### History
- `docs/history/YYYY-MM.md` (append, không ghi đè)

---

## 6. Cổng chặn (gates)

- **Branch**: tạo `feature/<slug>` hoặc `bug/<slug>`; chỉ push **`target_branch`**;
  **cấm push `forbidden_branch`**, cấm `--force`/`-f` (gate ở `opencode.jsonc` → `permission.bash`).
- **Commit/push**: mặc định không. Chỉ khi reviewer PASS + progress/history xong **và**
  `auto_commit_after_pass: true` (`.agent/PROJECT_PROFILE.md`). FAIL → không commit/push.
- **Migration**: chỉ áp dụng khi `db_tool != none` **và** `migration_required: true`;
  versioned + committed; không sửa migration đã apply. `db_tool: none` → bỏ qua gate này.
- **Secrets**: chỉ từ env; không hardcode/commit; không log.
- **UI**: responsive 375/768/1280; a11y; đo contrast; không slop (skills UI).
- **History bắt buộc**: mọi thay đổi code/config/docs/schema → append `docs/history/YYYY-MM.md`.
- **Progress bắt buộc**: mọi thay đổi trạng thái bug/feature → update `.context/progress.json`.
- **Docs không nhúng code tay**: API_SPEC/ERD là overview + pointer tới source of truth/generated.

### Check commands
Lấy từ `.agent/PROJECT_PROFILE.md` → các field `web_typecheck_command`, `web_lint_command`,
`api_typecheck_command`, `api_lint_command`, `test_command`. `check_commands` chỉ là alias tổng hợp
từ các field trên nếu project đã điền.
Ví dụ placeholder (thay bằng lệnh thật khi repo có app code):
```
web typecheck → <configured command or skip, no app configured>
web lint      → <configured command or skip, no app configured>
api typecheck → <configured command or skip, no app configured>
api lint      → <configured command or skip, no app configured>
test          → <configured command or skip, no app configured>
```
Monorepo: filter theo package bị đụng bằng package manager đã cấu hình (vd `<pm> --filter <pkg> ...`).
Nếu chưa cấu hình package manager/test command hoặc chưa có `apps/`, API/web/test → **skip, no app configured**,
không fail workflow và không tự hardcode lệnh.

### Reviewer level (risk-based)
- Reviewer tự chọn `FAST` / `NORMAL` / `STRICT`; mặc định `NORMAL`.
- `FAST` chỉ dùng khi scope rất hẹp, không shared/API/auth/tenant/schema, Builder đã test PASS.
- Bắt buộc `STRICT` nếu có risk đỏ: auth/RBAC/permission; tenant/school/org isolation;
  schema/migration/database; data loss/destructive/bulk update; API contract/DTO/response shape dùng nhiều màn/client;
  shared service/hook/component/API client/cache key/navigation; payment/subscription/entitlement;
  import/export/report; cron/background job/webhook; security/token/session/password/upload/file access;
  root cause chưa rõ; logic quan trọng thiếu test.
- Report bắt buộc có: `Review level`, `Reason`, `Blast radius`, `Verify commands + result`,
  `Findings`, `Verdict PASS/FAIL`.

---

## 7. Model mapping + luật opt-in

### Cách bật model mapping (bắt buộc khi clone template)
1. Khai model từng vai ở `.agent/PROJECT_PROFILE.md` → block `models:`
   (`builder`, `builder_strong`, `reviewer`, `spec_validator`).
2. **Bỏ comment** dòng `model:` trong frontmatter `.opencode/agent/*.md` (hiện đang comment
   `<provider>/<...>` để kế thừa).
3. **Restart opencode** — agent/config **không hot-reload**; chưa restart thì model mới chưa có hiệu lực.
4. Kiểm: `builder ≠ reviewer` (khác họ provider) để lộ blind spot khác nhau; `spec-validator` họ thứ 3 nếu có.

- Nếu **chưa** cấu hình, frontmatter để comment → subagent **kế thừa model chính**
  (builder == reviewer, mất tác dụng tránh bias). Khi cần, chạy lại `0.5.C` rồi restart.
- `reviewer` / `spec-validator` có `edit: deny` → không tự sửa khi đang review.
- Chạy dạng **subagent** → context sạch, không thừa hưởng completion report của builder.

### Luật opt-in `builder-strong`
- **CHỈ dùng khi user yêu cầu rõ.** Không tự chọn theo phán đoán "bài này khó".
- Bị chặn cứng ở `opencode.jsonc` → `permission.task."builder-strong": "ask"`.
- ⚠️ Auto-mode (`--auto` / auto-approve) sẽ tự duyệt `ask` → mất gate. Muốn giữ gate, không bật auto.

---

## 8. Human checkpoints

1. **Trước khi code** feature lớn → duyệt phase/task plan.
2. **Hết mỗi phase** → tóm tắt (task done, test, review) → chờ user.
3. **Trước deploy** → chờ user approve production.
4. **Khi bị block** (≥3 retry / không reproduce / nghi ngờ kiến trúc) → dừng, báo user.
