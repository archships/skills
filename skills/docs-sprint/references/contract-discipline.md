# Contract Discipline

Design docs are the source of truth for contracts. When code and docs disagree, update docs and confirm before changing code.

## Type And Error Rules

| Situation | Required behavior |
|---|---|
| required data missing | fail fast with explicit typed error |
| unknown variant | fail fast with explicit typed error |
| invalid input | fail fast with explicit typed error |
| contract mismatch | fail fast with explicit typed error |
| optional lookup missing | return absent value (`undefined` in TS, `None` in Python, etc.) |
| fallback/default value | allowed only when design docs explicitly authorize |
| empty container / zero / noop fallback | allowed only when design docs explicitly authorize |
| silent recovery / best-effort normalize | allowed only when design docs explicitly authorize |

Do not use empty containers, zero values, or no-op callbacks to mask missing required state.

## Test Expectations

| Change type | Verification |
|---|---|
| type / interface | compile or typecheck + contract tests |
| error behavior | tests for typed errors and missing optional lookups |
| storage / state | tests for persistence, versioning, and mutation path |
| integration | tests using real implementations across boundary |
| UI / TUI | layout docs + interaction tests or snapshots when implementation exists |

Document any verification that cannot run and why.
