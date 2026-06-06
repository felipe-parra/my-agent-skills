# Changelog

All notable changes to this repo are listed here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
for tagged releases — pin `REF=vX.Y.Z` in the installer for reproducible setups.

## [Unreleased]

### Added
- `customize-claude` skill — translates fuzzy project knowledge
  (conventions, recurring procedures, guardrails) into the right Claude Code
  primitive (skill, slash command, or hook) at the right scope (project vs user).
  Includes a decision matrix so the choice between the three primitives is
  explicit, not vibes-based.
- README "Quickstart" rewritten as a two-step flow (base install →
  customize-for-this-project) with a dedicated section explaining the
  customization loop.

## [0.1.0] - 2026-06-06

### Added
- Initial scaffolding: `skills/`, `standards/`, `install.sh`.
- Nine skills shipped:
  `create-tests`, `explain-module`, `generate-release-notes`,
  `investigate-build`, `publish-changes`, `refactor-component`,
  `review-pr`, `summarize-branch`, `tackle-issue`.
- Three baseline standards: `00-base.md`, `01-typescript.md`, `02-spanish-ui.md`.
- CI workflow that lints `install.sh` with shellcheck and validates that
  every `SKILL.md` has the required frontmatter keys.
- `README.md` documenting both the template-repo and install-script usage modes.
