---
name: figma-make-import
description: "Port a Figma Make export (chat.txt + downloaded project folder) into a runnable, assistant-friendly repo with correct tooling, tsconfig, package manifest, and conversion notes."
triggers:
  - figma make
  - figma make import
  - figma make export
  - convert figma make project
  - port figma make
  - f2c
  - figma to cursor
  - figma to claude
dependencies:
  - figma-make-chat-replay
  - figma-make-code-review
version: "0.1.0"
---

# figma-make-import

Port a downloaded Figma Make project (the folder Make gives you plus its
`chat.txt`) into a repo that a coding assistant (Cursor, Claude Code, etc.)
can work in productively.

Figma Make files are not fetchable via the Figma API. The user hands you
a local folder. The F2C convention is to drop it under
`Figma Make/Projects/<slug>/` (a short kebab-case slug picked by the
user) and produce the ported output at
`Code Conversion Output/Projects/<slug>/` — **same slug in both trees**.

The dropped export typically looks like:

```
Figma Make/Projects/<slug>/
├── chat.txt                                 # raw conversation transcript
└── <Human project name>/                    # the downloaded project
    ├── index.html
    ├── package.json
    ├── vite.config.ts
    ├── pnpm-workspace.yaml
    ├── postcss.config.mjs
    ├── default_shadcn_theme.css
    ├── guidelines/Guidelines.md
    └── src/
        ├── main.tsx
        ├── app/App.tsx
        ├── app/components/{ui,figma}/...    # shadcn components live here
        ├── imports/<FigmaNodeName>/*.tsx    # node-generated components
        └── styles/{index,fonts,tailwind,theme}.css
```

## When to use this skill

Use when the user asks to convert, port, import, or "bring over" a Figma
Make project into the current workspace — or says "Figma to Cursor",
"Figma to Claude", or "F2C".

## Preflight checks (do not skip)

1. **Confirm source layout.** If `chat.txt` or the project folder is
   missing under `Figma Make/Projects/<slug>/`, ask the user where they
   are.
2. **Confirm git state.** If the workspace is not a git repo, ask before
   `git init`. Use `main` as the default branch name.
3. **Decide destination.** Default is
   `Code Conversion Output/Projects/<slug>/` — **the same `<slug>` as
   the input folder**. Keep `Figma Make/Projects/<slug>/` untouched as
   a read-only source.
4. **Pick a package manager by your environment rules.** Honor any
   project-level rules first. If none, apply the usual lockfile rule
   (`package-lock.json`→npm, `pnpm-lock.yaml`→pnpm, `yarn.lock`→yarn,
   none→npm). Figma Make ships `pnpm-workspace.yaml` but no lockfile —
   ask the user if there's ambiguity.
5. **Confirm runtime.** Node 20 LTS or 24 LTS is safe. Drop a
   `.node-version` matching the user's global pin.

## Port workflow

Track progress with this checklist:

```
- [ ] Step 1: Inventory the source
- [ ] Step 2: Copy templates + source to destination
- [ ] Step 3: Fix the package manifest
- [ ] Step 4: Add missing tsconfig + env.d.ts
- [ ] Step 5: Add .node-version and project README (from template)
- [ ] Step 6: Install deps and smoke-test dev + typecheck
- [ ] Step 7: Fill CONVERSION_NOTES.md and IMPORT_REPORT.md (from templates)
- [ ] Step 8: Commit
```

### Step 1 — Inventory

- `cat` / read `package.json`, `vite.config.ts`, `src/main.tsx`,
  `src/app/App.tsx`, `guidelines/Guidelines.md`, and the CSS entry.
- Note the slug (folder name under `Figma Make/Projects/`).
- Look for Make smells before touching anything — see **figma-make-code-review**.

### Step 2 — Copy templates + source

Start from the template so the standard doc structure is in place:

```bash
cp -R "Code Conversion Output/Projects/_template/" "Code Conversion Output/Projects/<slug>/"
mv "Code Conversion Output/Projects/<slug>/CONVERSION_NOTES.template.md" "Code Conversion Output/Projects/<slug>/CONVERSION_NOTES.md"
mv "Code Conversion Output/Projects/<slug>/IMPORT_REPORT.template.md"    "Code Conversion Output/Projects/<slug>/IMPORT_REPORT.md"
mv "Code Conversion Output/Projects/<slug>/README.template.md"            "Code Conversion Output/Projects/<slug>/README.md"
```

Then copy the Make export's source files into the same directory:

```bash
cp -R "Figma Make/Projects/<slug>/<Human project name>/"* "Code Conversion Output/Projects/<slug>/"
find "Code Conversion Output/Projects/<slug>" -name ".DS_Store" -delete
```

Never edit the source under `Figma Make/Projects/<slug>/`.

Note: both `Figma Make/Projects/*` and `Code Conversion Output/Projects/*`
are git-ignored except their `_template/` exemplars. The exported source
and the ported project both live on disk only. The templates and skills
are the tracked, reusable artifacts.

### Step 3 — Fix the package manifest

Figma Make's default `package.json` is wrong for a real app:

