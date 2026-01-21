# Research: Script Versioning

**Feature**: 006-script-versioning
**Date**: 2026-01-22

## Overview

This feature adds version tracking to the specify-run wrapper script. The
implementation is straightforward with no external dependencies or complex
design decisions required.

## Research Topics

### 1. Version Display Format

**Decision**: Display wrapper version on a separate line before SpecKit version.

**Rationale**: Clear visual separation between wrapper and SpecKit versions.
Users can easily identify which component each version refers to.

**Format**:
```
specify-run 0.6.0
[SpecKit version output follows]
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

### 3. Version Command Detection

**Decision**: Check if first argument is `version` before delegating to SpecKit.

**Rationale**: Simple conditional check that intercepts only the version
command. All other commands pass through unchanged.

**Implementation approach**:
```bash
if [[ "${1:-}" == "version" ]]; then
  echo "specify-run $SPECIFYRUN_VERSION"
fi
exec "$SPECIFY" "$@"
```

**Alternatives considered**:
- Using getopts for --version flag - rejected per spec (use subcommand instead)
- Checking after bootstrap - current approach already does this

### 4. Semantic Versioning Compliance

**Decision**: Use standard MAJOR.MINOR.PATCH format without pre-release or
build metadata.

**Rationale**: Simple version format matches project's current maturity level.
Pre-release suffixes can be added later if needed.

**Starting version**: 0.6.0 (as specified by user, reflecting 5 prior SpecKit
iterations)

## No Clarifications Needed

All technical decisions have clear best practices. No NEEDS CLARIFICATION
markers from Technical Context require resolution.
