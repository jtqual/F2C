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
  - figma-make-sap72-font
version: "0.3.1"
---

# figma-make-import

Port a downloaded Figma Make project (the folder Make gives you plus its
`chat.txt`) into a repo that a coding assistant (Cursor, Claude Code, etc.)
can work in productively.

Figma Make files are not fetchable via the Figma API. The user hands you
a local folder. The F2C convention is to drop it under
`Figma Make/Projects/<input-name>/` (the user names this anything —
project title, casual name, even with spaces or emoji) and produce
the ported output at `Code Conversion Output/Projects/<output-slug>/`
where `<output-slug>` is the **kebab-ascii** form of the input name
(see preflight #3). The two trees no longer mirror exactly: the
input name documents what the user dropped, the output slug serves
the npm + shell + git toolchain that the port runs against.

The dropped export typically looks like:

```
Figma Make/Projects/<input-name>/
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
   missing under `Figma Make/Projects/<input-name>/`, ask the user
   where they are.
2. **Confirm git state.** If the workspace is not a git repo, ask before
   `git init`. Use `main` as the default branch name.
3. **Decide destination — input name preserved, output slug normalized.**
   The input folder under `Figma Make/Projects/<input-name>/` is
   whatever the user dropped in (could be `AI Scoring`, could be
   `Quality Management App`, could have emoji). Leave it alone; that
   tree is read-only and exists to mirror what Figma Make shipped.

   The destination under `Code Conversion Output/Projects/<output-slug>/`
   is **always kebab-ascii**: lowercase letters, digits, and hyphens
   only. No spaces, no uppercase, no emoji, no accents. This is
   non-negotiable — every shell command, npm `name` field, and CI
   tool downstream assumes ASCII slugs, and the failures from
   skipping this rule (the agent's `pwd` reports a different
   directory than `working_directory`, `xargs` choking, npm rejecting
   the manifest name, `cd` requiring careful quoting on every call)
   are exactly the bugs that surface when slugs have spaces.

   **Mapping rule.** Compute the output slug from the input name:

   ```
   "AI Scoring"               → "ai-scoring"
   "Quality Management App"   → "quality-management-app"
   "My Cool Demo (v2)"        → "my-cool-demo-v2"
   "Survey 🟢 GA"              → "survey-ga"
   ```

   Lowercase, replace non-ASCII (emoji, accents) with empty string,
   replace any run of non-`[a-z0-9]` with a single hyphen, trim
   leading/trailing hyphens. If the result is empty or starts with a
   digit only, ask the user to pick a slug.

   Show the user the mapping before creating the destination, so they
   can override if they want a different slug. Record both the input
   name and output slug in `CONVERSION_NOTES.md` so the source ↔ port
   linkage is findable later.

   This breaks the prior "same slug in both trees" rule — intentionally.
   Input name documents what the user dropped; output slug serves the
   toolchain.
4. **Package manager is npm — no question asked.** F2C standardizes
   on npm + nvm. Make exports ship `pnpm-workspace.yaml` and sometimes
   a pnpm lockfile; both are Make build-environment artifacts, not
   signals. Always:
   - Use `npm install` / `npm run dev` / `npm run typecheck` etc.
   - Delete `pnpm-workspace.yaml` and any `pnpm-lock.yaml` during the
     port (note in CONVERSION_NOTES.md §"Summary of changes").
   - Strip the `pnpm.overrides` block from `package.json`.

   **Only ask the user about package manager if there's a hard block** —
   e.g. the project genuinely needs pnpm workspace protocol (`workspace:*`),
   pnpm-specific hoisting, or the user explicitly says "use pnpm" up
   front. In that case, document the exception in `CONVERSION_NOTES.md`.
   Most users will not know to ask, so don't prompt them by default.

5. **Confirm runtime.** Node 24 LTS (matching the F2C default) is the
   right answer. Drop a `.node-version` containing the version string
   (e.g. `24.15.0`); both nvm and mise read this file.

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
- Note the input name (the folder under `Figma Make/Projects/` —
  preserve as-is).
- Compute the output slug by normalizing the input name to kebab-ascii
  (see preflight #3); show the mapping to the user.
- Look for Make smells before touching anything — see **figma-make-code-review**.

### Step 2 — Copy templates + source

Use the names from preflight #3:

- `<input-name>` — the folder the user dropped at
  `Figma Make/Projects/<input-name>/` (whatever they named it).
- `<output-slug>` — the kebab-ascii destination under
  `Code Conversion Output/Projects/<output-slug>/`.

Start from the template so the standard doc structure is in place:

```bash
cp -R "Code Conversion Output/Projects/_template/" "Code Conversion Output/Projects/<output-slug>/"
mv "Code Conversion Output/Projects/<output-slug>/CONVERSION_NOTES.template.md" "Code Conversion Output/Projects/<output-slug>/CONVERSION_NOTES.md"
mv "Code Conversion Output/Projects/<output-slug>/IMPORT_REPORT.template.md"    "Code Conversion Output/Projects/<output-slug>/IMPORT_REPORT.md"
mv "Code Conversion Output/Projects/<output-slug>/README.template.md"            "Code Conversion Output/Projects/<output-slug>/README.md"
```

Then copy the Make export's source files into the same directory:

```bash
cp -R "Figma Make/Projects/<input-name>/<Human project name>/"* "Code Conversion Output/Projects/<output-slug>/"
find "Code Conversion Output/Projects/<output-slug>" -name ".DS_Store" -delete
```

Note that the Make export's bundled `README.md` will overwrite the
template's `README.md` — copy the template README back afterwards:

```bash
cp "Code Conversion Output/Projects/_template/README.template.md" "Code Conversion Output/Projects/<output-slug>/README.md"
```

Never edit the source under `Figma Make/Projects/<input-name>/`.

The output slug also feeds the npm `package.json` `name` field in
Step 3 (it's already kebab-ascii, so no further transform is needed).

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
- Drop the `pnpm.overrides` block (npm doesn't read it).
- If Make duplicate-keyed every dep as `"foo": "1.2.3", "foo@1.2.3": "npm:foo@1.2.3"`
  (an artifact of the `@aikidosec/safe-chain` proxy), collapse each
  pair to a single normal entry.
- Add `devDependencies`: `typescript`, `@types/react`, `@types/react-dom`.
- Add `scripts.typecheck`: `"tsc --noEmit"`.
- Keep `scripts.dev` (`vite`) and `scripts.build` (`vite build`); add
  `scripts.preview` (`vite preview`).

Also delete sibling files at the project root that Make ships but npm
doesn't use:

- `pnpm-workspace.yaml`
- `pnpm-lock.yaml` (if present)

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
npm install
npm run dev       # confirm "VITE ... ready" line
npm run typecheck # record any errors verbatim
```

**Do auto-fix the versioned-import bug.** Make ships imports like
`from "@radix-ui/react-dialog@1.1.6"` (the package version baked into
the module specifier) in roughly every shadcn/ui wrapper. This is a
hard block: Vite starts but every page request fails with
`Failed to resolve import`. The fix is mechanical and required for the
port to be considered runnable:

```bash
# from the project root, after copying source
grep -rlE 'from "[^"]+@[0-9]+\.[0-9]+\.[0-9]+"' src \
  | while IFS= read -r f; do
      sed -i '' -E 's/"([^"]+)@[0-9]+\.[0-9]+\.[0-9]+"/"\1"/g' "$f"
    done
# verify zero remaining
grep -rE '"[^"]+@[0-9]+\.[0-9]+\.[0-9]+"' src | wc -l   # → 0
```

Note this in `CONVERSION_NOTES.md` as **fixed on port** (count of files
touched, count of imports stripped).

**Do not auto-fix other type errors on the port step.** They are
evidence for the review. Only add non-semantic fixes (e.g. the `*.png`
ambient declaration in env.d.ts). Capture the post-strip
`npm run typecheck` count verbatim. If dev still fails to start after
the strip, that *is* fixable — it's a blocker for the port.

**Wire SAP "72" font if referenced.** If `grep -rl "font-\['72:"`
finds matches anywhere under `src/`, run the **figma-make-sap72-font**
skill before considering the port runnable. The font files are
bundled in `resources/typefaces/72-TrueType-allstyles/` and the skill
copies + wires them in.

### Step 7 — Fill CONVERSION_NOTES.md and IMPORT_REPORT.md

Both files were copied from templates in Step 2. Fill the
`{{PLACEHOLDERS}}` using:

1. **CONVERSION_NOTES.md** — two sections:
   - §1 Resume-from-here context (via **figma-make-chat-replay**).
   - §2 Code review (via **figma-make-code-review**), including any
     `npm run typecheck` output.
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
chore: port <output-slug> — repo-level updates from Figma Make import

- skill / template / doc tweaks surfaced during the port
- (no project source committed; lives under Code Conversion Output/Projects/<output-slug>/)
```

## Common pitfalls

- **Absolute paths from Figma coordinates.** Do not "clean up" absolute
  positioning during the port — the user needs the original as a
  reference. Flag in review.
- **`figma:asset/` resolver.** Make's `vite.config.ts` often includes a
  resolver plugin for `figma:asset/*` imports pointing to `src/assets/`.
  If `src/assets/` doesn't exist and no code uses the prefix, leave both
  as-is; note it in review.
- **Missing fonts.** Make often references SAP's "72" family in
  hundreds of `font-['72:Regular',_sans-serif]` class strings without
  shipping a `fonts.css`, an `@font-face` block, or the typeface itself.
  Use the **figma-make-sap72-font** skill — the .ttf files are bundled
  at `resources/typefaces/72-TrueType-allstyles/`. If a project
  references a *different* missing font, flag it in CONVERSION_NOTES.md
  R-items rather than guessing.
- **Rename freeze (hard rule).** Do not rename files, folders, or
  exports during the port — not `Frame57793154.tsx`, not `EditMode-2-1/`,
  not the parallel-iteration siblings. The `src/imports/*` graph is
  fragile and undocumented; renames are only safe after (a) `npm run dev`
  runs cleanly, (b) `npm run typecheck` output is captured, and (c) there
  is at least a smoke-level E2E or visual baseline to catch regressions.
  Renaming belongs in the refactor pass, not the port.
- **Parallel iterations.** Watch for `FooName/` and `FooName-1/`,
  `-2/`, `-N-1/` sibling folders — Make emits divergent copies of the
  same component. Types often drift between them. Typecheck will expose
  this.
- **Safe-chain / minimum package age.** If the user's npm registry
  proxy (e.g. Aikido `safe-chain`) blocks or warns on packages below
  a minimum age, `npm install` will print a notice; it's not a
  failure. Capture and move on.

## Examples

- "I dropped a Figma Make export in the repo — can you bring it over?"
  → Run this skill end-to-end. Confirm the input folder name under
  `Figma Make/Projects/`, derive the kebab-ascii output slug, show
  the user the mapping before creating anything.
- "Port `Figma Make/Projects/My Cool Demo/MyCoolDemo` into this workspace."
  → Input name = `My Cool Demo`, output slug = `my-cool-demo`.
  Destination = `Code Conversion Output/Projects/my-cool-demo/`,
  follow the 8-step checklist.

## See also

- **figma-make-chat-replay** — builds the resume-from-here section.
- **figma-make-code-review** — catalogues Make-specific smells.
- **figma-make-sap72-font** — wire the SAP "72" typeface from
  `resources/typefaces/72-TrueType-allstyles/` when the port references
  `font-['72:*']` classes.
- **figma-make-import-report** — generate the stakeholder-facing
  IMPORT_REPORT.md once the port settles.
