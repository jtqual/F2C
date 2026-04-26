# Conversion notes — {{PROJECT_SLUG}}

<!--
  TEMPLATE — fill in all {{PLACEHOLDER}} values.
  This doc is internal context for the AI assistant. It is NOT shown to
  stakeholders. For the stakeholder-facing summary, see IMPORT_REPORT.md.

  Two mandatory parts, in order:
    1) Resume-from-here context (from chat.txt)
    2) Code review (from inspecting the Make output)
-->

Ported from Figma Make export `{{FIGMA_FILE_ID}} / {{HUMAN_PROJECT_NAME}}`.
Source export: `../../../Figma Make/Projects/{{PROJECT_SLUG}}/`.

This doc has two parts:

1. **Resume-from-here context** — reconstructs the Figma Make conversation so an
   assistant in Cursor/Claude can pick up as if the chat never ended. Written
   first from the Make assistant's point of view, then corrected where that
   POV was wrong or incomplete.
2. **Code review** — findings from inspecting the Make output as-shipped,
   independent of the chat.

---

## 1) Resume-from-here context

### Project intent (user-stated, verbatim fragments from `chat.txt`)

<!--
  Quote the user's own words. One bullet per distinct design decision.
  Later turns override earlier ones. Preserve the user's terminology
  (e.g. "Logic Set", not "Rule Group").
-->

{{BULLETED_LIST_OF_USER_INTENT}}

### What Make says it delivered (assistant POV, from `chat.txt`)

<!--
  Summarize what the Make assistant claims it built, using its own
  language. Do not editorialize — this is Make's POV, not yours.
  Later claims override earlier ones if contradictory.
-->

{{BULLETED_LIST_OF_MAKE_CLAIMS}}

### Corrections / gaps in the Make assistant's POV

<!--
  Cross-check every Make claim against the actual code. One numbered
  item per discrepancy. Categories to look for:

  - Stale claims (code no longer matches what Make said)
  - Partial claims (surface-true but a dependency is broken)
  - Missing claims (code does something chat never discusses)
  - Numbered/duplicate files (chat doesn't identify which is canonical)
  - No compile evidence (chat has no tsc step — errors are invisible)

  Trust the code, not the summary.
-->

1. {{CORRECTION_1}}
2. {{CORRECTION_2}}
3. …

### Open threads to pick up

<!--
  Label T1, T2, … so the next assistant can reference them.
  One sentence of action + one sentence of "why" per thread.
-->

- **T1**: {{ACTION}} — {{WHY}}
- **T2**: {{ACTION}} — {{WHY}}
- …

---

## 2) Code review of the Make output

<!--
  Number findings R1, R2, … Each item has:
    - Title (one line)
    - Status: "fixed on port" or "flagged, not fixed"
    - Body: 1–4 sentences — what, why, how to fix

  Always include these checks (skip if not applicable):
    R_  Package manifest (app vs library, react as dep)
    R_  tsconfig.json missing
    R_  figma:asset/ dead resolver
    R_  Unused heavy deps (MUI, Emotion, Popper, etc.)
    R_  fonts.css empty / font not loaded
    R_  Absolute positioning density
    R_  Opaque Figma-node file names
    R_  Accessibility gaps (modal, keyboard, screen reader)
    R_  Formatter quirks (leading blank lines, double indent)
    R_  Real TypeScript errors (paste tsc output verbatim)
    R_  pnpm-workspace.yaml in a single-package project

  Group cosmetic findings under one item at the end.
-->

### R1 — {{TITLE}} ({{STATUS}})

{{BODY}}

### R2 — {{TITLE}} ({{STATUS}})

{{BODY}}

…

---

## Summary of changes applied during port

<!--
  Bulleted list of everything changed relative to the Figma Make source.
  Include: file copies, manifest fixes, added configs, font wiring,
  deleted .DS_Store, etc. One bullet per change.
-->

- {{CHANGE_1}}
- {{CHANGE_2}}
- …
