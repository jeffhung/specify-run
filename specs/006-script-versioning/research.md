# Research: Script Versioning

**Feature**: 006-script-versioning
**Date**: 2026-01-22

## Overview

This feature adds version tracking to the specify-run wrapper script. The
implementation requires early exit logic to ensure no side effects occur when
the version command is invoked.

## Research Topics

### 1. Version Display Format

**Decision**: Display wrapper version on a separate line before SpecKit version.

**Rationale**: Clear visual separation between wrapper and SpecKit versions.
Users can easily identify which component each version refers to.

**Format**:
```
specify-run 0.6.0
[SpecKit version output follows, if installed]
```

**Alternatives considered**:
- Inline format (e.g., "specify-run 0.6.0 / SpecKit 0.0.90") - rejected as it
  would require parsing SpecKit output
- JSON format - rejected as overkill for simple version display

### 2. Version Variable Placement

**Decision**: Place `SPECIFYRUN_VERSION` in the Configuration section alongside
`SPECKIT_GIT` and `SPECKIT_REF`.

**Rationale**: Groups all version-related configuration in one location.
Maintainers updating SpecKit ref can easily see and update wrapper version.

**Alternatives considered**:
- Separate "Wrapper Configuration" section - rejected as unnecessary separation
- At top of file before shebang - not valid bash syntax

### 3. Version Command Detection - Early Exit Strategy

**Decision**: Check if first argument is `version` immediately after variable
initialization and before any bootstrap/hardening logic.

**Rationale**: The version command must have zero side effects per FR-003. By
checking early and exiting before any file system operations, we guarantee no
bootstrap, no hardenings, and no SpecKit installation is triggered.

**Implementation approach**:
```bash
# Early exit for version command (no side effects)
if [[ "${1:-}" == "version" ]]; then
  echo "specify-run $SPECIFYRUN_VERSION"
  if [[ -x "$VENV/bin/specify" ]]; then
    exec "$VENV/bin/specify" version
  fi
  exit 0
fi
```

**Key design points**:
- Check happens BEFORE bootstrap logic
- Only delegates to `specify version` if executable exists
- Exits with 0 after displaying wrapper version (even without SpecKit)
- No prompts, no file modifications, no network calls

**Alternatives considered**:
- Checking after bootstrap - rejected per clarification (must have no side
  effects)
- Using a flag like --version - rejected per spec (use subcommand instead)

### 4. Semantic Versioning Compliance

**Decision**: Use standard MAJOR.MINOR.PATCH format without pre-release or
build metadata.

**Rationale**: Simple version format matches project's current maturity level.
Pre-release suffixes can be added later if needed.

**Starting version**: 0.6.0 (as specified by user, reflecting 5 prior SpecKit
iterations)

### 5. Fresh Environment Behavior

**Decision**: Display only wrapper version when SpecKit is not installed.

**Rationale**: Per FR-003, no bootstrap should occur. Users see what's available
without triggering installation. This is useful for:
- Verifying specify-run is correctly placed in a project
- Checking wrapper version before deciding to bootstrap
- Quick version check in CI/scripts without side effects

**Output in fresh environment**:
```
specify-run 0.6.0
```

(No error message, no SpecKit version - clean output)

## No Clarifications Needed

All technical decisions have clear best practices. No NEEDS CLARIFICATION
markers from Technical Context require resolution.
