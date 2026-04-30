---
name: figma-make-sap72-font
description: "Wire SAP's '72' typeface (and 72 Mono) into a ported Figma Make project: copy the bundled .ttf files, emit fonts.css with @font-face blocks whose font-family literals match Make's font-['72:Style'] class strings, and import from index.css."
triggers:
  - sap 72 font
  - figma make font
  - figma make 72 font
  - figma make typography
  - 72 font missing
  - font 72 not loading
  - figma make sans-serif fallback
  - wire sap 72
  - missing fonts.css
dependencies: []
version: "0.1.0"
---

# figma-make-sap72-font

Wire the SAP "72" typeface family into a ported Figma Make project.

Figma Make exports for SAP-style designs reference the "72" family in
hundreds of `font-['72:Regular',_sans-serif]`-shaped class strings, but
ship no `fonts.css`, no `@font-face` declaration, and no font files.
Browsers silently fall back to the second `font-family` entry
(`sans-serif`), so typography never matches the Figma source.

The F2C repo bundles the licensed TrueType files at
`resources/typefaces/72-TrueType-allstyles/`. This skill copies the
ones a port actually uses, generates the right `@font-face` blocks,
and imports them.

## When to use this skill

Use during or just after `figma-make-import` when **any** of these are
true:

- The port references `font-['72:` anywhere under `src/`.
- The port references `font-['72Mono:` anywhere under `src/`.
- Designs render in `sans-serif` instead of the SAP design system look.
- The user mentions "missing fonts", "wrong font", "fallback font",
  "SAP 72", or "72 typeface".

Do **not** use this skill for non-SAP fonts. If a port references some
other family that isn't loaded, flag it in `CONVERSION_NOTES.md` and
ask the user how to source it instead of guessing.

## Preflight checks (do not skip)

1. **Confirm the bundled fonts exist.** Verify
   `resources/typefaces/72-TrueType-allstyles/` is reachable from the
   repo root with all twelve `.ttf` files listed in §Style mapping
   below. If files are missing, stop and tell the user — don't
   substitute fonts.

2. **Confirm the project actually uses 72.** Run:

   ```bash
   grep -rE "font-\['72:" src | wc -l
   grep -rE "font-\['72Mono:" src | wc -l
   ```

   If both return 0, the project doesn't need this skill. Stop.

3. **Confirm the styles entry chain.** Most Make exports load styles
   via `src/styles/index.css` which `@import`s sibling CSS files. If
   the project uses a different entry (e.g. CSS imported directly from
   `main.tsx`), read `main.tsx` and `index.css` first to find where
   `@import './fonts.css'` should land.

## Style ↔ TTF mapping

This is the canonical mapping. Treat it as the source of truth.

| Make class string | Font-family literal (must match) | TTF file (in `resources/typefaces/72-TrueType-allstyles/`) | Weight | Style |
|---|---|---|---|---|
| `font-['72:Light',_…]` | `'72:Light'` | `72-Light.ttf` | 300 | normal |
| `font-['72:Regular',_…]` | `'72:Regular'` | `72-Regular.ttf` | 400 | normal |
| `font-['72:Italic',_…]` | `'72:Italic'` | `72-Italic.ttf` | 400 | italic |
| `font-['72:Semibold',_…]` | `'72:Semibold'` | `72-Semibold.ttf` | 600 | normal |
| `font-['72:Semibold_Duplex',_…]` | `'72:Semibold Duplex'` | `72-SemiboldDuplex.ttf` | 600 | normal |
| `font-['72:Bold',_…]` | `'72:Bold'` | `72-Bold.ttf` | 700 | normal |
| `font-['72:BoldItalic',_…]` | `'72:BoldItalic'` | `72-BoldItalic.ttf` | 700 | italic |
| `font-['72:Black',_…]` | `'72:Black'` | `72-Black.ttf` | 900 | normal |
| `font-['72:Condensed',_…]` | `'72:Condensed'` | `72-Condensed.ttf` | 400 | normal (condensed) |
| `font-['72:CondensedBold',_…]` | `'72:CondensedBold'` | `72-CondensedBold.ttf` | 700 | normal (condensed) |
| `font-['72Mono:Regular',_…]` | `'72Mono:Regular'` | `72Mono-Regular.ttf` | 400 | normal (mono) |
| `font-['72Mono:Bold',_…]` | `'72Mono:Bold'` | `72Mono-Bold.ttf` | 700 | normal (mono) |

**Key insight:** Make's class strings use the literal `'72:Style'`
(with the colon) as the font-family name. The `@font-face` blocks
must match that literal *exactly*, otherwise Tailwind's
`font-['72:Regular',_sans-serif]` falls through to `sans-serif`. Do
not normalize to a "clean" family name like `SAP 72`.

The `Make class string ↔ Font-family literal` mapping has one quirk:
Make sometimes uses an underscore as a class-name escape for the
space in `Semibold_Duplex`, but the actual `font-family` it expects
is `'72:Semibold Duplex'` (with a real space). The `@font-face` block
for that one file must use the spaced form.

## Workflow

```
- [ ] Step 1: Detect which styles are referenced (subset)
- [ ] Step 2: Copy only the referenced TTFs into the project
- [ ] Step 3: Write src/styles/fonts.css with @font-face blocks
- [ ] Step 4: Wire @import from src/styles/index.css
- [ ] Step 5: Verify in dev (network panel + computed font-family)
- [ ] Step 6: Note in CONVERSION_NOTES.md as "fixed on port"
```

### Step 1 — Detect which styles are referenced

Don't copy all twelve files when the project uses four. Run:

```bash
# from project root (the ported one, not the source)
grep -rhoE "font-\['(72|72Mono):[^']+'" src \
  | sort -u
```

