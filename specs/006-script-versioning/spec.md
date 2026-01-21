# Feature Specification: Script Versioning

**Feature Branch**: `006-script-versioning`  
**Created**: 2026-01-21  
**Status**: Draft  
**Input**: User description: "The specify-run script itself should have versioning.
Display version information together with specify version when invoked with
`./specify-run version`. Start version from 0.6.0 following Semantic Versioning."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - View Combined Version Information (Priority: P1)

As a user running `./specify-run version`, I want to see both the specify-run
wrapper version and the underlying SpecKit version so I can quickly identify
what versions I'm working with for troubleshooting or compatibility purposes.

**Why this priority**: This is the core value proposition - users need to know
both versions when debugging issues, reporting bugs, or verifying their
environment is up to date.

**Independent Test**: Can be fully tested by running `./specify-run version`
and verifying both version strings appear in the output.

**Acceptance Scenarios**:

1. **Given** a properly configured environment with SpecKit installed, **When**
   the user runs `./specify-run version`, **Then** the output displays the
   specify-run version (e.g., "0.6.0") followed by or alongside the SpecKit
   version information.

2. **Given** a properly configured environment, **When** the user runs
   `./specify-run version`, **Then** the specify-run version follows Semantic
   Versioning format (MAJOR.MINOR.PATCH).

---

### User Story 2 - Version in Wrapper Script (Priority: P2)

As a maintainer, I want the version number to be declared within the specify-run
script itself so that version changes can be tracked through git commits and the
version is authoritative without external dependencies.

**Why this priority**: Maintainability requires a single source of truth for the
wrapper version that can be easily updated and tracked.

**Independent Test**: Can be verified by reading the script source and
confirming a version variable is declared near the top of the configuration
section.

**Acceptance Scenarios**:

1. **Given** the specify-run script, **When** a maintainer opens the file,
   **Then** they can find a clearly labeled version declaration in the
   configuration section.

---

### Edge Cases

- What happens when SpecKit is not yet installed (fresh environment)?
  - When running `./specify-run version` before bootstrap, the wrapper should
    proceed with bootstrap first (existing behavior), then show versions.
- What happens when the specify command fails or produces errors?
  - The wrapper version display should not interfere with normal error handling.
- What happens when other arguments are passed to specify-run?
  - Only `./specify-run version` triggers combined version display; other
    commands pass through unchanged.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The script MUST declare a version variable following Semantic
  Versioning format (MAJOR.MINOR.PATCH).
- **FR-002**: The initial version MUST be set to 0.6.0.
- **FR-003**: When invoked with `version` as the first argument, the script MUST
  display the specify-run version before passing through to SpecKit's version
  output.
- **FR-004**: The version display MUST clearly distinguish between the wrapper
  version and the SpecKit version.
- **FR-005**: When other arguments are provided, the script MUST pass them
  through to SpecKit without additional version output (current behavior
  preserved).
- **FR-006**: The version variable MUST be located in the configuration section
  at the top of the script for easy maintenance.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Running `./specify-run version` displays both wrapper and SpecKit
  versions in under 1 second (excluding bootstrap time).
- **SC-002**: 100% of existing specify-run functionality remains unchanged when
  other arguments are passed.
- **SC-003**: The version format passes Semantic Versioning validation
  (MAJOR.MINOR.PATCH pattern).
- **SC-004**: Maintainers can update the version in a single location within the
  script.

## Assumptions

- The `specify version` command displays SpecKit's version information.
- The wrapper version and SpecKit version are independent and do not need to
  match.
- No additional command-line options (like `--version`) are needed since
  `./specify-run version` serves this purpose.
