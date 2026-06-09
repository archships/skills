# Documentation System

## Directory Model

```text
docs/
├─ INDEX.md              # documentation entry point (required)
├─ plan/                 # delivery workflow (required, owned by this skill)
│  ├─ README.md
│  ├─ analysis/
│  ├─ tasks/
│  └─ backlog.md
├─ {scope}/              # design docs, organized by project's own scoping
│  ├─ README.md
│  └─ {module}/ or {module}.md
└─ issues/               # optional, for tracking inconsistencies and design gaps
   └─ archived/
```

| Directory | Required | Owner | Purpose |
|---|---|---|---|
| `docs/INDEX.md` | yes | project | Documentation entry point. Links to all design doc areas |
| `docs/plan/` | yes | this skill | Delivery plan: analysis, tasks, backlog. See [delivery-plan.md](delivery-plan.md) |
| `docs/{scope}/` | yes | project | Design docs. Scope is defined by the project — package, feature area, service, etc. |
| `docs/issues/` | no | project | Issue tracking when the project uses file-based issues |

`{scope}` is whatever grouping the project uses for its design docs. A monorepo might use `docs/packages/{pkg}/`. A single-package project might use `docs/modules/` or `docs/features/`. The skill does not prescribe the scoping — it only requires that design docs exist and are reachable from `INDEX.md`.

## Scope README Contract

Each scope has a README that answers relationship and ownership questions.

Required coverage:

- Purpose and boundary.
- Relationship to other scopes.
- Main concepts and one-line definitions.
- Ownership: who creates whom, who calls whom, who stores state.
- Lifecycle and state ownership.
- Public API surface or entry points.
- Design decisions that affect multiple modules.
- Module index linking to child docs.
- Scope-wide constraints.

Use diagrams for structure and flow.

## Module Doc Contract

Module docs cover local contracts. Cross-module collaboration belongs in the parent scope README.

| Section | Content |
|---|---|
| Purpose | What the module owns |
| Boundary | What it handles and what it delegates |
| Types | Inputs, outputs, optional/nullable fields, invariants |
| API | Function/method contracts and return types |
| State | State ownership, persistence, mutation path |
| Errors | Typed errors and thrown conditions |
| Extension points | Hooks, adapters, plugin points, injected dependencies |
| Tests | Required contract and error tests |

## Maintenance Rules

Docs are the source of truth for contracts. Code follows docs, not the other way around.

### When to create

- Before implementation. A task's `context` field points to design docs that must exist before development starts.
- When a new scope, module, or capability is introduced.

### When to update

- When a task changes the contract defined in a design doc, the same task updates the doc. Doc changes and code changes are one atomic delivery.
- When verify finds a mismatch between docs and code, the fix updates whichever is wrong — not just the code.

### Task path includes docs

A task's `path` field lists both code paths and doc paths it touches. This ensures overlap detection accounts for doc changes — two tasks modifying the same design doc cannot run in parallel.

### Staleness prevention

- Verify checks docs-code consistency: if code diverges from the design doc referenced in `context`, it is a blocking finding.
- `INDEX.md` must stay current. When a new scope or doc is added, `INDEX.md` is updated in the same task.
- Orphaned docs (docs that describe removed code) are deleted or archived when discovered.

## Single Source of Truth

- Each rule or process lives in exactly one location.
- Cross-module flows go in the parent scope README.
- Local API details go in module docs.
- Implementation tasks go in `docs/plan/tasks/`.
- Non-blocking findings go in `docs/plan/backlog.md` or `docs/issues/`.
- Reference existing docs instead of repeating rules.

## Writing Style

- Direct, concrete statements.
- Tables and ASCII diagrams for relationships.
- Define only terms that reduce ambiguity.
- Stable, searchable headings.
- English for code identifiers, package names, API names, type names, and commit messages.

## Issues

When the project uses `docs/issues/`:

- New issues: `状态：待处理`.
- Fixed issues: `状态：已修复`, then move to `docs/issues/archived/`.
- One issue per file, focused on a single inconsistency, bug, or design gap.
