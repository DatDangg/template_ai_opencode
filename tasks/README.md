# Tasks Directory

## Hai chế độ

| Mode | Đường dẫn | Dùng khi |
|------|-----------|----------|
| **Maintenance** (mặc định) | `tasks/feature-<slug>/phase-<N>-task-<NN>.md`<br>`tasks/bug-<slug>/phase-<N>-task-<NN>.md` | Bug / feature / update sau khi project đã tồn tại |
| **Greenfield** (legacy) | `tasks/layer-<N>/task-<NN>.md` | Build từ đầu qua `AGENT.md` + `.agent/graph.md` |

> Workflow maintenance: `.agent/FEATURE_WORKFLOW.md`. Phase model: 1 Schema/domain ·
> 2 Backend/API · 3 UI · 4 Integration · 5 Test/UAT.

## Cấu trúc (maintenance)

```
tasks/
├── README.md
├── feature-<slug>/
│   ├── phase-1-task-01.md
│   ├── phase-2-task-01.md
│   └── phase-2-task-02.md
└── bug-<slug>/
    ├── scan.md            ← output của /bug-check (READ-ONLY, danh sách defect)
    └── phase-1-task-01.md
```

## Task file format

```markdown
# Task <NN>: <Title>

## Type
feature (<ADDITIVE|MODIFY|REMOVE>) | bug

## Phase
<N>   # 1 Schema/domain · 2 Backend/API · 3 UI · 4 Integration · 5 Test/UAT

## Description
{Mục tiêu rõ ràng, ngắn gọn}

## Root cause (bug only)
{Triệu chứng → nguyên nhân gốc + evidence; KHÔNG fix khi chưa có}

## Dependencies
- task-<XX> (lý do)

## Acceptance Criteria
- [ ] Tiêu chí đo được, testable
- [ ] ...

## DoD (Definition of Done)
- [ ] Code written (chỉ trong scope)
- [ ] Tests added + pass (bug: test tái hiện fail trước fix)
- [ ] Check commands pass (theo `.agent/PROJECT_PROFILE.md`)
- [ ] Reviewer độc lập PASS (`.opencode/agent/reviewer.md`)
- [ ] `docs/history/YYYY-MM.md` đã append

## Files to Create/Modify
- `<path>`

## Notes
{Edge cases, gotchas, giả định đã nêu}
```

## Rules

1. Task sinh bởi **Change Request workflow** hoặc **Bug workflow** (`.agent/FEATURE_WORKFLOW.md`).
2. **Mỗi bug / mỗi feature tách task riêng** — không gộp nhiều bug/feature vào 1 task/diff
   (trừ khi cùng root cause / cùng scope — ghi rõ lý do).
3. Chốt **thứ tự ưu tiên với user**, xử lý **tuần tự**.
4. Task nhỏ, focused (1–3 files). Acceptance criteria **testable**, không mơ hồ.
5. Không nhảy phase: schema → API → UI → integration → UAT.
6. Không sửa tay task sau khi đã chạy — tạo task mới nếu cần.
7. `tasks/bug-<slug>/scan.md` sinh bởi `/bug-check` là **read-only report** — không sửa code,
   không phải task; user chọn defect xong mới tạo task `/bug` cho từng defect.
