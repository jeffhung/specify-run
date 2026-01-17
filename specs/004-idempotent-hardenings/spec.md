# Feature Specification: Idempotent Security Hardenings

**Feature Branch**: `004-idempotent-hardenings`  
**Created**: 2026-01-17  
**Status**: Draft  
**Input**: User description: "Ensure security hardenings are applied idempotently"

## Clarifications

### Session 2026-01-17

- Q: What should happen when remediation is needed but the git working copy is
  dirty? → A: Stop immediately without applying fixes; instruct user to commit
  existing changes first or use another worktree/branch for re-hardening.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Verify Hardenings on Every Run (Priority: P1)

A user runs `./specify-run` on a project where security hardenings (e.g.,
`.gitignore` patterns) were previously configured but have been accidentally
modified or removed. The script detects the deviation, explains what is wrong,
and offers to fix it with user consent.

**Why this priority**: Core value proposition—ensures security guarantees are
continuously enforced, not just applied once during initial setup.

**Independent Test**: Run `./specify-run` after manually removing a required
`.gitignore` pattern; verify the script detects and reports the issue.

**Acceptance Scenarios**:

1. **Given** a project with `.gitignore` missing required SpecKit patterns,
   **When** user runs `./specify-run`,
   **Then** the script displays a clear message explaining the misconfiguration.

2. **Given** a project with correct `.gitignore` patterns,
   **When** user runs `./specify-run`,
   **Then** the script proceeds to SpecKit delegation without any hardening
   prompts.

---

### User Story 2 - Consent Before Remediation (Priority: P1)

When the script detects missing or incorrect hardenings, it prompts the user
for consent before making any changes. The user can approve or decline the fix.

**Why this priority**: Equal to P1 because consent is constitutionally mandated
(Principle VI) and must work in tandem with detection.

**Independent Test**: Trigger a hardening fix scenario and verify the prompt
appears before any file modification; verify declining prevents changes.

**Acceptance Scenarios**:

1. **Given** missing `.gitignore` patterns detected,
   **When** user is prompted and approves (Y),
   **Then** the script applies the fix.

2. **Given** missing `.gitignore` patterns detected,
   **When** user is prompted and declines (n),
   **Then** the script exits without modifying files.

3. **Given** agentic mode (`SPECIFYRUN_BY_AGENT=1`) with no pre-authorized answer,
   **When** remediation is needed,
   **Then** the script prints a hint and exits with code 75.

---

### User Story 3 - Stop After Remediation (Priority: P1)

After applying hardening fixes, the script stops execution and instructs the
user to commit the changes before re-running. This ensures security fixes are
committed separately from SpecKit operations.

**Why this priority**: Equal to P1 because clean commit separation is
constitutionally mandated (Principle VII).

**Independent Test**: Apply a hardening fix and verify the script exits with a
message instructing the user to commit, without delegating to SpecKit.

**Acceptance Scenarios**:

1. **Given** hardenings were just fixed with user consent,
   **When** fixes are applied,
   **Then** the script prints a message instructing the user to commit changes
   and exits without running SpecKit.

2. **Given** hardenings were just fixed,
   **When** user re-runs `./specify-run` after committing,
   **Then** the script proceeds normally to SpecKit delegation.

---

### User Story 4 - Agentic Mode Remediation (Priority: P2)

AI agents can trigger remediation by providing the appropriate answer via the
`SPECIFYRUN_ANSWERS` environment variable after receiving the exit-75 hint.

**Why this priority**: Agentic support is important but builds on the core
consent mechanism from User Stories 1-3.

**Independent Test**: Set `SPECIFYRUN_BY_AGENT=1` and verify the hint/exit-75
behavior, then retry with `SPECIFYRUN_ANSWERS` containing the fix consent.

**Acceptance Scenarios**:

1. **Given** agentic mode with `SPECIFYRUN_ANSWERS="gitignore_fix=y"`,
   **When** remediation is needed,
   **Then** the script applies the fix, prints commit instructions, and exits.

---

### User Story 5 - Block Remediation on Dirty Working Copy (Priority: P1)

When the script detects missing hardenings but the git working copy has
uncommitted changes, it refuses to apply fixes. Instead, it stops immediately
and instructs the user to commit their existing work first (or switch to another
worktree/branch for re-hardening). This ensures remediation can be committed as
a standalone commit.

