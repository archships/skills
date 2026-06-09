# UI Layout Documentation

## Scope

Use this reference when documenting UI layouts and interactions — web, mobile, desktop, or terminal.

UI layout docs describe user-visible contracts: what regions exist, how they are arranged, how users reach each function, how input behaves, and how loading/empty/error states display.

## Layout Doc Template

Use this structure when it fits. Shorter files may omit empty sections.

```markdown
# {Feature} Layout

Status: Draft

## Design Principles
## Overall Structure
## Page-Level Layout
## Region Layout
## Interactions
## State Variants
## Responsive / Size Constraints
## Component Tree
## Page Entry Points
```

Each layout file focuses on one page, shell, dialog family, or component group.

## Visual Layout Rules

Use ASCII art or diagrams for every meaningful layout.

```text
┌──────────────────────────────────────┐
│ Header                               │
├──────────────────────────────────────┤
│ Main content area                    │
├──────────────────────────────────────┤
│ Footer / status bar                  │
└──────────────────────────────────────┘
```

Include:

- Region names.
- Conditional regions (shown/hidden by state).
- Dialog and overlay layering.
- Focus target when relevant.
- Responsive behavior for size constraints.

## Interaction Contract

Document input behavior in tables:

| Input | Scope | Behavior |
|---|---|---|
| `Enter` | Input area | submit |
| `Esc` | Dialog | close |
| `Tab` | Form | next field |

For dialogs, document: open/close trigger, focus movement, selection behavior, empty state, error state, apply/cancel behavior.

## State Variants

Document these states for each region when applicable:

- **Loading**: what the user sees while data is being fetched.
- **Empty**: what the user sees when there is no content.
- **Error**: what the user sees on failure, and how to recover.
- **Busy**: what the user sees during a long-running operation.
- **Constrained**: how the layout adapts to limited screen/window size.
