# Implementation Plan: Preserve Memory on Upgrade

**Branch**: `005-preserve-memory-upgrade` | **Date**: 2026-01-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/005-preserve-memory-upgrade/spec.md`

## Summary

Implement differentiated file handling during SpecKit upgrades: preserve project
memory files (`.specify/memory/`) while unconditionally replacing scripts and
templates (`.specify/scripts/`, `.specify/templates/`) with versions from the
new SpecKit release. This prevents loss of accumulated project knowledge while
ensuring auditable, tamper-free tooling.

## Technical Context

**Language/Version**: Bash (POSIX-compatible with bashisms)
**Primary Dependencies**: Git, Python 3.9+ (for SpecKit), pip
**Storage**: File-based (`.specify/` directory hierarchy)
**Testing**: Manual integration tests; shell script validation with `shellcheck`
**Target Platform**: macOS, Linux, WSL
**Project Type**: Single-file script (no separate src/tests structure)
**Performance Goals**: Upgrade completes in <30 seconds (network-bound)
**Constraints**: Must remain single-file, no external dependencies beyond
Python 3.9+ and Git
**Scale/Scope**: Single bash script (~350 lines)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Single-File Distribution | ✅ Pass | All changes within `specify-run` script |
| II. Zero Configuration | ✅ Pass | No new config files required |
| III. Pinned Reproducibility | ✅ Pass | No impact on version pinning |
| IV. Explicit Upgrades | ✅ Pass | Upgrade triggered by `SPECKIT_REF` change |
| V. Agent-Safe Design | ✅ Pass | Upgrade behavior is deterministic |
| VI. User Consent for Modifications | ✅ Pass | Upgrade already requires `upgrade=y` consent |
| VII. Idempotent Security Enforcement | ✅ Pass | No impact on security hardenings |
| VIII. Upgrade File Handling | ✅ Pass | Core feature; implements this principle |

## Project Structure

### Documentation (this feature)

```text
specs/005-preserve-memory-upgrade/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```

### Source Code (repository root)

```text
specify-run              # Single-file bash script (modified in place)
```

**Structure Decision**: This is a single-file script project. No separate
`src/` or `tests/` directories. All implementation changes go directly into
the `specify-run` bash script.

## Complexity Tracking

No constitution violations. No complexity justifications needed.