**Why this priority**: P1 because clean commit separation is constitutionally
mandated (Principle VII) and dirty working copy would violate this.

**Independent Test**: Create uncommitted changes, remove a `.gitignore` pattern,
run `./specify-run`, and verify it stops without applying fixes.

**Acceptance Scenarios**:

1. **Given** remediation is needed AND git working copy is dirty,
   **When** user runs `./specify-run`,
   **Then** the script stops immediately with a message explaining the user must
   commit existing changes first before re-hardening can be applied.

2. **Given** remediation is needed AND git working copy is clean,
   **When** user runs `./specify-run`,
   **Then** the script proceeds with normal consent flow for remediation.

3. **Given** hardenings are correct AND git working copy is dirty,
   **When** user runs `./specify-run`,
   **Then** the script proceeds normally to SpecKit delegation (no remediation
   needed, so dirty state is irrelevant).

---

### Edge Cases

- What happens when `.gitignore` does not exist at all? (Script should create
  it with required patterns, following the existing bootstrap behavior.)
- What happens when the file system is read-only? (Script should warn and
  continue without modifying, or exit with a clear error.)
- What happens when the user runs the script in non-interactive mode without
  `SPECIFYRUN_ANSWERS`? (Script should exit with code 78 per existing behavior.)
- What happens when only some patterns are missing from `.gitignore`? (Script
  should detect partial compliance and offer to add only the missing patterns.)
- What happens when remediation is needed but git working copy is dirty? (Script
  stops immediately without applying fixes; user must commit first.)

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The script MUST verify all security hardenings on every execution,
  before delegating to the SpecKit command.

- **FR-002**: The script MUST detect when required `.gitignore` patterns are
  missing, modified, or incomplete.

- **FR-003**: When a deviation is detected, the script MUST display a clear,
  human-readable message explaining what is misconfigured and why it matters.

- **FR-004**: The script MUST prompt the user for consent before applying any
  hardening fixes, following existing consent mechanisms (interactive prompt,
  agentic hint with exit 75, or `SPECIFYRUN_ANSWERS` bypass).

- **FR-005**: After applying hardening fixes, the script MUST stop execution
  without delegating to SpecKit.

- **FR-006**: After applying fixes, the script MUST display a message
  instructing the user to commit the hardening changes before re-running.

- **FR-007**: If hardenings are already correct, the script MUST proceed
  silently to SpecKit delegation without any hardening-related prompts.

- **FR-008**: The script MUST support partial remediation—if only some patterns
  are missing, only those missing patterns should be added.

- **FR-009**: The hardening verification MUST be idempotent—running the script
  multiple times on a correctly configured project MUST NOT produce additional
  changes or prompts.

- **FR-010**: If remediation is needed AND the git working copy is dirty (has
  uncommitted changes), the script MUST stop immediately without applying fixes
  and display a message instructing the user to commit existing changes first
  or use another worktree/branch for re-hardening.

- **FR-011**: The dirty working copy check MUST only block remediation; if
  hardenings are already correct, the script MUST proceed to SpecKit delegation
  regardless of working copy state.

### Key Entities

- **Hardening Check**: A verification rule that defines what constitutes a
  correct security configuration (e.g., required `.gitignore` patterns).

- **Remediation Action**: A corrective action applied when a hardening check
  fails (e.g., adding missing `.gitignore` patterns).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of `./specify-run` invocations verify hardenings before
  SpecKit delegation.

- **SC-002**: When hardenings are missing, users see a clear explanation in
  under 2 seconds.

- **SC-003**: After remediation, 100% of executions stop before SpecKit
  delegation with commit instructions displayed.

- **SC-004**: On a correctly hardened project, the script adds zero overhead
  for hardening checks (verification completes in under 100ms).

- **SC-005**: Agentic users can complete remediation in a single retry cycle
  (initial run → exit 75 with hint → retry with answer → fixed and stopped).

## Assumptions

- The primary security hardening is `.gitignore` configuration. Additional
  hardenings (file permissions, etc.) may be added in future iterations.
- The existing `configure_gitignore()` function and `GITIGNORE_MARKER` provide
  the baseline for detection and remediation logic.
- The script already has working consent mechanisms (`prompt_user`, `is_agentic`,
  `SPECIFYRUN_ANSWERS`) that can be reused for hardening consent.
