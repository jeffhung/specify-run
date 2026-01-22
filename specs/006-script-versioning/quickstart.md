# Quickstart: Script Versioning

**Feature**: 006-script-versioning
**Date**: 2026-01-22

## Overview

This guide covers implementing version display for the specify-run wrapper
script. The version command MUST have no side effects - it displays versions
without triggering bootstrap, hardenings, or SpecKit installation.

## Implementation Steps

### Step 1: Add Version Variable

Add `SPECIFYRUN_VERSION` to the Configuration section of `specify-run`:

```bash
# ---------------------------------------------------------------------------
# Configuration (PIN HERE)
# ---------------------------------------------------------------------------

# Wrapper version (Semantic Versioning)
SPECIFYRUN_VERSION="0.6.0"

# Pin SpecKit source and ref
SPECKIT_GIT="https://github.com/github/spec-kit.git"
SPECKIT_REF="v0.0.90"
```

### Step 2: Add Early Exit Version Logic

Add version check EARLY in the script, immediately after the REQUIRED_GITIGNORE_PATTERNS
definition and BEFORE any bootstrap/hardening logic:

```bash
# ---------------------------------------------------------------------------
# Version command (early exit, no side effects)
# ---------------------------------------------------------------------------

if [[ "${1:-}" == "version" ]]; then
  echo "specify-run $SPECIFYRUN_VERSION"
  if [[ -x "$VENV/bin/specify" ]]; then
    exec "$VENV/bin/specify" version
  fi
  exit 0
fi
```

**Key placement**: This block MUST appear before:
- Python/command checks
- Virtualenv creation
- SpecKit installation
- Security hardenings verification

## Verification

### Test 1: Version Command (SpecKit Installed)

```bash
./specify-run version
```

**Expected output**:
```
specify-run 0.6.0
SpecKit version 0.0.90
```

### Test 2: Version Command (Fresh Environment - No SpecKit)

```bash
# In a fresh clone without .venv/
./specify-run version
```

**Expected output**:
```
specify-run 0.6.0
```

**Expected behavior**: NO bootstrap prompt, NO .venv creation, NO network calls.

### Test 3: Other Commands Unchanged

```bash
./specify-run init --help
```

**Expected**: Normal SpecKit help output without wrapper version prefix.

### Test 4: Version Format Validation

```bash
./specify-run version | head -1 | grep -E '^specify-run [0-9]+\.[0-9]+\.[0-9]+$'
```

**Expected**: Exit code 0 (pattern matches).

### Test 5: No Side Effects Verification

```bash
# Remove .venv if exists, then run version
rm -rf .venv
./specify-run version
ls .venv 2>/dev/null && echo "FAIL: .venv created" || echo "PASS: No .venv"
```

**Expected**: "PASS: No .venv" - version command should not create .venv.

## Files Modified

| File | Change |
|------|--------|
| `specify-run` | Add `SPECIFYRUN_VERSION` variable and early-exit version logic |

## Rollback

Remove the `SPECIFYRUN_VERSION` variable and the early-exit version `if` block.
