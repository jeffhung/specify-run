# Implementation Plan: Idempotent Security Hardenings

**Branch**: `004-idempotent-hardenings` | **Date**: 2026-01-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/004-idempotent-hardenings/spec.md`

## Summary

Implement idempotent security hardening verification in `specify-run`. Every
execution verifies `.gitignore` patterns are correctly configured. Key behaviors:

- **Fresh provisioning** (no `.venv/`): Complete all steps in one pass
- **Gitignore remediation**: Check target dirty state, prompt, fix, stop (exit 75)
- **SpecKit upgrade**: Install new version, update project files, stop (exit 75)
- **All correct**: Proceed silently to `specify` delegation

Stop-after-fix uses exit code 75 (EX_TEMPFAIL) to signal retry needed.

## Technical Context

**Language/Version**: Bash (POSIX-compatible with bashisms)
**Primary Dependencies**: Git (for dirty check and repo detection)
**Storage**: File-based (`.gitignore` at repository root, stamp in `.venv/`)
**Testing**: Manual integration tests; shell script validation with `shellcheck`
**Target Platform**: macOS, Linux, WSL
**Project Type**: Single-file script (no separate src/tests structure)
**Performance Goals**: Verification completes in <100ms
**Constraints**: Must remain single-file, no external dependencies beyond
Python 3.9+ and Git
**Scale/Scope**: Single bash script (~300 lines)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Single-File Distribution | ✅ Pass | All changes within `specify-run` script |
| II. Zero Configuration | ✅ Pass | No new config files required |
| III. Pinned Reproducibility | ✅ Pass | No impact on version pinning |
| IV. Explicit Upgrades | ✅ Pass | No impact on upgrade mechanism |
| V. Agent-Safe Design | ✅ Pass | Uses `gitignore` key; exit 75 signals retry |
| VI. User Consent for Modifications | ✅ Pass | Prompts before applying fixes |
| VII. Idempotent Security Enforcement | ✅ Pass | Core feature; commit isolation, fresh exception, upgrade stop |

## Project Structure

### Documentation (this feature)

```text
specs/004-idempotent-hardenings/
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
