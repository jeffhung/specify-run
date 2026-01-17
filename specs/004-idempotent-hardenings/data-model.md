# Data Model: Idempotent Security Hardenings

**Feature**: 004-idempotent-hardenings
**Date**: 2026-01-17

## Overview

This feature operates on file-based configuration. No database entities or
complex data structures are involved. The "data model" consists of:

1. **Configuration constants** in the bash script
2. **File-based state** (`.gitignore` content, stamp file)
3. **Git working copy state** (file-specific dirty check)

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

### Environment State

The state of the `specify-run` environment.

**States**:
- `FRESH`: No `.venv/` directory exists
- `EXISTING`: `.venv/` exists

**Detection**:
```bash
is_fresh_provisioning() {
  [[ ! -d "$VENV" ]]
}
```

### SpecKit Installation State

The state of SpecKit installation within the environment.

**States**:
- `NOT_INSTALLED`: No stamp file exists (and no old stamp files)
- `INSTALLED`: Current stamp file exists (`$VENV/.speckit-installed-$SPECKIT_REF`)
- `NEEDS_UPGRADE`: Old stamp file exists but not current version

**Detection**:
```bash
get_speckit_state() {
  if [[ -f "$STAMP" ]]; then
    echo "INSTALLED"
  elif ls "$VENV"/.speckit-installed-* >/dev/null 2>&1; then
    echo "NEEDS_UPGRADE"
  else
    echo "NOT_INSTALLED"
  fi
}
```

### Hardening State

The state of security hardenings.

**States**:
- `COMPLETE`: All patterns present, marker exists
- `INCOMPLETE`: Some patterns missing or marker absent
- `NO_GITIGNORE`: The `.gitignore` file does not exist

### File Dirty State

Whether a specific file has uncommitted changes.

**States**:
- `CLEAN`: No staged or unstaged changes to the file
- `DIRTY`: File has staged or unstaged changes

**Detection**:
```bash
is_file_dirty() {
  local file="$1"
  [[ -n "$(git diff --name-only -- "$file" 2>/dev/null)" ]] ||
  [[ -n "$(git diff --cached --name-only -- "$file" 2>/dev/null)" ]]
}
```

## State Transitions

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Script Start                                   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Check Environment   │
                         │ State (.venv/)      │
                         └─────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
              ┌──────────┐                   ┌────────────┐
              │  FRESH   │                   │  EXISTING  │
              └──────────┘                   └────────────┘
                    │                               │
                    ▼                               ▼
           ┌────────────────┐             ┌─────────────────┐
           │ Prompt:        │             │ Check SpecKit   │
           │ bootstrap      │             │ State           │
           └────────────────┘             └─────────────────┘
                    │                               │
                    ▼                     ┌─────────┼─────────┐
           ┌────────────────┐             ▼         ▼         ▼
           │ Create .venv   │      ┌──────────┐ ┌────────┐ ┌───────────┐
           │ Install SpecKit│      │INSTALLED │ │UPGRADE │ │NOT_INSTALL│
           │ Apply hardenings│     └──────────┘ └────────┘ └───────────┘
           │ (no stop)      │             │         │             │
           └────────────────┘             │         ▼             ▼
                    │                     │  ┌────────────┐  ┌──────────┐
                    │                     │  │ Prompt:    │  │ Install  │
                    │                     │  │ upgrade    │  │ SpecKit  │
                    │                     │  └────────────┘  │ (no stop)│
                    │                     │         │        └──────────┘
                    │                     │         ▼             │
                    │                     │  ┌────────────┐       │
                    │                     │  │ Upgrade    │       │
                    │                     │  │ SpecKit    │       │
                    │                     │  └────────────┘       │
                    │                     │         │             │
                    │                     │         ▼             │
                    │                     │  ┌────────────┐       │
                    │                     │  │ Exit 75    │       │
                    │                     │  │ (commit)   │       │
                    │                     │  └────────────┘       │
                    │                     │                       │
                    │                     ▼                       ▼
                    │              ┌─────────────────────────────────┐
                    │              │ Check Hardening State           │
                    │              └─────────────────────────────────┘
                    │                               │
                    │              ┌────────────────┴────────────────┐
                    │              ▼                                 ▼
                    │       ┌────────────┐                    ┌────────────┐
                    │       │ COMPLETE   │                    │ INCOMPLETE │
                    │       └────────────┘                    └────────────┘
                    │              │                                 │
                    │              │                                 ▼
                    │              │                    ┌─────────────────────┐
                    │              │                    │ Check .gitignore    │
                    │              │                    │ Dirty State         │
                    │              │                    └─────────────────────┘
                    │              │                         │           │
                    │              │                         ▼           ▼
                    │              │                   ┌─────────┐  ┌─────────┐
                    │              │                   │ DIRTY   │  │ CLEAN   │
                    │              │                   └─────────┘  └─────────┘
                    │              │                         │           │
                    │              │                         ▼           ▼
                    │              │                   ┌─────────┐  ┌──────────┐
                    │              │                   │ Block   │  │ Prompt:  │
                    │              │                   │ (msg)   │  │ gitignore│
                    │              │                   └─────────┘  └──────────┘
                    │              │                                      │
                    │              │                                      ▼
                    │              │                               ┌──────────┐
                    │              │                               │ Apply    │
                    │              │                               │ Fix      │
                    │              │                               └──────────┘
                    │              │                                      │
                    │              │                                      ▼
                    │              │                               ┌──────────┐
                    │              │                               │ Exit 75  │
                    │              │                               │ (commit) │
                    │              │                               └──────────┘
                    │              │
                    └──────────────┴──────────────────────┐
                                                          ▼
                                              ┌─────────────────────┐
                                              │ Delegate to specify │
                                              │ exec "$SPECIFY" "$@"│
                                              └─────────────────────┘
```

## Validation Rules

1. **Marker must exist**: If `.gitignore` has SpecKit patterns, it MUST have
   the `# specify-run` marker.

2. **All patterns required**: Each pattern in `REQUIRED_GITIGNORE_PATTERNS`
   MUST be present (exact string match per line).

3. **Order not enforced**: Patterns can appear in any order within the file.

4. **User additions allowed**: Additional patterns outside the SpecKit block
   are permitted and preserved.

5. **File-specific dirty check**: Only the hardening target file's dirty state
   blocks remediation; other files may be dirty.
