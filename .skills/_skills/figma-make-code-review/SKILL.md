---
name: figma-make-code-review
description: "Review Figma Make-generated code for the specific smells that always show up: absolute positioning, SVG-mask text, size-full, empty font files, numbered/hashed file names, parallel iteration drift, orphan resolvers, and app-vs-library manifest mistakes. Produces a prioritized findings list."
triggers:
  - figma make code review
  - review figma make
  - audit figma make
  - make export smells
  - figma make bugs
  - figma make typecheck
dependencies: []
version: "0.1.1"
---

# figma-make-code-review

Figma Make produces working-ish React apps from Figma designs. The
output is recognizable and its bugs cluster into a short, repeatable
list. This skill walks that list.

## When to use this skill

Use during a Figma Make port (after the source is copied, before the
first refactor) or when the user asks to "audit", "review", or
"clean up" a Figma Make export.

## Review checklist

Copy this into your working notes and tick items off:

```
Manifest & tooling
- [ ] package.json: react/react-dom are `dependencies`, not optional peers
- [ ] package.json: name is the project slug, not @figma/my-make-file
- [ ] tsconfig.json exists; strict; jsx: react-jsx
- [ ] env.d.ts declares *.png / *.svg / *.jpg modules
- [ ] .node-version matches the user's runtime pin
- [ ] `npm run dev` boots without errors
- [ ] `tsc --noEmit` output captured (do NOT auto-fix)

Code smells
- [ ] Absolute positioning density in src/imports/*
- [ ] `size-full` usage (Tailwind arbitrary: size-full = w-full h-full)
- [ ] `font-['<family>',sans-serif]` references with no loaded font
- [ ] SVG-mask text (mask-image / WebkitMaskImage) for visible labels
- [ ] Hash/numeric file names (Frame5779…, EditMode-2-1, image-N.png)
- [ ] Parallel iteration folders (FooName + FooName-1 + FooName-2)
- [ ] TargetCondition / shared-type drift across iterations
- [ ] Magic numbers for layout (e.g. marginRight: '355px')

Vite & assets
- [ ] `figma:asset/` resolver present but no src/assets/ folder
- [ ] assetsInclude lists svg/csv but PNGs imported relatively (OK)
- [ ] Emotion/MUI/Popper present in deps but unused in src/

UI hygiene
- [ ] Modal has role="dialog", aria-modal, Esc-to-close, focus trap
- [ ] Keyboard reachability of all interactive controls
- [ ] Viewport responsiveness (resize to narrow + tall, note breakage)

Manifest deltas
- [ ] Unused deps candidate list prepared (MUI, Emotion, Popper, etc.)
- [ ] Stale pnpm artifacts (`pnpm-workspace.yaml`, `pnpm-lock.yaml`,
      `pnpm.overrides` in package.json) — should be deleted during port
      since F2C runs on npm; flag if any survived
- [ ] SAP "72" font references without bundled fonts — if `font-['72:`
      appears anywhere under `src/`, the **figma-make-sap72-font** skill
      should have wired it in. If not, this is an R-item.
```

## How to run each check

### Absolute positioning density

```bash
rg -c "\babsolute\b" src/imports | sort -t: -k2 -nr
```

Any file with >10 hits is a candidate for refactor to flex/grid after a
visual regression baseline exists.

### SVG-mask text

```bash
rg "mask-image|WebkitMaskImage|maskImage" src
```

If hits exist in label positions, Make exported text as SVG shapes.
Chat often claims this was "fixed to real text"; verify per-file.

### Empty `fonts.css`

```bash
wc -l src/styles/fonts.css
rg "font-\[" src | head
```

Empty file + many `font-['…']` references = silent fallback to
`sans-serif`. Note font family used and whether the project licenses it.

### Parallel iteration drift

```bash
ls src/imports | sort
```

Look for `FooName`, `FooName-1`, `FooName-2-1` patterns. Then:

```bash
rg "export (default )?function|export interface|export type" src/imports/FooName*
```

Compare signatures across siblings. If a shared type (commonly
`TargetCondition`, `Event`, `Item`) has different shapes in different
iterations, that's drift — flag for consolidation.

### Typecheck

```bash
npm run typecheck 2>&1 | tee /tmp/make-tsc.log
```

Paste the output verbatim into `CONVERSION_NOTES.md` review section.
Errors fall into recognizable buckets:

- **TS2739 / missing prop** — iteration X dropped a prop that iteration
  Y still requires.
- **TS2307 / cannot find module '*.png'** — fixed by `env.d.ts`.
- **TS2322 / incompatible TargetCondition** — sibling type drift.

### Unused deps

```bash
rg "@mui|@emotion|@popperjs|react-popper" src
```

Zero matches = safe to drop after first run. Keep the shadcn wrappers
that import them unless the wrapper itself is unused.

## Output format

Finish the review by emitting a section in `CONVERSION_NOTES.md` titled
**"Code review"**, numbered **R1**, **R2**, … — one item per finding.
Each item has:

1. **Title** — one-line summary.
2. **Status** — `fixed on port` / `flagged, not fixed`.
3. **Body** — 1–4 sentences: what, why, how to fix.

Keep findings even when they're cosmetic (double indentation, leading
blank lines in `index.html`/`main.tsx`). Cosmetic findings earn a
single-paragraph "R(n)" item at the bottom grouped under a "Formatter
quirks" title.

## Do not fix during the port

- Absolute positioning — needs visual regression baseline first.
- Font references — licensing decision.
- File renames — imports graph is fragile without tests.
- Unused deps — removing them may hide `npm install` drift issues.
- Semantic TS errors — they are evidence of iteration drift; the next
  refactor uses them as a map.

## Fix during the port

- Missing `tsconfig.json` and `env.d.ts` (build-enabling only).
- `package.json` app-vs-library mistakes (app won't install otherwise).
- Deleting `.DS_Store` inside the copied destination.
- Adding `.node-version`.

## Examples

- User: "Review the Make output I just ported."
  → Run the checklist, run typecheck, emit numbered R-items.

- User: "Why does the modal text look wrong?"
  → Check empty `fonts.css` and `font-['<family>']` references first.

## See also

- **figma-make-import** — wraps this review into the port flow.
- **figma-make-chat-replay** — the review feeds "Corrections" in the
  resume-from-here doc.