- `name` is `@figma/my-make-file` → rename to the project slug.
- `react` and `react-dom` are under `peerDependencies` with
  `peerDependenciesMeta.*.optional: true`. Move them to `dependencies`.
- Drop `peerDependencies`/`peerDependenciesMeta` entirely.
- Add `devDependencies`: `typescript`, `@types/react`, `@types/react-dom`.
- Add `scripts.typecheck`: `"tsc --noEmit"`.
- Keep `scripts.dev` (`vite`) and `scripts.build` (`vite build`); add
  `scripts.preview` (`vite preview`).

Do **not** remove unused deps yet (MUI, emotion, popperjs, etc.). Flag
them in `CONVERSION_NOTES.md` and let the first refactor pass trim.

### Step 4 — Add tsconfig + env.d.ts

Figma Make does not ship a `tsconfig.json`. Add:

- `tsconfig.json`: `strict: true`, `jsx: "react-jsx"`,
  `module: "ESNext"`, `moduleResolution: "bundler"`,
  `paths: { "@/*": ["src/*"] }` (matches Vite alias).
- `tsconfig.node.json`: for `vite.config.ts`.
- `src/env.d.ts`: `/// <reference types="vite/client" />` plus module
  declarations for `*.png`, `*.jpg`, `*.svg` (Make exports hash-named
  PNGs with no declarations).

### Step 5 — .node-version + README

- `.node-version` matching the user's mise/nvm pin (e.g. `24.15.0`).
- Fill in the `README.md` (already copied from `README.template.md`):
  replace `{{PLACEHOLDERS}}` with project-specific values.

### Step 6 — Install + smoke test

```bash
pnpm install   # or npm/yarn per chosen PM
pnpm dev       # confirm "VITE ... ready" line
pnpm typecheck # record any errors verbatim
```

**Do not auto-fix type errors on the port step.** They are evidence for
the review. Only add non-semantic fixes (e.g. the `*.png` ambient
declaration above). If dev fails to start, that *is* fixable — it's a
blocker for the port.

### Step 7 — Fill CONVERSION_NOTES.md and IMPORT_REPORT.md

Both files were copied from templates in Step 2. Fill the
`{{PLACEHOLDERS}}` using:

1. **CONVERSION_NOTES.md** — two sections:
   - §1 Resume-from-here context (via **figma-make-chat-replay**).
   - §2 Code review (via **figma-make-code-review**), including any
     `pnpm typecheck` output.
2. **IMPORT_REPORT.md** (via **figma-make-import-report**) — the
   stakeholder-facing summary. Pull findings from CONVERSION_NOTES.md
   and rewrite for a mixed audience.

The template files contain HTML comments with authoring instructions
for every section. Follow them, then delete the comments.

### Step 8 — Commit

The port output itself is **not committed** (the
`Code Conversion Output/Projects/*` tree is gitignored). What you may
commit are repo-level changes the port surfaced — skills, templates,
docs, fonts, etc. Suggested message for that:

```
chore: port <slug> — repo-level updates from Figma Make import

- skill / template / doc tweaks surfaced during the port
- (no project source committed; lives under Code Conversion Output/Projects/<slug>/)
```

## Common pitfalls

- **Absolute paths from Figma coordinates.** Do not "clean up" absolute
  positioning during the port — the user needs the original as a
  reference. Flag in review.
- **`figma:asset/` resolver.** Make's `vite.config.ts` often includes a
  resolver plugin for `figma:asset/*` imports pointing to `src/assets/`.
  If `src/assets/` doesn't exist and no code uses the prefix, leave both
  as-is; note it in review.
- **Empty `fonts.css`.** Make sometimes ships an empty `src/styles/fonts.css`
  while referencing a font family (e.g. SAP's "72") hundreds of times in
  class strings. Don't delete the import; flag that the font won't load.
- **Rename freeze (hard rule).** Do not rename files, folders, or
  exports during the port — not `Frame57793154.tsx`, not `EditMode-2-1/`,
  not the parallel-iteration siblings. The `src/imports/*` graph is
  fragile and undocumented; renames are only safe after (a) `pnpm dev`
  runs cleanly, (b) `pnpm typecheck` output is captured, and (c) there
  is at least a smoke-level E2E or visual baseline to catch regressions.
  Renaming belongs in the refactor pass, not the port.
- **Parallel iterations.** Watch for `FooName/` and `FooName-1/`,
  `-2/`, `-N-1/` sibling folders — Make emits divergent copies of the
  same component. Types often drift between them. Typecheck will expose
  this.
- **Safe-chain / minimum package age.** If the user's npm registry
  proxy blocks packages below a minimum age, `pnpm install` may warn;
  it's not a failure. Capture and move on.

## Examples

- "I dropped a Figma Make export in the repo — can you bring it over?"
  → Run this skill end-to-end. Confirm the slug under
  `Figma Make/Projects/`.
- "Port `Figma Make/Projects/my-app/MyApp` into this workspace."
  → Destination = `Code Conversion Output/Projects/my-app/`, follow the
  8-step checklist.

## See also

- **figma-make-chat-replay** — builds the resume-from-here section.
- **figma-make-code-review** — catalogues Make-specific smells.
