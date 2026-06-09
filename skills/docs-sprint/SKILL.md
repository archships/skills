---
name: docs-sprint
description: Docs-driven delivery workflow. Use when splitting design docs into tasks, creating implementation plans under docs/plan, executing develop/verify/merge cycles, maintaining design doc consistency, or writing UI layout documentation with ASCII diagrams.
---

# Docs Sprint

Docs-driven delivery workflow — given design docs, manage the process from task split to verified merge. Agent-agnostic. Platform-specific agent configs live under `agents/`.

## Role

You are the orchestrator. You plan, dispatch, monitor, and merge. You do NOT write code or run tests yourself. All implementation and verification work must be delegated to sub-agents.

| You do | You delegate |
|---|---|
| Read and update design docs | Code implementation (develop agent) |
| Write analysis and split tasks | Code review and verification (verify agent) |
| Check task status and dispatch | Fix blocking findings (develop agent) |
| Execute merge and run full test | |
| Manage recovery from interruption | |

## Parallel Execution

You have worktree capability. When the execution mode allows parallel dispatch, create worktrees for sub-agents following [worktree-guide.md](references/worktree-guide.md). Create the worktree before dispatching; pass the worktree path as the sub-agent's working directory.

## Core Workflow

1. Read `docs/INDEX.md`, then read the design docs related to the request.
2. For planning work, read `docs/plan/README.md` and use the plan/task/review loop.
3. For UI work, read the relevant layout docs and represent interface changes with ASCII diagrams.
4. If implementation needs behavior missing from docs, update the design docs first and ask for confirmation before changing code.
5. Dispatch develop agents for implementation. Dispatch verify agents for review. Do not do both in the same agent.
6. Treat review findings as delivery gates: fix blocking findings via develop agent, archive or record non-blocking findings according to project docs.

## Task Classification

| Request type | Action |
|---|---|
| Architecture/design docs | Read [documentation-system.md](references/documentation-system.md), then update design docs |
| Development plan or task split | Read [delivery-plan.md](references/delivery-plan.md), then create analysis/tasks/backlog entries |
| Code implementation | Read task file and referenced design docs; follow [contract-discipline.md](references/contract-discipline.md) |
| Review or fix review findings | Use the verify/develop loop in [delivery-plan.md](references/delivery-plan.md) |
| UI layout docs | Read [ui-layout.md](references/ui-layout.md), then write layout definition docs with ASCII diagrams |

## Documentation Rules

- Keep one source of truth for each rule or process.
- Use scope README files for ownership, relationships, state ownership, lifecycle, and cross-module flows.
- Use module docs for local contract: inputs, outputs, state boundary, errors, extension points, and tests.
- Prefer diagrams, tables, and explicit flow charts over broad prose.
- Define only necessary terms. Keep terms concrete and traceable to code or user-visible behavior.
- When docs and implementation need different behavior, update docs first.
- Doc changes and code changes in the same task are one atomic delivery.

## Implementation Rules

- Follow [contract-discipline.md](references/contract-discipline.md) for type, error, and fallback rules.
- Tests must cover contract behavior and error handling for the touched boundary.

## UI Layout Rules

- Show UI structure with ASCII diagrams.
- Write each layout doc around one page or component.
- Use this section order when it fits: overall structure, page-level layout, region layout, interactions, state variants, responsive behavior, size reference, component tree, page entry points.
- Include input behavior, dialog/overlay layering, and loading/empty/error states.

## References

- `references/documentation-system.md`: docs directory structure, maintenance rules, writing style.
- `references/delivery-plan.md`: analysis, tasks, develop/verify review loop, isolation, merge rules.
- `references/contract-discipline.md`: type, error, fallback, and implementation discipline.
- `references/ui-layout.md`: UI layout documentation style, ASCII diagrams, interaction contracts.
- `references/worktree-guide.md`: worktree creation, dispatch, cleanup, and recovery.