The output is the unique set of `'72:Style'` literals used.
Map each to its TTF row in §Style mapping.

### Step 2 — Copy the TTFs

Create `src/styles/fonts/` and copy only the referenced files:

```bash
mkdir -p src/styles/fonts
cp ../../../resources/typefaces/72-TrueType-allstyles/72-Regular.ttf       src/styles/fonts/
cp ../../../resources/typefaces/72-TrueType-allstyles/72-Bold.ttf          src/styles/fonts/
cp ../../../resources/typefaces/72-TrueType-allstyles/72-Semibold.ttf      src/styles/fonts/
cp ../../../resources/typefaces/72-TrueType-allstyles/72-SemiboldDuplex.ttf src/styles/fonts/
# …only the ones the project actually references
```

The relative path `../../../resources/typefaces/…` assumes the standard
F2C layout (`Code Conversion Output/Projects/<slug>/`). Adjust if the
port lives elsewhere.

### Step 3 — Write src/styles/fonts.css

Generate one `@font-face` block per copied TTF. Template:

```css
/*
 * SAP "72" typeface — wired in from F2C's bundled resources.
 * The font-family literals (e.g. '72:Regular') intentionally match
 * Make's class strings like font-['72:Regular',_sans-serif] so the
 * existing component code resolves without any rewrite.
 */

@font-face {
  font-family: '72:Regular';
  src: url('./fonts/72-Regular.ttf') format('truetype');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: '72:Bold';
  src: url('./fonts/72-Bold.ttf') format('truetype');
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

/* …repeat per referenced style, using §Style mapping */
```

Required attributes per block:

- `font-family`: the literal `'72:Style'` (or `'72Mono:Style'`) form —
  must match Make's class strings exactly.
- `src: url('./fonts/<file>.ttf') format('truetype')`.
- `font-weight` and `font-style` per §Style mapping.
- `font-display: swap` (avoid invisible-text-flash; matches normal SaaS
  convention).

For the `Semibold Duplex` row, the `font-family` value is
`'72:Semibold Duplex'` (real space, not underscore).

### Step 4 — Wire it up

Add a single line to `src/styles/index.css`, before the design-token
imports:

```css
@import 'tailwindcss' source(none);
@source '../../**/*.{js,ts,jsx,tsx}';
@import 'tw-animate-css';
@import './fonts.css';        /* ← add this */
@import './default_theme.css';
@import './globals.css';
```

If the project uses a different entry pattern, place the import as
early as possible so other rules can rely on the family being
declared.

### Step 5 — Verify

```bash
npm run dev
```

In the browser:

1. Open DevTools → Network → filter `Font`. Confirm each referenced
   `.ttf` loads with `200`.
2. Inspect any element with `font-['72:Regular',_sans-serif]`. The
   computed `font-family` should be `'72:Regular', sans-serif` and the
   rendered glyphs should match the SAP design (not the platform
   fallback).
3. If a glyph still looks wrong, check whether Make's class string used
   a `:Style` variant you didn't include in the subset (Step 1) — the
   `grep` may have missed an unusual escape.

### Step 6 — Note in CONVERSION_NOTES.md

Mark the relevant code-review item as **fixed on port**:

```markdown
### R_ — SAP "72" font wired in (fixed on port)

Make ships ~N references to font-['72:*'] without bundling the typeface
or shipping a fonts.css. Resolved on port via the figma-make-sap72-font
skill: copied K of 12 referenced styles from
resources/typefaces/72-TrueType-allstyles/ into src/styles/fonts/, wrote
src/styles/fonts.css with matching @font-face blocks, and imported it
from src/styles/index.css. Computed font-family verified in dev.
```

## Common pitfalls

- **Tailwind class string ≠ font-family.** Make uses the literal
  `'72:Regular'` as the family name (with the colon). Don't "fix"
  this to `SAP 72` or `72 Regular` — it would require rewriting every
  `font-['…']` class string in the project.

- **Underscore in `Semibold_Duplex`.** That underscore is Tailwind's
  arbitrary-value escape for a space. The `@font-face` `font-family`
  declaration must use the actual space (`'72:Semibold Duplex'`),
  not the underscore.

- **Missing `font-display: swap`.** Without it, Safari and some Chrome
  versions show invisible text until the font loads. Always include.

- **Copying all twelve files.** Most ports use 3–5 styles. Only ship
  what the project references — keeps the bundle small and the diff
  reviewable.

- **Wrong relative path.** The bundled fonts live at the F2C repo
  root, not in `node_modules/`. The path from
  `Code Conversion Output/Projects/<slug>/` is
  `../../../resources/typefaces/72-TrueType-allstyles/`. If the port
  was relocated, recompute.

- **Licensing.** SAP 72 is licensed for use in SAP-related products;
  the bundled copy in `resources/` is the F2C-blessed copy. Don't
  redistribute via a public CDN. For non-SAP projects, substitute a
  free family — but that's a *different* skill (out of scope here).

## Examples

- "The fonts look wrong" → run grep step 1; if `font-['72:` matches,
  run the full skill.
- "The port runs but typography is off" → check fonts before
  layout/spacing.
- "Wire SAP 72" → run the skill end-to-end.
- "Add 72Mono" → if the project uses `font-['72Mono:`, add those
  blocks too (steps 2–4 with the 72Mono TTFs).

## See also

- **figma-make-import** — calls into this skill at Step 6 of its port
  workflow when a 72 reference is detected.
- **figma-make-refactor** — runs after the font is wired; layout pass
  needs computed widths to be stable, which depends on the right font.
- **figma-make-code-review** — surfaces missing-font findings in the
  R-items list.
