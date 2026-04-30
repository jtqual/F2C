# Changelog

All notable changes to F2C are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Versioning rules

- **MAJOR** — breaking changes to the layout, skills contract, or anything
  that requires consumers to re-clone or migrate their working tree.
- **MINOR** — additive features (new skills, new templates, new docs).
- **PATCH** — bug fixes, doc clarifications, internal cleanup with no
  user-visible behavior change.

## Release process

1. Land changes on `main` via squash-merge.
2. Update this `CHANGELOG.md` under `[Unreleased]`.
3. Bump the version, move entries from `[Unreleased]` to a new dated section.
4. Tag `vX.Y.Z` on `main` and push the tag.
5. Fast-forward `clone-me` to that tag and push.

```bash
git tag -a vX.Y.Z -m "vX.Y.Z"
git push origin main vX.Y.Z
git checkout clone-me && git merge --ff-only vX.Y.Z && git push
```

## [Unreleased]

## [0.2.0] - 2026-04-30

### Added
- New skill: `language` — voice/register guidance, always-on, governs
  *how* the agent talks (not *what* it does). Audience baseline is
  UX designers porting Figma Make exports who are new to Terminal.
  Imported from a sibling repo and rewritten for F2C: dropped the
  `uxpm-` name prefix, removed references to `scripts/env-check.sh`,
  `local/`, `REPO_VERSION`, and the Homebrew-path branching that
  don't exist here. Wired into `.skills/_index.md` (with an
  always-on note at the top) and AGENTS.md (load at session start).
- New skill: `figma-make-sap72-font` — wires SAP "72" typeface from
  bundled `resources/typefaces/72-TrueType-allstyles/` when a port
  references `font-['72:*']` class strings. Includes the canonical
  Make-class-string ↔ TTF-file ↔ font-family-literal mapping.
- `figma-make-import` Step 6 now performs the mechanical fix for
  Make's versioned-import bug (`from "pkg@1.2.3"` → `from "pkg"`)
  during the port instead of deferring to refactor; this is required
  for dev to render anything.

### Changed
- **Standardized on npm + nvm.** AGENTS.md, README, README template,
  and all skills now assume npm. Removed the pnpm-by-lockfile rule
  and the historical "opted into pnpm for `real-time-trigger-proto`"
  note. `figma-make-import` Step 3 now explicitly removes
  `pnpm-workspace.yaml`, `pnpm-lock.yaml`, and `pnpm.overrides` from
  Make exports during the port; the package-manager prompt to the
  user is gated on a hard block (workspace protocol etc.) rather
  than default-on. All skill examples (`figma-make-refactor`,
  `figma-make-code-review`, `figma-make-type-consolidate`) use
  `npm run …` commands.
- **Slug rule clarified.** Input folder under `Figma Make/Projects/`
  keeps whatever name the user dropped (spaces, mixed case, even
  emoji); output slug under `Code Conversion Output/Projects/` is
  **always kebab-ascii**. The two trees no longer mirror exactly —
  intentional — because shell/npm/CI pain from spaces and emoji in
  output paths was recurring. `figma-make-import` preflight #3
  enforces this with an explicit input-name → output-slug mapping
  shown to the user before anything is created. AGENTS.md
  Conventions block updated; existing ports are not retroactively
  renamed.
- **Refactor rename pass strips non-ASCII from filenames.** Make
  embeds Figma release-status emoji into node names
  (`Banner🟢GeneralAvailability.tsx`,
  `Tier3ToolPane🚨FlaggedForDeprecation.tsx`); these are always an
  R-item in the code review and always stripped during the rename
  sub-pass of `figma-make-refactor`. Not stripped at port time
  because the rename freeze still applies until typecheck is green
  and a visual baseline exists.
- `figma-make-code-review` checklist adds explicit items for stale
  pnpm artifacts, missing SAP 72 wiring, and emoji-in-filenames.

### Fixed
- **Emoji-detection one-liner is portable on stock macOS.** The first
  draft used `LC_ALL=C grep -P "[^\x00-\x7F]"` which fails on BSD
  grep (no `-P` flag). Replaced with
  `perl -ne 'print if /[^\x00-\x7F]/'`, which uses `/usr/bin/perl`
  (always shipped with macOS, no Homebrew required). Verified
  against `Code Conversion Output/Projects/AI Scoring/` (returned
  the expected 19 emoji-named files).
- **Nested fences in the code-review checklist** would have broken
  rendering in some Markdown parsers. Moved the detection one-liner
  out of the checklist code block and into the "How to run each
  check" section.
- **Slug terminology drift.** Swept all skills, AGENTS.md, README.md,
  and `Code Conversion Output/Projects/_template/README.md` so the
  input tree consistently uses `<input-name>` and the output tree
  uses `<output-slug>`. Removes the leftover "same `<slug>` in both
  trees" claim from before the kebab-ascii rule landed.
- **Refactor rename pass — overlapping numbering.** The two "jobs"
  of the rename pass (strip non-ASCII, then domain renames) are now
  named **Sub-pass A** / **Sub-pass B** so they don't collide with
  the numbered Steps 1–4 underneath. Steps apply to each sub-pass.

### Skill versions
- `figma-make-import` 0.1.0 → 0.3.1
- `figma-make-refactor` 0.1.0 → 0.2.1
- `figma-make-code-review` 0.1.0 → 0.1.3
- `figma-make-type-consolidate` 0.1.0 → 0.1.1
- `figma-make-import-report` 0.1.0 → 0.1.1
- `figma-make-sap72-font` 0.1.1 (new)
- `language` 1.0.1 (new)

## [0.1.0] - 2026-04-29

### Added
- Initial public structure: `Figma Make/Projects/`, `Code Conversion Output/Projects/`, `resources/`, `.skills/`.
- Skills: `figma-make-import`, `figma-make-chat-replay`, `figma-make-code-review`, `figma-make-import-report`.
- Two-branch release model: `main` (development trunk) and `clone-me` (stable, end-user clone target).
- `CHANGELOG.md` and semantic versioning policy.

[Unreleased]: https://github.com/jtqual/F2C/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/jtqual/F2C/releases/tag/v0.2.0
[0.1.0]: https://github.com/jtqual/F2C/releases/tag/v0.1.0
