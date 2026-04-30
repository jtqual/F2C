# F2C — Figma Make → Cursor / Claude conversion workspace

This repo holds the machinery to port Figma Make exports into a
runnable, assistant-friendly codebase.

**Layout (input → output, mirrored slugs):**

- `Figma Make/Projects/<slug>/` — the user drops a Figma Make export
  here. **Untracked** (gitignored) except for `_template/`, which is a
  structural placeholder with a drop-zone README.
- `Code Conversion Output/Projects/<slug>/` — the ported, runnable
  version of that export. **Untracked** (gitignored) except for
  `_template/`, which holds the tracked doc standards
  (`CONVERSION_NOTES`, `IMPORT_REPORT`, `README` templates) copied into
  each port.

Each project keeps the **same `<slug>`** in both trees so input and
output line up.

## Skills (agnostic / multi-ecosystem)

This repo maintains portable skills under `.skills/` (manifest:
`.skills/_index.md`). We follow **Path B** of the skills-harness
bootstrap: we do **not** install a tool-specific runtime harness from
`.skills/_harness/*_template.md` in this tree. Those files are
**reference** for consumers who clone this repo and run Path A in their
own environment (Cursor, Claude Code, etc.).

**Authoring:** use the bundled `skill-template` and `skill-author`
skills, and register new skills in `.skills/_index.md`. Do not paste
ecosystem harness blocks into this file for this repository.

**Currently-authored skills for the F2C workflow:**

- `figma-make-import` — end-to-end port of a Figma Make export.
- `figma-make-chat-replay` — reconstruct `chat.txt` as resume context.
- `figma-make-code-review` — catalogue Make's recurring code smells.
- `figma-make-import-report` — generate a stakeholder-facing import report.
- `figma-make-sap72-font` — wire SAP's "72" typeface from
  `resources/typefaces/` when the port references `font-['72:*']`.
- `figma-make-refactor`, `figma-make-type-consolidate`,
  `figma-make-a11y-modal` — sequenced post-port refactor passes.

## Conventions

- **Source exports are read-only.** Never edit anything under
  `Figma Make/Projects/<slug>/`. Copy to
  `Code Conversion Output/Projects/<slug>/` first, edit there.
- **Both `Projects/` trees are git-ignored** (except their
  `_template/` exemplars). Exports and ports live on disk but are not
  committed — they're per-machine working content. The templates
  (doc standards under `Code Conversion Output/Projects/_template/`:
  `CONVERSION_NOTES.md`, `IMPORT_REPORT.md`, `README.md`) are the
  tracked, reusable artifacts.
- **Personal post-mortem** — optional file
  `Code Conversion Output/Projects/POST_MORTEM.md` (or next to a port)
  for your own process review; it is not tracked and must not live at
  the repo root (root `POST_MORTEM.md` is ignored).
- **Package manager is npm.** F2C standardizes on **npm + nvm**.
  Figma Make exports ship `pnpm-workspace.yaml` and (occasionally) a
  pnpm lockfile — both are artifacts of Make's build environment, not
  signals about how F2C ports should run. The `figma-make-import` skill
  removes `pnpm-workspace.yaml` and any pnpm lockfile during the port
  and produces an npm-shaped `package.json`. Only deviate from npm if
  the user explicitly asks, or if a port is hard-blocked on a
  pnpm-only feature (workspace protocol, hoisting flags, etc.) — and
  document the deviation in `CONVERSION_NOTES.md`.
- **Node runtime** is pinned per-project via `.node-version`
  (currently Node 24 LTS). The file is read by both nvm and mise, so
  it works regardless of which runtime manager a contributor uses.

## Notes for agents

- When asked to convert a Figma Make project, load
  **`figma-make-import`** first; it links to the chat-replay and
  code-review skills.
- The Figma API **cannot** fetch Figma Make projects. The only source
  is the folder the user downloads from Figma Make, which always
  contains a `chat.txt` next to the project directory.
- Reference: [Figma Make docs](https://developers.figma.com/docs/code/intro-to-figma-make/).
