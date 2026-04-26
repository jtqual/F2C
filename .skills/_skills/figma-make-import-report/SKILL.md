---
name: figma-make-import-report
description: "Generate a stakeholder-facing IMPORT_REPORT.md summarizing a completed Figma Make port: what was imported, what state it's in, what was fixed, what remains, and next steps. Saved in the project directory."
triggers:
  - import report
  - figma make report
  - write import report
  - conversion report
  - port summary
  - figma make summary
dependencies:
  - figma-make-import
version: "0.1.0"
---

# figma-make-import-report

After porting a Figma Make export (via **figma-make-import**), generate a
clean `IMPORT_REPORT.md` saved in the project directory. This report is
the stakeholder-facing deliverable — readable by designers, PMs, and
engineers — unlike `CONVERSION_NOTES.md` which is internal context for
the AI assistant.

## When to use this skill

Use after the import is complete (the 8-step port checklist is done) or
when the user asks for an "import report", "conversion report", or
"summary of the port".

## Output location

```
Code Conversion Output/Projects/<slug>/IMPORT_REPORT.md
```

Always save inside the ported project directory, not at the repo root.

## Report structure

The canonical template lives at
`Code Conversion Output/Projects/_template/IMPORT_REPORT.template.md`. It is copied into the
project directory during the **figma-make-import** Step 2 and renamed to
`IMPORT_REPORT.md`. Fill in the `{{PLACEHOLDERS}}`; follow the HTML
comments for authoring guidance in each section.

The template has these sections:

1. **Header** — source file ID, destination path, date, status.
2. **What was imported** — overview, source structure table, stack.
3. **What was fixed during the port** — plain-English numbered items.
4. **Known issues (not fixed)** — with *Impact* and *Fix* per item.
5. **TypeScript health** — dev boot status, typecheck error table.
6. **Next steps** — ordered actions for the next session.
7. **Appendix: file name mapping** — Figma node names → domain terms.

## Workflow

```
- [ ] Confirm the import is complete (port checklist done, CONVERSION_NOTES.md exists)
- [ ] Read CONVERSION_NOTES.md for review items, corrections, and open threads
- [ ] Read package.json for stack versions
- [ ] Inventory src/imports/ and src/app/components/ui/ for counts
- [ ] Read App.tsx and key import files to build the file name mapping
- [ ] Fill the template
- [ ] Save as IMPORT_REPORT.md in the project directory
```

## Tone and audience

- **Part 1 of the report** (What was imported, What was fixed) should be
  readable by a UX designer or PM. Avoid jargon; when technical terms are
  necessary, add a parenthetical ("TypeScript — a type-safety layer on
  top of JavaScript").
- **Part 2** (TypeScript health, Appendix) can be fully technical.
- Known issues should always state **who is impacted** so a non-developer
  can triage.

## Anti-patterns

- **Don't duplicate CONVERSION_NOTES.md verbatim.** The report is a
  curated summary, not a copy. CONVERSION_NOTES is the raw evidence;
  IMPORT_REPORT is the narrative.
- **Don't include assistant-internal context** (resume-from-here, Make
  POV corrections). Those live in CONVERSION_NOTES.md only.
- **Don't skip the file name mapping.** Designers will look for their
  Figma frame names and won't find them without this table.

## Examples

- User: "Write the import report for real-time-trigger-proto."
  → Read CONVERSION_NOTES.md, fill template, save to
  `Code Conversion Output/Projects/real-time-trigger-proto/IMPORT_REPORT.md`.

- User: "Give me a summary of the port."
  → Same as above.

## See also

- **figma-make-import** — the port workflow that precedes this.
- **figma-make-code-review** — source of the review findings.
- **figma-make-chat-replay** — source of the open threads.
