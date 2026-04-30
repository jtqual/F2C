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

## [0.1.0] - 2026-04-29

### Added
- Initial public structure: `Figma Make/Projects/`, `Code Conversion Output/Projects/`, `resources/`, `.skills/`.
- Skills: `figma-make-import`, `figma-make-chat-replay`, `figma-make-code-review`, `figma-make-import-report`.
- Two-branch release model: `main` (development trunk) and `clone-me` (stable, end-user clone target).
- `CHANGELOG.md` and semantic versioning policy.

[Unreleased]: https://github.com/jtqual/F2C/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/jtqual/F2C/releases/tag/v0.1.0
