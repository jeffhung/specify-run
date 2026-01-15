<!--
Sync Impact Report
==================
Version change: N/A → 0.1.0 (initial planning/PoC)
Modified sections:
  - Project Mission: clarified flexible directory placement
  - Distribution Model: updated for any-directory installation
Templates requiring updates:
  - plan-template.md: ✅ Compatible (Constitution Check section exists)
  - spec-template.md: ✅ Compatible (no constitution-specific content)
  - tasks-template.md: ✅ Compatible (no constitution-specific content)
Follow-up TODOs: None
-->

# specify-run Constitution

## Project Mission

This project builds the `specify-run` tool: a self-bootstrapping, directory-
pinned SpecKit entrypoint script. Any project wanting to use SpecKit securely
installs the `specify-run` script into the directory where SpecKit should be
provisioned (repository root or any subdirectory), then invokes all SpecKit
commands exclusively through it.

**Status**: Planning and proof-of-concept stage.

## Core Principles

### I. Single-File Distribution

The `specify-run` script MUST be a single, self-contained bash file with no
external dependencies beyond Python 3.9+ and Git. Users MUST be able to copy
one file into their project to gain full functionality.

**Rationale**: Single-file distribution minimizes installation friction and
eliminates dependency management for adopting projects.

### II. Zero Configuration

The script MUST work immediately after copying, with sensible defaults. No
configuration files, environment setup, or manual steps MUST be required for
basic operation.

**Rationale**: Zero-friction adoption encourages correct usage over shortcuts.

### III. Pinned Reproducibility

The script MUST pin SpecKit to a specific Git ref (`SPECKIT_REF`). Adopting
projects MUST get deterministic, reproducible behavior across machines, CI,
and time.

**Rationale**: Reproducibility is the core value proposition—without it, the
tool has no purpose.

### IV. Explicit Upgrades

Version changes MUST require editing `SPECKIT_REF` in the script. No automatic,
silent, or network-triggered upgrades are permitted. All changes MUST be visible
in version control.

**Rationale**: Explicit upgrades enable security review and prevent supply-chain
attacks.

### V. Agent-Safe Design

AI agents (Claude Code, etc.) MUST be treated as first-class users. The script
MUST provide clear, unambiguous invocation patterns that agents can follow
without inferring paths or versions.

**Rationale**: AI-assisted development is a primary use case; fragile automation
defeats the tool's purpose.

## Distribution Model

Adopting projects install `specify-run` by:

1. Copying the script to the target directory (repository root or subdirectory)
2. Making it executable (`chmod +x specify-run`)
3. Running `./specify-run` (auto-bootstraps on first run)

The script provisions SpecKit relative to its own location:
- Virtualenv creation (`.venv/` in script's directory)
- SpecKit installation from pinned ref
- `.specify/` folder created in script's directory
- All subsequent SpecKit invocations

Example: placing `specify-run` in `foo_module/` provisions SpecKit at
`foo_module/.venv/` and `foo_module/.specify/`.

## Amendment Process

1. Propose change via pull request modifying this file
2. Document rationale in PR description
3. Increment version per semantic versioning:
   - MAJOR: Backward-incompatible governance changes
   - MINOR: New principles or material expansions
   - PATCH: Clarifications or typo fixes
4. Update `LAST_AMENDED_DATE` to amendment date
5. Merge after review

## Governance

This constitution supersedes conflicting practices. All contributors and agents
MUST verify compliance before making changes. Use `CLAUDE.md` for runtime
development guidance.

**Version**: 0.1.0 | **Ratified**: 2026-01-15 | **Last Amended**: 2026-01-15
