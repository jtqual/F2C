---
name: figma-make-refactor
description: "Sequence the first refactor pass on a ported Figma Make project: gate on preconditions, then drive layout, type consolidation, file renames, and a11y in the right order."
triggers:
  - figma make refactor
  - refactor figma make
  - make refactor pass
  - clean up figma make
  - figma make flex grid
  - figma make rename
dependencies:
  - figma-make-code-review
  - figma-make-type-consolidate
  - figma-make-a11y-modal
version: "0.1.1"
---

# figma-make-refactor

Orchestrates the refactor pass that follows a successful Figma Make port.
The port (`figma-make-import`) gets the project to "it runs". This skill
gets it to "it's maintainable".

## When to use this skill

Use after a `figma-make-import` run has finished and the user asks to
"clean up", "refactor", "fix the layout", "deduplicate components", or
"make the modal accessible". Do **not** use during the port itself.

## Precondition gate (do not skip)

Refuse to start until all of these hold. If any fails, stop and tell the
user what's missing.

- [ ] `npm run dev` boots without errors.
- [ ] `npm run typecheck` output is captured in `CONVERSION_NOTES.md` §2.
- [ ] `CONVERSION_NOTES.md` R-items exist (from `figma-make-code-review`).
- [ ] At least one smoke-level E2E, visual baseline, or screenshot set
      exists so regressions are catchable. If none, offer to add a
      minimal Playwright smoke before proceeding.

The gate matters because every pass below mutates the fragile
`src/imports/*` graph. Without a baseline, regressions are silent.

## Pass order (do not reorder)

```
1. Type consolidation   → figma-make-type-consolidate
2. A11y on modals       → figma-make-a11y-modal
3. Layout (abs → flex)  → this skill, §Layout pass
4. File renames         → this skill, §Rename pass
```

**Why this order.** Types first because drift errors make every other
pass harder to verify. A11y second because it's small, self-contained,
and doesn't touch layout or names. Layout third because it needs the
component graph stable. Renames last because they touch every import
site and want a green typecheck + green smoke test to confirm nothing
broke.

## Layout pass (absolute → flex/grid)

Judgment-heavy. No universal recipe — work one component at a time.

1. Pick a target from the code review's "absolute positioning density"
   finding (highest count first).
2. Capture a baseline screenshot at 3 widths (narrow 375, medium 768,
   wide 1440) before touching anything.
3. Replace `position: absolute` + pixel offsets with flex/grid at the
   parent level. Keep Tailwind classes; prefer `gap-*`, `grid-cols-*`,
   and `flex-col` over margin math.
4. Compare against the baseline at the same 3 widths. Diff visually.
5. Commit per component, not per file. The diff is reviewable only at
   that granularity.

Do not attempt to convert all absolute positioning in one pass. Budget
~1 component per session.

## Rename pass

Mechanical but touches every import site. Do this last.

1. Build the rename map from `chat.txt` (via `figma-make-chat-replay`)
   and any Figma node names you have. Example:
   `Frame57793154 → TriggerModal`, `EditMode-2-1 → ConditionRow`.
2. Write the map to `Code Conversion Output/Projects/<slug>/RENAME_MAP.md` so reviewers can
   audit it.
3. Rename file by file, updating imports with a codemod:

   ```bash
   # dry run
   rg -l "from ['\"].*Frame57793154" src | xargs -I{} echo {}
   # then sed in place once the list looks right
   ```

4. After each rename: `npm run typecheck && npm test`. Do not batch.
5. Commit per rename with the old→new name in the message.

## When the pass doesn't fit

Some Make exports are better rewritten than refactored — especially if
parallel-iteration drift is severe (3+ sibling folders for the same
component). If type consolidation finds more than ~5 shared types with
diverging shapes, surface that to the user and ask whether to continue
refactoring or reimplement from the chat spec.

## Examples

- "The port is done, let's clean it up." → Run the gate, then pass 1.
- "Fix the modal accessibility." → Skip straight to
  `figma-make-a11y-modal`; don't run the full sequence.
- "Convert the absolute positioning to flex." → Layout pass only, after
  confirming the gate.

## See also

- **figma-make-import** — the port that precedes this refactor.
- **figma-make-code-review** — the findings list this skill consumes.
- **figma-make-type-consolidate** — pass 1.
- **figma-make-a11y-modal** — pass 2.
- **figma-make-sap72-font** — wire SAP "72" typography from F2C's
  bundled fonts. If the project still falls back to `sans-serif`,
  layout work is unreliable (computed widths shift); run this before
  the layout pass.
