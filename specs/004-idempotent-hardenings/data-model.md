# Data Model: Idempotent Security Hardenings

**Feature**: 004-idempotent-hardenings
**Date**: 2026-01-17

## Overview

This feature operates on file-based configuration. No database entities or
complex data structures are involved. The "data model" consists of:

1. **Configuration constants** in the bash script
2. **File-based state** (`.gitignore` content)
3. **Git working copy state** (clean/dirty)

## Entities

### Required Gitignore Patterns

An array of patterns that MUST be present in `.gitignore` for the security
hardening to be considered complete.

**Definition** (bash array):
```bash
REQUIRED_GITIGNORE_PATTERNS=(
  ".venv/"
  "!.specify/"
  "!.specify/scripts/**"
  "!.specify/templates/**"
  "!.specify/memory/**"
  ".specify/cache/"
  ".specify/tmp/"
  ".specify/logs/"
  ".specify/.runtime/"
)
```

**Marker**: `# specify-run` (identifies the SpecKit-managed block)

### Hardening Check Result

The result of verifying all required patterns.

**States**:
- `COMPLETE`: All patterns present, marker exists
- `MISSING_MARKER`: The marker comment is not in `.gitignore`
- `MISSING_PATTERNS`: Marker exists but some patterns are missing
- `NO_GITIGNORE`: The `.gitignore` file does not exist

**Representation** (bash function return + global array):
```bash
MISSING_PATTERNS=()  # Populated by check function

check_gitignore_hardenings() {
  # Returns 0 if complete, 1 if needs remediation
  # Populates MISSING_PATTERNS array with missing items
}
```

### Working Copy State

**States**:
- `CLEAN`: No uncommitted changes (safe to apply fixes)
- `DIRTY`: Uncommitted changes exist (block remediation)
- `NOT_GIT`: Not a git repository (skip dirty check)

**Detection**:
```bash
is_working_copy_dirty() {
  [[ -n "$(git status --porcelain 2>/dev/null)" ]]
}
```

## State Transitions

```text
┌─────────────────────────────────────────────────────────────────┐
│                         Script Start                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │ Check Hardenings │
                    └──────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
         ┌────────┐     ┌──────────┐    ┌──────────────┐
         │Complete│     │ Missing  │    │ Not Git Repo │
         └────────┘     └──────────┘    └──────────────┘
              │               │               │
              │               ▼               │
              │      ┌─────────────────┐      │
              │      │Check Dirty State│      │
              │      └─────────────────┘      │
              │         │           │         │
              │         ▼           ▼         │
              │    ┌────────┐  ┌───────┐      │
              │    │ Dirty  │  │ Clean │      │
              │    └────────┘  └───────┘      │
              │         │           │         │
              │         ▼           ▼         │
              │    ┌────────┐  ┌────────┐     │
              │    │  STOP  │  │Prompt  │     │
              │    │(msg)   │  │Consent │     │
              │    └────────┘  └────────┘     │
              │                     │         │
              │         ┌───────────┼─────────┤
              │         ▼           ▼         │
              │    ┌────────┐  ┌────────┐     │
              │    │Declined│  │Approved│     │
              │    │(exit)  │  └────────┘     │
              │    └────────┘       │         │
              │                     ▼         │
              │               ┌──────────┐    │
              │               │Apply Fix │    │
              │               └──────────┘    │
              │                     │         │
              │                     ▼         │
              │               ┌──────────┐    │
              │               │  STOP    │    │
              │               │(commit)  │    │
              │               └──────────┘    │
              │                               │
              └───────────────┬───────────────┘
                              ▼
                    ┌──────────────────┐
                    │ Delegate to      │
                    │ SpecKit          │
                    └──────────────────┘
```

## Validation Rules

1. **Marker must exist**: If `.gitignore` exists and contains SpecKit patterns,
   it MUST have the `# specify-run` marker.

2. **All patterns required**: Each pattern in `REQUIRED_GITIGNORE_PATTERNS`
   MUST be present (exact string match per line).

3. **Order not enforced**: Patterns can appear in any order within the file.

4. **User additions allowed**: Additional patterns outside the SpecKit block
   are permitted and preserved.
