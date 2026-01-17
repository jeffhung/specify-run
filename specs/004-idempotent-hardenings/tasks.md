# Implementation Tasks: Idempotent Security Hardenings

**Feature**: 004-idempotent-hardenings
**Date**: 2026-01-17
**Source**: [spec.md](spec.md), [plan.md](plan.md)

## Phase 0: Infrastructure Setup

### T001: Add required constants and helper functions

**User Story**: N/A (Infrastructure)
**Functional Requirements**: FR-002, FR-010, FR-011

Add the following to the `specify-run` script:

1. Add `REQUIRED_GITIGNORE_PATTERNS` array constant after `GITIGNORE_MARKER`:
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

2. Add `is_file_dirty()` helper function:
   ```bash
   is_file_dirty() {
     local file="$1"
     [[ -n "$(git diff --name-only -- "$file" 2>/dev/null)" ]] ||
     [[ -n "$(git diff --cached --name-only -- "$file" 2>/dev/null)" ]]
   }
   ```

3. Add `is_fresh_provisioning()` helper function:
   ```bash
   is_fresh_provisioning() {
     [[ ! -d "$VENV" ]]
   }
   ```

**Checklist**:
- [x] `REQUIRED_GITIGNORE_PATTERNS` array added
- [x] `is_file_dirty()` function added
- [x] `is_fresh_provisioning()` function added
- [x] Functions placed in Helpers section

---

## Phase 1: Core Detection (User Story 1)

### T002: Implement gitignore pattern verification

**User Story**: US1 - Verify Hardenings on Every Run
**Functional Requirements**: FR-001, FR-002, FR-007, FR-009

Implement `check_gitignore_hardenings()` function that:
1. Returns early if not a git repo
2. Returns success if `.gitignore` has marker and all patterns
3. Returns failure with list of missing patterns stored in variable

```bash
check_gitignore_hardenings() {
  is_git_repo || return 0
  local gitignore="$ROOT/.gitignore"
  MISSING_PATTERNS=()
  
  if [[ ! -f "$gitignore" ]]; then
    MISSING_PATTERNS=("${REQUIRED_GITIGNORE_PATTERNS[@]}")
    return 1
  fi
  
  if ! grep -q "$GITIGNORE_MARKER" "$gitignore"; then
    MISSING_PATTERNS=("${REQUIRED_GITIGNORE_PATTERNS[@]}")
    return 1
  fi
  
  for pattern in "${REQUIRED_GITIGNORE_PATTERNS[@]}"; do
    if ! grep -Fxq "$pattern" "$gitignore"; then
      MISSING_PATTERNS+=("$pattern")
    fi
  done
  
  [[ ${#MISSING_PATTERNS[@]} -eq 0 ]]
}
```

**Checklist**:
- [x] `check_gitignore_hardenings()` function implemented
- [x] Returns 0 when all patterns present
- [x] Returns 1 with `MISSING_PATTERNS` populated when patterns missing
- [x] Handles missing `.gitignore` file
- [x] Handles missing marker
- [x] Idempotent: multiple runs on correct config produce no output

---

### T003: Implement hardening status display

**User Story**: US1 - Verify Hardenings on Every Run
**Functional Requirements**: FR-003

Implement display function that shows missing patterns:

```bash
show_missing_hardenings() {
  echo "[specify-run] Security hardenings incomplete."
  echo "Missing .gitignore patterns:"
  for pattern in "${MISSING_PATTERNS[@]}"; do
    echo "  - $pattern"
  done
}
```

**Checklist**:
- [x] `show_missing_hardenings()` function implemented
- [x] Lists each missing pattern clearly
- [x] Message is human-readable

---

## Phase 2: Consent and Remediation (User Stories 2, 5)

### T004: Implement dirty target file check

**User Story**: US5 - Block Remediation on Dirty Hardening Target
**Functional Requirements**: FR-010, FR-011, FR-012

Add logic to block remediation when `.gitignore` has uncommitted changes:

