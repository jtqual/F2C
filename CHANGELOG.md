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
  note.
- `figma-make-import` Step 3 now explicitly removes
  `pnpm-workspace.yaml`, `pnpm-lock.yaml`, and `pnpm.overrides` from
  Make exports during the port; package-manager prompt to the user
  is gated on a hard block (workspace protocol etc.) rather than
  default-on.
- All skill examples (`figma-make-refactor`, `figma-make-code-review`,
  `figma-make-type-consolidate`) use `npm run …` commands.
- `figma-make-code-review` checklist adds explicit items for stale
  pnpm artifacts and missing SAP 72 wiring.

## [0.1.0] - 2026-04-29

### Added
- Initial public structure: `Figma Make/Projects/`, `Code Conversion Output/Projects/`, `resources/`, `.skills/`.
- Skills: `figma-make-import`, `figma-make-chat-replay`, `figma-make-code-review`, `figma-make-import-report`.
- Two-branch release model: `main` (development trunk) and `clone-me` (stable, end-user clone target).
- `CHANGELOG.md` and semantic versioning policy.

[Unreleased]: https://github.com/jtqual/F2C/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/jtqual/F2C/releases/tag/v0.1.0
