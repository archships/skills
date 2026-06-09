# Delivery Plan Workflow

## Plan Directory

```text
docs/plan/
├─ README.md
├─ analysis/
│  └─ {topic}.md
├─ tasks/
│  └─ {area}-{seq}.md
├─ reviews/
│  └─ {id}-{seq}.md
└─ backlog.md
```

## Analysis

Write analysis for planning and orchestration. Do not assign analysis docs directly to implementation agents.

Analysis must cover:

1. **Module decomposition**: package/module scope, inputs, outputs, dependencies. Produce module-level delivery tasks.
2. **Integration enumeration**: enumerate every documented "module A creates/calls/injects module B" relationship. Produce integration tasks that verify each connection with real implementations, not mocks.

Common miss: only analyzing module-internal responsibilities without enumerating creation chains, injection chains, and call chains between modules. Result: every module passes its own tests, but stub boundaries are never broken through.

| Task type | Objective | Verification |
|---|---|---|
| Module task | Implement module X | Module tests cover types, contract, and errors |
| Integration task | Connect A → B, replace stub | Real call path works, integration test covers path |
| E2E task | Validate user-observable flow | Entry-to-output path executes |

Integration task `depends-on` points to the module tasks it connects.

Task boundary = the smallest independently verifiable deliverable. If fixing task A's blocking findings necessarily produces task B's deliverable, they are one task.

## Task File Format

```yaml
id: {area}-{seq}
scope: ...
status: pending | ready | in-progress | done | blocked
depends-on: [task-id, ...]
```

`area` can be a package prefix, module prefix, or capability prefix (e.g. `acp-001`). `scope` identifies which part of the project this task belongs to.

Required sections: `objective`, `context` (design doc paths to read), `path` (files and directories this task touches), `verification`.

```text
pending ── depends-on all done ──► ready ──► in-progress ──► done
                                                │
                                             blocked
```

## Develop Prompt

```text
你的任务文件是 docs/plan/tasks/{id}.md。

阅读任务文件，只完成 objective 中定义的开发目标，不超出任务范围。

context 指向的设计文档是你理解需求的来源。docs/INDEX.md 是文档总索引。

完成后运行 typecheck 和任务要求的测试。
```

For review fixes:

```text
你的任务文件是 docs/plan/tasks/{id}.md。

阅读 docs/plan/reviews/{id}-{seq}.md，修复其中标记为 blocking 的 findings。

完成后运行 typecheck 和任务要求的测试。
```

## Verify Prompt

```text
review 任务 {id} 的开发产出。

任务文件：docs/plan/tasks/{id}.md
开发产出：任务文件中 path 指向的路径
设计文档：任务文件中 context 指向的路径
项目规范：docs/INDEX.md 是文档总索引。

详细阅读源码与设计文档，判断实现是否达到可交付状态，是否与设计文档完全一致。

写入 docs/plan/reviews/{id}-{seq}.md，格式：

1. Findings 列表，每条标注：
   - 严重程度（P1/P2/P3）
   - 是否阻塞交付（blocking / non-blocking）
   - 设计文档位置与代码位置

   blocking = 与设计文档 contract 不一致，或对接路径残留 stub/mock/fake。
   non-blocking = 实现合理但设计文档未提及。

2. 结论：pass 或 blocked
```

## Isolation

All task development happens on isolation branches. Branch sees only its creation-time baseline.

| Rule | Detail |
|---|---|
| Branch naming | `task/{id}` |
| Branch base | Main HEAD at creation time |
| Depends-on | Wait until all predecessors merge to main before creating branch |
| Commit before verify | Verify agent only sees committed code |
| Lifecycle | create → develop → commit → verify → (pass → merge → delete) or (blocked → fix → re-verify) |

## Execution Mode

```text
ready tasks ──► has worktree? ──► no  ──► serialize all
                                  yes ──► path overlap? ──► yes ──► serialize
                                                            no  ──► parallel
```

Without worktree, all tasks execute serially — one working directory cannot sustain concurrent branches.

With worktree, two tasks can run in parallel only when no entry in one task's `path` is a parent, child, or same path as another's. Tasks with `depends-on` chains are inherently serial.

Serial tasks need no extra coordination — each starts from updated main HEAD after its predecessor merges.

For worktree creation, dispatch, and cleanup procedures, see [worktree-guide.md](worktree-guide.md).

## Merge

```text
develop ──► verify ──► pass ──► merge to main ──► full test ──► done
              │
              └─ blocked ──► fix ──► re-verify
```

On pass:

- Merge to main. Resolve conflicts file-by-file, preserving both sides.
- Run full test. If tests fail, revert merge — task must rebase and re-verify.
- Mark task `done`. Append non-blocking findings to `backlog.md`. Delete review file and branch.

On blocked:

- Fix blocking findings on the same branch. Run typecheck and task tests. Re-verify.

When multiple parallel tasks pass simultaneously, merge one at a time. Each subsequent merge may require conflict resolution and re-test.

## Recovery

When resuming after interruption, reconstruct state from task files and git:

| Observed state | Action |
|---|---|
| Task `in-progress`, branch `task/{id}` exists, no review file | Resume develop or re-dispatch develop |
| Task `in-progress`, review file exists with `blocked` | Re-dispatch fix |
| Task `in-progress`, review file exists with `pass`, not merged | Execute merge |
| Task `in-progress`, branch missing | Reset task to `ready`, re-dispatch |
| Task `blocked`, no review file | Reset task to `ready`, re-dispatch |
