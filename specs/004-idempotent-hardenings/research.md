# Research: Idempotent Security Hardenings

**Feature**: 004-idempotent-hardenings
**Date**: 2026-01-17

## Research Topics

### 1. Hardening Target Dirty Check

**Decision**: Use `git diff --name-only` and `git diff --cached --name-only` to
check if the specific hardening target file has changes.

**Rationale**: Checking only the hardening target (`.gitignore`) rather than
the entire working copy allows remediation to proceed when other files are
dirty. This provides better UX while maintaining commit isolation.

**Alternatives considered**:
- `git status --porcelain`: Checks entire working copy; too broad
- `git status --porcelain -- .gitignore`: Works but harder to parse
- `git diff-index HEAD -- .gitignore`: Requires HEAD to exist

**Implementation**:
```bash
is_file_dirty() {
  local file="$1"
  [[ -n "$(git diff --name-only -- "$file" 2>/dev/null)" ]] ||
  [[ -n "$(git diff --cached --name-only -- "$file" 2>/dev/null)" ]]
}
```

### 2. Fresh Provisioning Detection

**Decision**: Check for `.venv/` directory existence to determine if this is
fresh provisioning or an existing environment.

**Rationale**: The presence of `.venv/` is the canonical marker for an existing
environment. If it doesn't exist, this is fresh provisioning and all setup
steps can complete in one pass.

**Implementation**:
```bash
is_fresh_provisioning() {
  [[ ! -d "$VENV" ]]
}
```

### 3. Gitignore Pattern Verification

**Decision**: Check for presence of `GITIGNORE_MARKER` and each required pattern
individually using `grep`.

**Rationale**: The existing `configure_gitignore()` function uses a marker
comment (`# specify-run`) to identify the SpecKit block. Individual pattern
checks enable partial remediation (adding only missing patterns).

**Required patterns** (stored in array):
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

### 4. Stop-After-Fix Exit Code

**Decision**: Exit with code 75 (EX_TEMPFAIL) after applying fixes.

**Rationale**: The user's intent when invoking `./specify-run` is to execute
the `specify` command. If remediation or upgrade stops before delegation, the
intended operation was NOT completed. Exit code 0 would incorrectly signal
success. Exit 75 signals "try again later" to both humans and AI agents.

**When stop-after-fix applies**:
- Gitignore remediation on existing environment
- SpecKit upgrade (scripts/templates modified)

**When stop-after-fix does NOT apply**:
- Fresh provisioning (no `.venv/`)
- Fresh SpecKit install (only `.venv/` modified, not VCS files)

### 5. SpecKit Upgrade Detection and Stop

**Decision**: Detect upgrade by comparing stamp file version with `SPECKIT_REF`.
After upgrade, stop with exit 75.

**Rationale**: SpecKit upgrade modifies VCS-tracked files (scripts, templates
in `.specify/`). These changes deserve a standalone commit, separate from any
`specify` command output.

**Implementation logic**:
```bash
if [[ ! -f "$STAMP" ]]; then
  OLD_REF=""
  for f in "$VENV"/.speckit-installed-*; do
    [[ -f "$f" ]] && OLD_REF="${f##*-installed-}" && break
  done
  if [[ -n "$OLD_REF" ]]; then
    # This is an upgrade
    prompt_user "upgrade" "About to upgrade SpecKit from $OLD_REF to $SPECKIT_REF"
    rm -f "$VENV"/.speckit-installed-*
    pip install ...
    touch "$STAMP"
    info "SpecKit upgraded. Please commit changes, then re-run ./specify-run"
    exit "$EX_TEMPFAIL"
  fi
  # Fresh install - continue
fi
```

### 6. Stamp File Location

**Decision**: Keep stamp file inside `.venv/` directory.

**Rationale**: When users delete `.venv/` to clean up the environment, the
stamp file is automatically removed. This prevents stale stamp files from
causing confusion about SpecKit installation state.

**Location**: `$VENV/.speckit-installed-$SPECKIT_REF`

### 7. Agentic Mode Answer Key

**Decision**: Use `gitignore` as the answer key for hardening consent.

**Rationale**: Single key for both initial provision and remediation simplifies
agent experience. Existing keys `bootstrap` and `upgrade` remain for venv
creation and SpecKit upgrade respectively.

**Answer keys**:
- `bootstrap`: Consent for `.venv/` creation
- `upgrade`: Consent for SpecKit upgrade
- `gitignore`: Consent for gitignore hardening

### 8. Execution Flow

**Flow diagram**:
```text
1. Sanity checks (Python exists)

2. Check if fresh provisioning (no .venv/)
   2a. If fresh:
       - Prompt for bootstrap consent (key: bootstrap)
       - Create virtualenv
       - Install SpecKit (fresh, no stop)
       - Apply hardenings (no stop)
       - Delegate to specify
   
   2b. If existing (.venv/ exists):
       - Check SpecKit version
         - If upgrade needed:
           - Prompt (key: upgrade)
           - Install new version
           - Exit 75 with commit instructions
       - Verify hardenings
         - If correct → proceed to step 3
         - If incorrect:
           - Check if .gitignore is dirty → block with message
           - Prompt for consent (key: gitignore)
           - Apply fix
           - Exit 75 with commit instructions

3. exec "$SPECIFY" "$@"
```

## Summary

Key implementation decisions:
1. Check `.gitignore` dirty state specifically, not entire working copy
2. Use `.venv/` existence to detect fresh vs. existing environment
3. Fresh provisioning completes in one pass; remediation/upgrade exit 75
4. Exit 75 signals intended operation needs retry after commit
5. Stamp file stays in `.venv/` for easy cleanup
6. Three answer keys: `bootstrap`, `upgrade`, `gitignore`
