# Worktree Guide

Parallel task execution via git worktrees.

## Convention

- Directory: `.worktrees/<task-id>/`
- Branch: `task/<task-id>`
- Base: determine the correct base branch before creating. Check which branch the current delivery targets (main, a feature branch, a release branch, etc.) and verify it is up to date. A stale or wrong base causes the agent to work against an outdated snapshot, and its merge can silently revert other tasks' work.
- Create: `git worktree add .worktrees/<task-id> -b task/<task-id> <base>`
- Remove: `git worktree remove .worktrees/<task-id>`

## Workflow

1. Check execution mode from the delivery plan (path overlap, dependency chain).
2. For parallel tasks, create the worktree before dispatching. Pass the worktree path as the sub-agent's cwd.
3. Verify agent uses the same worktree as its develop agent.
4. Sub-agent works in its cwd as normal — it does not need to know it is in a worktree.
5. After merge, remove the worktree. For abandoned tasks, force remove and delete the branch.

## Rules

- Only the orchestrator creates and removes worktrees. Sub-agents never manage worktrees.
- Sub-agents operate only within their own cwd. Never touch other worktrees or the main working directory.
- Serial tasks use the main working directory — no worktree needed.

## Cleanup

| Trigger | Action |
|---|---|
| Task merged to base | `git worktree remove .worktrees/<task-id>` then `git branch -d task/<task-id>` |
| Task abandoned | `git worktree remove --force .worktrees/<task-id>` then `git branch -D task/<task-id>` |
| Sprint end or session start | Run `git worktree list`, cross-check against task statuses, remove all worktrees whose task is `done` or no longer exists |

Orchestrator is responsible for cleanup. Do not leave worktrees accumulating — clean up immediately after merge or abandonment. On session resume, always run a sweep before creating new worktrees.

## Recovery

See delivery-plan.md Recovery for task-level state→action mapping. For worktree-specific issues:

| Observed state | Action |
|---|---|
| Worktree exists but branch is missing | Force remove worktree, reset task to `ready` |
| Worktree has `.git/worktrees/<id>/locked` | Unlock with `git worktree unlock`, then decide based on task status |
| Worktree exists for a `done` task | Remove worktree and delete branch (merge cleanup was interrupted) |
