# {{PROJECT_SLUG}}

Ported from a Figma Make export (Figma file `{{FIGMA_FILE_ID}}`, project
"{{HUMAN_PROJECT_NAME}}"). Source export kept read-only under
`../../../Figma Make/Projects/{{PROJECT_SLUG}}/`. Do not edit the source
export; make changes here.

## Stack

- Vite {{VERSION}} + React {{VERSION}} + TypeScript
- Tailwind CSS v4 (via `@tailwindcss/vite`)
- shadcn/ui components under `src/app/components/ui/`
- Figma-node-generated components under `src/imports/<NodeName>/`

## Dev

```bash
{{PACKAGE_MANAGER}} install
{{PACKAGE_MANAGER}} dev          # → http://localhost:5173/
{{PACKAGE_MANAGER}} typecheck    # tsc --noEmit
{{PACKAGE_MANAGER}} build        # production build
```

<!--
  Adjust the package manager command above based on the project's lockfile:
    pnpm-lock.yaml → pnpm
    package-lock.json → npm
    yarn.lock → yarn
-->

Node {{NODE_VERSION}} is pinned via `.node-version` (mise).

## Docs

| File | Audience | Contents |
|---|---|---|
| `IMPORT_REPORT.md` | Designers, PMs, engineers | What was ported, fixes, known issues, next steps |
| `CONVERSION_NOTES.md` | AI assistant / engineers | Resume-from-here context + detailed code review |
