# {{PROJECT_SLUG}}

Ported from a Figma Make export (Figma file `{{FIGMA_FILE_ID}}`, project
"{{HUMAN_PROJECT_NAME}}"). Source export kept read-only under
`../../../Figma Make/Projects/{{PROJECT_SLUG}}/`. Do not edit the source
export; make changes here.

## Stack

- Vite {{VERSION}} + React {{VERSION}} + TypeScript
- Tailwind CSS v4 (via `@tailwindcss/vite`)
- shadcn/ui components under `src/app/components/ui/`
- Figma-node-generated components under `src/app/imports/<NodeName>/`

## Dev

```bash
npm install
npm run dev          # → http://localhost:5173/
npm run typecheck    # tsc --noEmit
npm run build        # production build
```

Node {{NODE_VERSION}} is pinned via `.node-version` (read by both nvm
and mise). F2C standardizes on **npm + nvm**; if you see a
`pnpm-workspace.yaml` or `pnpm-lock.yaml` in the source export, it's a
Make build-environment artifact and is removed during the port.

## Docs

| File | Audience | Contents |
|---|---|---|
| `IMPORT_REPORT.md` | Designers, PMs, engineers | What was ported, fixes, known issues, next steps |
| `CONVERSION_NOTES.md` | AI assistant / engineers | Resume-from-here context + detailed code review |