```bash
block_if_target_dirty() {
  local gitignore="$ROOT/.gitignore"
  if [[ -f "$gitignore" ]] && is_file_dirty "$gitignore"; then
    echo "[specify-run] Security hardenings incomplete, but .gitignore has uncommitted changes."
    echo "Commit or revert your .gitignore changes first, then re-run."
    exit "$EX_CONFIG"
  fi
}
```

**Checklist**:
- [x] `block_if_target_dirty()` function implemented
- [x] Only checks `.gitignore` dirty state, not entire working copy
- [x] Exits with code 78 when blocked
- [x] Clear message explains what user must do

---

### T005: Implement partial remediation

**User Story**: US2 - Consent Before Remediation
**Functional Requirements**: FR-004, FR-008

Modify or create `apply_gitignore_hardenings()` to add only missing patterns:

```bash
apply_gitignore_hardenings() {
  local gitignore="$ROOT/.gitignore"
  
  if [[ ! -f "$gitignore" ]] || ! grep -q "$GITIGNORE_MARKER" "$gitignore"; then
    # Full block needed
    local block="
$GITIGNORE_MARKER
.venv/
!.specify/
!.specify/scripts/**
!.specify/templates/**
!.specify/memory/**

.specify/cache/
.specify/tmp/
.specify/logs/
.specify/.runtime/
"
    echo "$block" >> "$gitignore"
  else
    # Partial: add only missing patterns
    for pattern in "${MISSING_PATTERNS[@]}"; do
      echo "$pattern" >> "$gitignore"
    done
  fi
  
  info "Security hardenings applied."
}
```

**Checklist**:
- [x] `apply_gitignore_hardenings()` function implemented
- [x] Adds full block when marker missing
- [x] Adds only missing patterns when marker exists
- [x] Uses existing `info()` for output

---

## Phase 3: Stop After Fix (User Story 3)

### T006: Implement stop-after-fix for remediation

**User Story**: US3 - Stop After Remediation
**Functional Requirements**: FR-005, FR-006

Create function to stop after applying fixes with commit instructions:

```bash
stop_after_gitignore_fix() {
  echo "[specify-run] Security hardenings applied."
  echo "Please commit these changes, then re-run ./specify-run:"
  echo "  git add .gitignore && git commit -m \"fix: restore specify-run security hardenings\""
  exit "$EX_TEMPFAIL"
}
```

**Checklist**:
- [x] `stop_after_gitignore_fix()` function implemented
- [x] Exits with code 75
- [x] Provides complete git command for user

---

## Phase 4: Agentic Mode (User Story 4)

### T007: Ensure agentic mode works for gitignore remediation

**User Story**: US4 - Agentic Mode Remediation
**Functional Requirements**: FR-004

Verify that existing `prompt_user()` with key `gitignore` works correctly:
- First run without answer: exits 75 with hint
- Retry with `SPECIFYRUN_ANSWERS="gitignore=y"`: applies fix and stops

**Checklist**:
- [x] Agentic mode hint shows `gitignore=y` for remediation
- [x] Answer key `gitignore` is used consistently
- [x] Exit code 75 on first agentic attempt without answer

---

## Phase 5: Fresh Provisioning (User Story 6)

### T008: Implement fresh provisioning one-pass flow

**User Story**: US6 - Fresh Provisioning in One Pass
**Functional Requirements**: FR-013, FR-014

Modify the main flow so that when `.venv/` does not exist:
1. Prompt for bootstrap consent
2. Create virtualenv
3. Install SpecKit
4. Apply gitignore hardenings (no prompt, included in bootstrap)
5. Delegate to `specify` (no stop)

The existing code already does venv creation and SpecKit install. Ensure
`configure_gitignore()` is called during bootstrap and does NOT stop.

**Checklist**:
- [x] Fresh provisioning (no `.venv/`) completes in one pass
- [x] Gitignore hardenings applied during bootstrap
- [x] No exit 75 during fresh provisioning
- [x] Delegation to `specify` happens after provisioning

---

## Phase 6: SpecKit Upgrade Stop (User Story 7)

### T009: Implement stop-after-upgrade

**User Story**: US7 - Stop After SpecKit Upgrade
**Functional Requirements**: FR-015, FR-016, FR-017

