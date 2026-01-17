<!--
Sync Impact Report
==================
Version change: 0.3.0 → 0.3.1 (PATCH: clarification to Principle VII)
Modified sections:
  - Principle VII: Clarified commit isolation requirements and fresh provisioning exception
Added sections: None
Removed sections: None
Templates requiring updates:
  - plan-template.md: ✅ Compatible (no changes needed)
  - spec-template.md: ✅ Compatible (no changes needed)
  - tasks-template.md: ✅ Compatible (no changes needed)
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

### VI. User Consent for Modifications

When `specify-run` itself (not the delegated SpecKit command) intends to modify
any file or directory in the project, it MUST first prompt the user with a clear
explanation of what will be performed and why. The script MUST only proceed if
the user explicitly agrees.

This requirement does NOT apply when execution is delegated to the SpecKit
command (`specify`), as SpecKit has its own prompting logic.

**Rationale**: Explicit user consent prevents unexpected changes to the project
and maintains trust. Users must understand and approve modifications before they
occur.

### VII. Idempotent Security Enforcement

Every execution of `specify-run` MUST verify that all security hardenings are
in place before delegating to the SpecKit command. The script MUST reach an
expected secure state regardless of the previous environment state.

If hardenings are missing or have been modified (e.g., by accidental human
edits), the script MUST:

1. Detect the deviation from the expected secure configuration
2. Display a clear message explaining what is misconfigured and why it matters
3. Prompt the user for consent before applying fixes (per Principle VI)
4. Apply the fixes upon consent
5. **Stop execution without delegating to SpecKit**, displaying a message that
   instructs the user to commit the hardening changes first

**Commit Isolation (Remediation)**: When fixing drifted hardenings on an
existing environment, changes made by `specify-run` MUST NOT mix with other
changes in the same commit:

- **Project changes**: If the hardening target file (e.g., `.gitignore`) has
  staged or unstaged changes, the script MUST block remediation until the user
  commits or reverts those changes. Other dirty files do not block remediation
  as long as the hardening target is clean.
- **SpecKit changes**: After applying fixes, the script MUST stop before
  delegating to `specify`, ensuring hardening changes are committed separately
  from any changes the SpecKit command might make.

The script MUST NOT commit changes automatically. It only modifies files and
instructs the user to commit separately.

**Fresh Provisioning Exception**: When provisioning a fresh environment (e.g.,
`./specify-run init` with no existing `.venv/`), all setup steps MAY complete
in one pass:

- Create virtualenv
- Install SpecKit
- Apply security hardenings
- Delegate to `specify`

This is acceptable because initial setup changes logically belong together in
one commit. The commit isolation rule applies only to remediation of drifted
hardenings, not initial provisioning.

**Rationale**: Security configurations can drift over time due to accidental
modifications. Idempotent enforcement ensures every invocation verifies the
expected secure baseline. Commit isolation during remediation enables proper
review and rollback of each concern independently: (a) prior project work,
(b) hardening fixes, (c) SpecKit operations. Fresh provisioning is exempt
because all changes are part of a single logical setup operation.

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

**Version**: 0.3.1 | **Ratified**: 2026-01-15 | **Last Amended**: 2026-01-17
