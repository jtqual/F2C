---
name: figma-make-type-consolidate
description: "Deduplicate parallel-iteration type drift in a Figma Make export (e.g. TargetCondition with different shapes across EditMode-1/EditMode-2/Frame57793154) into a single shared type."
triggers:
  - figma make type drift
  - consolidate figma make types
  - dedupe targetcondition
  - figma make shared types
  - fix targetcondition drift
  - figma make iteration drift
dependencies: []
version: "0.1.0"
---

# figma-make-type-consolidate

Figma Make emits multiple iteration folders (`Foo/`, `Foo-1/`, `Foo-2-1/`)
as the chat evolves. Shared types like `TargetCondition`, `Event`, or
`Item` often drift between them — one copy adds a `category` field,
another doesn't. Typecheck surfaces these as TS2322 or TS2739.

This skill collapses them to a single source of truth.

## When to use this skill

Use when `pnpm typecheck` output (captured in `CONVERSION_NOTES.md` §2)
includes TS2322/TS2739 errors pointing at type mismatches across
`src/imports/*` siblings, or when the code review flags
"parallel-iteration drift".

## Preconditions

- `pnpm dev` boots and `pnpm typecheck` output is captured.
- Git working tree is clean — this skill makes many small commits.
- If there's no visual/smoke baseline, add one before starting.

## Workflow

### Step 1 — Inventory the drift

```bash
# List sibling iteration folders
ls src/imports | sort

# Find all exported type/interface declarations in src/imports
rg "export (interface|type) " src/imports -n
```

For each type name that appears in more than one folder, record the
locations and the declared shape. A small table in working notes is
enough:

```
TargetCondition:
  src/imports/EditMode-1/EditMode-2-4678.tsx  → { id, operator, value, category }
  src/imports/EditMode-4/EditMode-44-2575.tsx → { id, operator, value, category }
  src/imports/Frame57793154/Frame57793154.tsx → { id, operator, value }
```

### Step 2 — Pick the canonical shape

Rules of thumb, in priority order:

1. The shape with the most fields wins if the extra fields are actually
   used elsewhere in the code (`rg "\.category" src`).
2. Otherwise the shape used in the newest iteration folder wins (folders
   with higher suffixes — `-2-1` > `-1` > bare — were edited later).
3. When in doubt, ask the user. Never silently drop a field.

### Step 3 — Extract to a shared module

Create `src/types/<typename>.ts` (plural if natural):

```ts
export interface TargetCondition {
  id: string;
  operator: Operator;
  value: string;
  category?: Category;   // optional if not all callers set it
}
```

Optional fields are how you merge shapes without breaking callers that
didn't set them. Preserve comments and JSDoc from whichever copy had
them.

### Step 4 — Redirect imports one file at a time

For each sibling folder that declared the type:

1. Delete its local `export interface TargetCondition { ... }`.
2. Replace with `import type { TargetCondition } from '@/types/target-condition';`
3. `pnpm typecheck` immediately. Fix call sites that relied on the
   dropped/renamed fields before moving to the next folder.
4. Commit: `refactor(types): consolidate TargetCondition from EditMode-1`

Do not batch multiple folders into one commit. The diff becomes
unreviewable and a regression is untraceable.

### Step 5 — Verify

```bash
# No more local declarations of the type
rg "export (interface|type) TargetCondition" src

# Only the canonical import is left
rg "TargetCondition" src/imports | rg -v "from '@/types"
```

The second query should return zero lines. If it returns anything,
there's a stray local declaration or a shape-specific use that needs a
narrower type alias.

## Common pitfalls

- **Field dropped, not merged.** If iteration A has `category` and B
  doesn't, the answer is usually `category?: Category` in the shared
  type, not removing it. Check every call site before deleting a field.
- **Name collision with a different concept.** Two folders can have
  `Event` meaning different things (DOM event vs. analytics event).
  Rename one before extracting.
- **Generated JSX inside the type file.** Figma Make sometimes colocates
  a component and its types. Extract the type only; leave the component
  alone.

## Examples

- TS2322 `Property 'category' is missing in type 'TargetCondition' but
  required in type '…/EditMode-1.TargetCondition'` → classic drift; run
  this skill against `TargetCondition`.
- User: "The same interface is declared in three places and they
  disagree." → run this skill.

## See also

- **figma-make-code-review** — surfaces drift in the R-items list.
- **figma-make-refactor** — this skill is pass 1 of that sequence.
