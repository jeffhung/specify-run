# Research: Idempotent Security Hardenings

**Feature**: 004-idempotent-hardenings
**Date**: 2026-01-17

## Research Topics

### 1. Git Dirty Working Copy Detection

**Decision**: Use `git status --porcelain` to detect uncommitted changes.

**Rationale**: The `--porcelain` flag provides machine-readable output that is
stable across Git versions. An empty output means clean working copy; any
output means dirty.

**Alternatives considered**:
- `git diff --quiet`: Only detects tracked file changes, misses untracked files
- `git status -s`: Similar to porcelain but less stable format guarantees
- `git diff-index HEAD`: Requires HEAD to exist, fails on initial commit

**Implementation**:
```bash
is_working_copy_dirty() {
  [[ -n "$(git status --porcelain 2>/dev/null)" ]]
}
```

### 2. Gitignore Pattern Verification

**Decision**: Check for presence of `GITIGNORE_MARKER` and each required pattern
individually.

**Rationale**: The existing `configure_gitignore()` function uses a marker
comment (`# specify-run`) to identify the SpecKit block. Verifying the marker
confirms the block was added. Individual pattern checks enable partial
remediation (adding only missing patterns).

**Alternatives considered**:
- Hash-based verification: Too strict; rejects user additions to `.gitignore`
- Line-by-line exact match: Breaks if user reorders or adds comments
- Marker-only check: Current approach; simple but doesn't detect partial removal

**Implementation approach**:
1. Check if `GITIGNORE_MARKER` exists in `.gitignore`
2. Check each required pattern (`.venv/`, `!.specify/`, etc.)
3. Report which specific patterns are missing
4. Offer to add only missing patterns (partial remediation)

**Required patterns**:
```text
# specify-run
.venv/
!.specify/
!.specify/scripts/**
!.specify/templates/**
!.specify/memory/**

.specify/cache/
.specify/tmp/
.specify/logs/
.specify/.runtime/
```

### 3. Stop-After-Fix Behavior

**Decision**: Exit with code 0 after applying fixes, with clear message.

**Rationale**: Exit code 0 indicates successful operation (the fix was applied).
The message clearly instructs the user to commit before re-running. Using a
non-zero exit code would be misleading since no error occurred.

**Alternatives considered**:
- Exit with code 1: Implies error, but fix succeeded
- Exit with special code (e.g., 76): Non-standard, harder to document
- Continue to SpecKit: Violates Principle VII (clean commit separation)

**Message format**:
```text
[specify-run] Security hardenings applied.
Please commit these changes before re-running:
  git add .gitignore && git commit -m "fix: restore specify-run security hardenings"
```

### 4. Agentic Mode Answer Key

**Decision**: Use `gitignore_fix` as the answer key for remediation consent.

**Rationale**: Consistent with existing pattern (`bootstrap`, `upgrade`). Clear
and descriptive name that agents can understand from the hint message.

**Hint message**:
```text
Append `gitignore_fix=y` to SPECIFYRUN_ANSWERS environment variable to proceed.
```

### 5. Execution Flow

**Decision**: Insert hardening verification after venv/SpecKit setup, before
`exec "$SPECIFY"`.

**Rationale**: Hardenings must be verified on every run (Principle VII), but
only after the environment is ready. Placing the check just before delegation
ensures:
1. Venv exists (needed for SpecKit, not for hardening check)
2. SpecKit is installed (ensures stamp file logic runs first)
3. Hardenings verified before any SpecKit operation

**Flow diagram**:
```text
1. Sanity checks (Python exists)
2. Ensure virtualenv (prompt if needed)
3. Ensure SpecKit installed (prompt if upgrade)
4. [NEW] Verify security hardenings
   4a. If all patterns present → continue
   4b. If patterns missing:
       - If dirty working copy → stop with message (no fix)
       - If clean → prompt for consent → apply fix → stop with commit message
5. exec "$SPECIFY" "$@"
```

## Summary

No external research needed. All decisions use existing bash/git patterns
already present in the codebase. Implementation is straightforward modification
of the existing `specify-run` script.
