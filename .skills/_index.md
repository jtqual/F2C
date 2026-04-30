# Skills Index

Kit version: see `.skills/_meta.yml` (optional metadata only). Human-facing copies also appear in root `README.md` and `AGENTS_skills.md`; bump them with the **kit-release** skill and validate with `check.sh`.

Load a skill only when the task clearly requires it.
Read the full `SKILL.md` only at that point — never preemptively.

| name | description | triggers |
|------|-------------|----------|
| figma-make-chat-replay | Reconstruct a Figma Make `chat.txt` into a resume-from-here context doc, correcting the Make assistant's POV where it diverges from the code. | figma make chat, chat.txt, replay figma make, resume figma make, reconstruct figma make conversation, figma make transcript |
| figma-make-code-review | Review Figma Make-generated code for its recurring smells (absolute positioning, SVG-mask text, empty fonts, iteration drift, app-vs-library manifest) and emit a prioritized findings list. | figma make code review, review figma make, audit figma make, make export smells, figma make bugs, figma make typecheck |
| figma-make-import | Port a Figma Make export (chat.txt + project folder) into a runnable, assistant-friendly repo with correct tooling, tsconfig, manifest, and CONVERSION_NOTES.md. | figma make, figma make import, figma make export, convert figma make project, port figma make, f2c, figma to cursor, figma to claude |
| figma-make-import-report | Generate a stakeholder-facing IMPORT_REPORT.md summarizing a completed Figma Make port: what was imported, state, fixes, known issues, and next steps. | import report, figma make report, conversion report, port summary, figma make summary |
| figma-make-refactor | Sequence the first refactor pass on a ported Figma Make project: gate on preconditions, then drive types, a11y, layout, and renames in order. | figma make refactor, refactor figma make, make refactor pass, clean up figma make, figma make flex grid, figma make rename |
| figma-make-type-consolidate | Deduplicate parallel-iteration type drift across `src/imports/*` siblings into a single shared type module. | figma make type drift, consolidate figma make types, dedupe targetcondition, figma make shared types, fix targetcondition drift, figma make iteration drift |
| figma-make-a11y-modal | Add the missing a11y primitives to Figma Make modals: role=dialog, aria-modal, labelled heading, Escape-to-close, focus trap, initial focus. | figma make a11y, figma make modal accessibility, fix modal a11y, add focus trap, escape to close modal, aria-modal figma make |
| figma-make-sap72-font | Wire SAP's "72" typeface (and 72 Mono) into a ported Figma Make project from F2C's bundled `resources/typefaces/72-TrueType-allstyles/`: copy referenced TTFs, emit fonts.css with @font-face blocks matching Make's `font-['72:Style']` literals, import from index.css. | sap 72 font, figma make font, figma make 72 font, figma make typography, 72 font missing, font 72 not loading, figma make sans-serif fallback, wire sap 72, missing fonts.css |
| harness-upgrade | Upgrade a skills-harness installation to the latest version with native IDE discovery. | upgrade harness, update harness, migrate harness, add native discovery, enable IDE symlinks, update skills system |
| kit-release | Bump the skills-harness kit semver and keep CHANGELOG, README, AGENTS_skills.md, and _meta.yml in sync. | bump kit version, bump harness version, release skills harness, cut a harness release, skills-harness version, kit release |
| skill-author | How to write a new SKILL.md from scratch and register it in the index. | write a skill, author a skill, new skill, add a skill |
| skill-template | Canonical SKILL.md format with authoring notes and refactor guide. | new skill, skill format, create skill, reformat skill, convert rule |

Add a row here whenever a new skill is added to `.skills/_skills/`.
