# Quickstart: Script Versioning

**Feature**: 006-script-versioning
**Date**: 2026-01-22

## Overview

This guide covers implementing version display for the specify-run wrapper
script.

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

### Step 2: Add Version Display Logic

Add version check before the final `exec` statement:

```bash
# ---------------------------------------------------------------------------
# Version display
# ---------------------------------------------------------------------------

if [[ "${1:-}" == "version" ]]; then
  echo "specify-run $SPECIFYRUN_VERSION"
fi

# ---------------------------------------------------------------------------
# Exec
# ---------------------------------------------------------------------------

exec "$SPECIFY" "$@"
```

## Verification

### Test 1: Version Command

```bash
./specify-run version
```

**Expected output**:
```
specify-run 0.6.0
SpecKit version 0.0.90
```

### Test 2: Other Commands Unchanged

```bash
./specify-run init --help
```

**Expected**: Normal SpecKit help output without wrapper version.

### Test 3: Version Format Validation

```bash
./specify-run version | head -1 | grep -E '^specify-run [0-9]+\.[0-9]+\.[0-9]+$'
```

**Expected**: Exit code 0 (pattern matches).

## Files Modified

| File | Change |
|------|--------|
| `specify-run` | Add `SPECIFYRUN_VERSION` variable and version display logic |

## Rollback

Remove the `SPECIFYRUN_VERSION` variable and the version display `if` block.
