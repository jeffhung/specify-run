# Feature Specification: Preserve Memory on Upgrade

**Feature Branch**: `005-preserve-memory-upgrade`  
**Created**: 2026-01-17  
**Status**: Draft  
**Input**: User description: "Ensure project memory are not affected by SpecKit upgrade"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Memory Files Preserved After Upgrade (Priority: P1)

A user upgrades SpecKit by changing `SPECKIT_REF` and re-running `./specify-run`.
After the upgrade completes, all files in `.specify/memory/` remain unchanged from
before the upgrade. The project's accumulated context and learnings are preserved.

**Why this priority**: Memory files contain irreplaceable project-specific knowledge
accumulated over time. Losing this data would require rebuilding context from scratch.

**Independent Test**: Change `SPECKIT_REF` to a new version, run `./specify-run`,
verify all memory files have identical content before and after upgrade.

**Acceptance Scenarios**:

1. **Given** a project with existing memory files in `.specify/memory/`,
   **When** user upgrades SpecKit by changing `SPECKIT_REF` and running `./specify-run`,
   **Then** all memory files remain unchanged after the upgrade.

2. **Given** a project with custom memory files added by the user,
   **When** SpecKit upgrade occurs,
   **Then** custom memory files are preserved alongside SpecKit-managed memory files.

3. **Given** the new SpecKit version includes new memory file templates,
   **When** upgrade occurs on a project with existing memory files,
   **Then** existing files are preserved and new templates are NOT applied.

---

### User Story 2 - Scripts and Templates Replaced on Upgrade (Priority: P1)

When SpecKit is upgraded, all files in `.specify/scripts/` and `.specify/templates/`
are replaced with the versions from the new SpecKit release. Local modifications
are discarded without prompting.

**Why this priority**: Scripts and templates are executable tooling where local
modifications could indicate tampering or drift from audited behavior.

**Independent Test**: Modify a script file locally, upgrade SpecKit, verify the
file is replaced with the new version from SpecKit.

**Acceptance Scenarios**:

1. **Given** a project with unmodified scripts in `.specify/scripts/`,
   **When** SpecKit is upgraded to a new version,
   **Then** scripts are replaced with the new version's scripts.

2. **Given** a project with locally modified scripts in `.specify/scripts/`,
   **When** SpecKit is upgraded,
   **Then** local modifications are discarded and scripts are replaced with new version.

3. **Given** a project with locally modified templates in `.specify/templates/`,
   **When** SpecKit is upgraded,
   **Then** local modifications are discarded and templates are replaced.

---

### User Story 3 - Clear Upgrade Messages (Priority: P2)

When SpecKit is upgraded, the script displays clear messages indicating what
file operations occurred, distinguishing between preserved and replaced files.

**Why this priority**: Users should understand what happened during upgrade
for auditability and debugging purposes.

**Independent Test**: Run an upgrade and verify output messages clearly indicate
which directories were preserved vs replaced.

**Acceptance Scenarios**:

1. **Given** a SpecKit upgrade is in progress,
   **When** the upgrade completes,
   **Then** the script displays a message indicating memory files were preserved.

2. **Given** a SpecKit upgrade is in progress,
   **When** scripts or templates are replaced,
   **Then** the script displays a message indicating scripts/templates were updated.

---

### Edge Cases

- What happens when `.specify/memory/` does not exist before upgrade?
  (New memory directory created from templates)
- What happens when `.specify/scripts/` or `.specify/templates/` do not exist?
  (Created from scratch with new version)
- What happens when a memory file has the same name as a new template file?
  (Existing memory file preserved; new template not applied)
- What happens during the first SpecKit install (not an upgrade)?
  (All directories and files created from templates; no preservation logic)

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The script MUST preserve all files in `.specify/memory/` during
  SpecKit upgrades, leaving their content unchanged.

- **FR-002**: The script MUST replace all files in `.specify/scripts/` with
  the corresponding files from the new SpecKit version during upgrades.

- **FR-003**: The script MUST replace all files in `.specify/templates/` with
  the corresponding files from the new SpecKit version during upgrades.

- **FR-004**: The script MUST NOT prompt for consent before replacing scripts
  or templates during upgrade, as these are considered SpecKit-managed artifacts.

- **FR-005**: The script MUST discard any local modifications to scripts and
  templates without attempting to merge or preserve changes.

- **FR-006**: During fresh SpecKit installation (not upgrade), the script MUST
  create all directories and files from SpecKit templates, including memory files.

- **FR-007**: The script MUST display informational messages during upgrade
  indicating what file operations were performed.

### Key Entities

- **Memory Files**: Project-specific context files in `.specify/memory/` that
  accumulate over time and must be preserved across upgrades.

- **Scripts**: Executable shell scripts in `.specify/scripts/` that are
  SpecKit-managed and replaced on upgrade.

- **Templates**: Document templates in `.specify/templates/` that are
  SpecKit-managed and replaced on upgrade.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of files in `.specify/memory/` are byte-identical before
  and after any SpecKit upgrade operation.

- **SC-002**: 100% of files in `.specify/scripts/` match the new SpecKit
  version's scripts after upgrade.

- **SC-003**: 100% of files in `.specify/templates/` match the new SpecKit
  version's templates after upgrade.

- **SC-004**: Users can verify upgrade behavior by inspecting the upgrade
  output messages, which clearly indicate preserved vs replaced files.

## Assumptions

- The distinction between "upgrade" and "fresh install" is determined by whether
  `.venv/` exists with a previous SpecKit stamp file.

- SpecKit provides scripts and templates in a predictable structure that
  `specify-run` can copy during upgrade.

- The upgrade process is atomic: if interrupted, the user can delete `.venv/`
  and re-run to get a clean install.
