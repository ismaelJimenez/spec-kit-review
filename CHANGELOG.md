# Changelog

All notable changes to this extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.2] - 2026-09-09

### Fixed

- `provides.scripts[].name` now uses plain slugs (`detect-changed-files`, `detect-changed-files-powershell`) instead of file names. Spec Kit 1.0.x validates these entries against `^[a-z0-9-]+$`, so the previous `detect-changed-files.sh` / `detect-changed-files.ps1` values failed with `Validation Error: Invalid script name` and blocked installation entirely (#4). Script paths and command frontmatter are unchanged.

### Added

- `runtimes` and `description` metadata on both `provides.scripts` entries
- CI manifest validation for `provides.scripts` and `provides.templates`: slug naming, duplicate names, `file:` existence, valid `runtimes`, and the non-authorable `strategy` key

## [1.0.1] - 2026-04-04

### Fixed

- Removed invalid alias `speckit.review` (two-segment name); the canonical command `speckit.review.run` is now the only entry point — fixes `Validation Error: Invalid alias` on `specify extension add`
- Added alias naming validation to CI workflow to catch invalid aliases before release

## [1.0.0] - 2026-03-05

### Added

- Command: `/speckit.review.run` (alias: `/speckit.review`) — coordinator that orchestrates all agents
- Command: `/speckit.review.code` — code quality reviewer (guideline compliance, bugs, security)
- Command: `/speckit.review.comments` — comment accuracy analyzer (documentation, comment rot)
- Command: `/speckit.review.tests` — test coverage analyzer (behavioral coverage, critical gaps)
- Command: `/speckit.review.errors` — error handling reviewer (silent failures, catch blocks)
- Command: `/speckit.review.types` — type design analyzer (encapsulation, invariants)
- Command: `/speckit.review.simplify` — code simplification advisor (clarity, complexity)
- Targeted review via aspect arguments (`/speckit.review.run tests errors`)

### Requirements

- Spec Kit: >=0.1.0
- git: Required for change detection
