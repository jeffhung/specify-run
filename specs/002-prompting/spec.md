# Feature Specification: User Consent Prompting

**Feature Branch**: `002-prompting`  
**Created**: 2026-01-16  
**Status**: Draft  
**Input**: User description: "Align with Principle VI - prompting"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - First-Run Bootstrap Consent (Priority: P1)

A user runs `./specify-run` for the first time in a project directory. The
script needs to create a `.venv/` directory and install SpecKit. Before making
any changes, the script displays what it will do and asks the user to confirm.

**Why this priority**: This is the most common scenario where `specify-run`
modifies the filesystem. Users must understand what happens on first run.

**Independent Test**: Run `./specify-run` in a clean directory without `.venv/`
and verify the prompt appears before any directories are created.

**Acceptance Scenarios**:

1. **Given** a directory without `.venv/`, **When** the user runs `./specify-run`,
   **Then** the script displays a message explaining it will create `.venv/` and
   install SpecKit, and waits for user confirmation.

2. **Given** a first-run prompt is displayed, **When** the user confirms (e.g.,
   types "y" or presses Enter), **Then** the script proceeds with installation.

3. **Given** a first-run prompt is displayed, **When** the user declines (e.g.,
   types "n"), **Then** the script exits without making any changes.

---

### User Story 2 - SpecKit Upgrade Consent (Priority: P2)

A user edits `SPECKIT_REF` to upgrade SpecKit. On next run, the script detects
the version mismatch and needs to reinstall. Before modifying `.venv/`, it
prompts the user to confirm the upgrade.

**Why this priority**: Upgrades are less frequent than first runs but equally
important for user awareness of changes.

**Independent Test**: Change `SPECKIT_REF` in the script and run it; verify the
upgrade prompt appears before any package installation occurs.

**Acceptance Scenarios**:

1. **Given** `.venv/` exists with a different SpecKit version stamp, **When**
   the user runs `./specify-run`, **Then** the script displays a message
   explaining the version change and asks for confirmation.

2. **Given** an upgrade prompt is displayed, **When** the user confirms, **Then**
   the script installs the new SpecKit version.

3. **Given** an upgrade prompt is displayed, **When** the user declines, **Then**
   the script exits without modifying the environment (user keeps old version).

---

### User Story 3 - Delegated Execution Without Prompting (Priority: P3)

After the environment is set up, running `./specify-run <command>` delegates to
SpecKit. The wrapper script makes no modifications and therefore shows no
prompts—SpecKit handles its own prompting internally.

**Why this priority**: Ensures the prompting requirement doesn't interfere with
normal SpecKit usage after initial setup.

**Independent Test**: Run `./specify-run init` after setup is complete; verify
no `specify-run` prompts appear (only SpecKit's own prompts if any).

**Acceptance Scenarios**:

1. **Given** `.venv/` exists with correct SpecKit version, **When** the user
   runs `./specify-run <any-command>`, **Then** the script passes through to
   SpecKit without displaying any `specify-run` prompts.

---

### Edge Cases

- What happens when the user runs in non-interactive mode (e.g., CI)?
  The script detects non-interactive terminals and requires an explicit `--yes`
  flag to proceed, otherwise exits with an error message.
- What happens if the user interrupts during the confirmation prompt?
  The script exits cleanly without partial modifications.
- What happens if `.venv/` exists but is corrupted or incomplete?
  The script prompts before attempting to recreate it.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The script MUST display a clear description of pending filesystem
  modifications before creating `.venv/` on first run.
- **FR-002**: The script MUST display a clear description of pending package
  changes before upgrading SpecKit when `SPECKIT_REF` changes.
- **FR-003**: The script MUST wait for explicit user confirmation before
  proceeding with any modifications.
- **FR-004**: The script MUST exit without making changes if the user declines
  confirmation.
- **FR-005**: The script MUST NOT prompt when delegating execution to SpecKit
  (SpecKit handles its own prompting).
- **FR-006**: The script MUST provide a `--yes` or `-y` flag to skip prompts
  for non-interactive environments (CI, automation).
- **FR-007**: The script MUST detect non-interactive terminals and exit with
  an error message indicating `--yes` is required.

### Key Entities

- **Prompt Message**: Text explaining what modification will occur and why.
- **Confirmation Input**: User's response (yes/no) to proceed.
- **Modification Type**: Category of change (first-run setup, version upgrade).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users always see a prompt before any `.venv/` creation or
  modification occurs.
- **SC-002**: 100% of filesystem modifications by `specify-run` are preceded
  by user consent (excluding delegated SpecKit operations).
- **SC-003**: Users can run the script in CI/automation using `--yes` without
  interactive prompts.
- **SC-004**: Users can decline any modification and exit cleanly with no
  partial changes.

## Assumptions

- The script runs in a terminal environment where stdin/stdout are available
  for interactive prompting.
- Non-interactive detection uses standard terminal detection (e.g., `[ -t 0 ]`
  in bash).
- The `--yes` flag is consistent with common CLI conventions for bypassing
  prompts.
