# Implementation Plan: User Consent Prompting

**Branch**: `002-prompting` | **Date**: 2026-01-16 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/002-prompting/spec.md`

## Summary

Implement user consent prompting in `specify-run` per Principle VI. The script
must prompt before creating `.venv/` or upgrading SpecKit. Support three modes:
interactive (terminal), non-interactive (CI with `--yes`), and agentic (AI
agents via environment variables). Use BSD sysexits for distinct exit codes.

## Technical Context

**Language/Version**: Bash (POSIX-compatible with bashisms)
**Primary Dependencies**: None (self-contained script)
**Storage**: N/A (stateless, uses stamp file for version tracking)
**Testing**: Manual testing + shell script tests (bats or shellspec)
**Target Platform**: macOS, Linux (any POSIX shell environment)
**Project Type**: Single file script
**Performance Goals**: N/A (interactive prompting, no performance constraints)
**Constraints**: Must remain single-file, no external dependencies
**Scale/Scope**: Single script file (~100-150 lines added)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Single-File Distribution | ✅ Pass | All changes within `specify-run` script |
| II. Zero Configuration | ✅ Pass | No config files; env vars are optional overrides |
| III. Pinned Reproducibility | ✅ Pass | No impact on version pinning |
| IV. Explicit Upgrades | ✅ Pass | Prompting enhances upgrade visibility |
| V. Agent-Safe Design | ✅ Pass | Agentic mode provides clear env var interface |
| VI. User Consent | ✅ Pass | This feature implements Principle VI |

## Project Structure

### Documentation (this feature)

```text
specs/002-prompting/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output (minimal for bash script)
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output (N/A for bash script)
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```

### Source Code (repository root)

```text
specify-run              # Single bash script (modified in place)
```

**Structure Decision**: Single-file modification. No new directories or files
in source tree. All prompting logic added to existing `specify-run` script.

## Complexity Tracking

No violations. Implementation stays within single-file constraint.