Modify the SpecKit upgrade section to stop after upgrade:

```bash
if [[ ! -f "$STAMP" ]]; then
  OLD_REF=""
  for f in "$VENV"/.speckit-installed-*; do
    [[ -f "$f" ]] && OLD_REF="${f##*-installed-}" && break
  done
  if [[ -n "$OLD_REF" ]]; then
    prompt_user "upgrade" "About to upgrade SpecKit from $OLD_REF to $SPECKIT_REF"
    rm -f "$VENV"/.speckit-installed-*
    info "Installing SpecKit ($SPECKIT_REF)"
    "$PIP" install --upgrade "git+$SPECKIT_GIT@$SPECKIT_REF"
    touch "$STAMP"
    echo "[specify-run] SpecKit upgraded."
    echo "Please commit these changes, then re-run ./specify-run:"
    echo "  git add -A && git commit -m \"chore: upgrade SpecKit to $SPECKIT_REF\""
    exit "$EX_TEMPFAIL"
  fi
  # Fresh install continues...
fi
```

**Checklist**:
- [ ] Upgrade path stops with exit 75
- [ ] Commit instructions provided
- [ ] Fresh install does NOT stop
- [ ] Stamp file remains in `.venv/`

---

## Phase 7: Main Flow Integration

### T010: Integrate hardening verification into main flow

**User Story**: All
**Functional Requirements**: FR-001, FR-007

Add the main verification logic after SpecKit installation check but before
`exec "$SPECIFY" "$@"`:

```bash
# ---------------------------------------------------------------------------
# Verify security hardenings (existing environment only)
# ---------------------------------------------------------------------------

if ! is_fresh_provisioning && ! check_gitignore_hardenings; then
  show_missing_hardenings
  block_if_target_dirty
  prompt_user "gitignore" "About to restore missing .gitignore patterns"
  apply_gitignore_hardenings
  stop_after_gitignore_fix
fi
```

**Checklist**:
- [ ] Verification runs on every execution (existing environment)
- [ ] Skipped during fresh provisioning
- [ ] Correct order: detect → show → block-if-dirty → prompt → fix → stop
- [ ] Proceeds silently when hardenings correct

---

## Phase 8: Testing

### T011: Manual integration testing

**User Story**: All
**Functional Requirements**: All

Test the following scenarios manually:

1. **Fresh provisioning**: Delete `.venv/`, run `./specify-run init`
   - Expected: One pass, no stop, delegates to `specify init`

2. **Normal operation**: Run on fully provisioned environment
   - Expected: Proceeds silently to `specify`

3. **Remediation needed**: Remove a pattern from `.gitignore`
   - Expected: Detects, prompts, fixes, stops with exit 75

4. **Dirty target**: Edit `.gitignore`, remove a pattern, run
   - Expected: Blocked with message, exit 78

5. **Other dirty files**: Edit `README.md`, remove pattern from `.gitignore`
   - Expected: Remediation proceeds (not blocked)

6. **Upgrade**: Change `SPECKIT_REF`, run
   - Expected: Upgrades, stops with exit 75

7. **Agentic mode**: Set `SPECIFYRUN_BY_AGENT=1`
   - Expected: Hint with exit 75, retry with answer works

**Checklist**:
- [ ] Fresh provisioning tested
- [ ] Normal operation tested
- [ ] Remediation flow tested
- [ ] Dirty target blocking tested
- [ ] Other dirty files don't block tested
- [ ] Upgrade stop tested
- [ ] Agentic mode tested
- [ ] All exit codes verified (0, 75, 78, 130)

---

## Summary

| Phase | Tasks | User Stories Covered |
|-------|-------|---------------------|
| 0 | T001 | Infrastructure |
| 1 | T002, T003 | US1 |
| 2 | T004, T005 | US2, US5 |
| 3 | T006 | US3 |
| 4 | T007 | US4 |
| 5 | T008 | US6 |
| 6 | T009 | US7 |
| 7 | T010 | All |
| 8 | T011 | All |

**Total Tasks**: 11
**Estimated Scope**: Single-file bash script modifications (~100 lines added)
