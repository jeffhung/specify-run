# Feature Specification: Gitignore Configuration for SpecKit

**Feature Branch**: `003-gitignore-config`  
**Created**: 2026-01-16  
**Status**: Draft  
**Input**: User description: "Configure .gitignore to ensure SpecKit memory, template,
and script files are NOT ignored. And to ensure SpecKit cache, tmp, logs, and
runtime are ignored. The virtualenv folder should be ignored. And SpecKit version
should be versioned."

## Clarifications

### Session 2026-01-16

- Q: How should version-controlled files be protected from broader gitignore patterns? → A: Use negation patterns (`!path`) to explicitly un-ignore SpecKit directories.

**Reference gitignore entries:**
```
!.specify/
!.specify/scripts/**
!.specify/templates/**
!.specify/memory/**

.specify/cache/
.specify/tmp/
.specify/logs/
.specify/.runtime/
```

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Bootstrap Creates Proper Gitignore (Priority: P1)

When a user runs `./specify-run` for the first time in a git-controlled repository,
the system should automatically configure `.gitignore` to ensure proper version
control of SpecKit files.

**Why this priority**: This is the core functionality. Without proper gitignore
configuration during bootstrap, users may accidentally commit runtime artifacts or
lose important specification files.

**Independent Test**: Can be fully tested by running `./specify-run` in a fresh git
repository and verifying `.gitignore` contents.

**Acceptance Scenarios**:

1. **Given** a git repository without `.gitignore`, **When** user runs `./specify-run`
   for the first time, **Then** a `.gitignore` file is created with SpecKit-specific
   entries.
2. **Given** a git repository with existing `.gitignore`, **When** user runs
   `./specify-run` for the first time, **Then** SpecKit entries are appended without
   duplicating existing entries.
3. **Given** a git repository, **When** bootstrap completes, **Then** `.venv/`
   directory is listed in `.gitignore`.

---

### User Story 2 - Version-Controlled Files Remain Trackable (Priority: P1)

Users must be able to version control SpecKit memory, templates, and scripts to
ensure team collaboration and history tracking.

**Why this priority**: Equal to P1 because losing version control of specifications
defeats the purpose of the tool.

**Independent Test**: After bootstrap, run `git status` and verify that files in
`.specify/memory/`, `.specify/templates/`, and `.specify/scripts/` are not ignored.

**Acceptance Scenarios**:

1. **Given** a bootstrapped repository with broad ignore patterns, **When** user
   creates a file in `.specify/memory/`, **Then** `git status` shows the file as
   untracked because negation pattern `!.specify/memory/**` overrides.
2. **Given** a bootstrapped repository, **When** user creates a file in
   `.specify/templates/`, **Then** the file is trackable due to negation pattern.
3. **Given** a bootstrapped repository, **When** user creates a file in
   `.specify/scripts/`, **Then** the file is trackable due to negation pattern.

---

### User Story 3 - Runtime Artifacts Are Ignored (Priority: P2)

Runtime artifacts such as cache, temporary files, logs, and runtime environments
should be automatically ignored to keep the repository clean.

**Why this priority**: Important for repository hygiene but secondary to core
version control functionality.

**Independent Test**: Create files in cache/tmp/logs directories and verify they
appear as ignored in `git status --ignored`.

**Acceptance Scenarios**:

1. **Given** a bootstrapped repository, **When** SpecKit creates cache files,
   **Then** those files are ignored by git.
2. **Given** a bootstrapped repository, **When** log files are generated, **Then**
   those files are ignored by git.
3. **Given** a bootstrapped repository, **When** temporary files are created,
   **Then** those files are ignored by git.

---

### User Story 4 - SpecKit Version File Is Versioned (Priority: P2)

The SpecKit version configuration (embedded in `specify-run` script) must be
tracked in version control so team members use the same version.

**Why this priority**: Important for team consistency but the script is already a
tracked file by default.

**Independent Test**: Verify that `specify-run` and any version stamp files are not
in `.gitignore`.

**Acceptance Scenarios**:

1. **Given** a bootstrapped repository, **When** user modifies `SPECKIT_REF` in
   `specify-run`, **Then** `git diff` shows the change.
2. **Given** a repository with `.gitignore` configured, **When** user checks git
   status, **Then** `specify-run` is not ignored.

---

### Edge Cases

- What happens when `.gitignore` already contains conflicting patterns (e.g.,
  `*.md` that would ignore specs)?
  - System should warn but not automatically modify user's existing patterns.
- How does system handle a non-git repository?
  - System should skip gitignore configuration silently (not an error).
- What happens if `.gitignore` is read-only?
  - System should warn the user and continue bootstrap without modifying gitignore.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST detect if current directory is under git version control
  before attempting gitignore configuration.
- **FR-002**: System MUST create `.gitignore` if it does not exist in a git
  repository.
- **FR-003**: System MUST append SpecKit entries to existing `.gitignore` without
  duplicating entries already present.
- **FR-004**: System MUST add `.venv/` to gitignore to exclude the virtualenv
  directory.
- **FR-005**: System MUST add patterns to ignore SpecKit cache directories
  (`.specify/cache/`).
- **FR-006**: System MUST add patterns to ignore SpecKit temporary directories
  (`.specify/tmp/`).
- **FR-007**: System MUST add patterns to ignore SpecKit log files
  (`.specify/logs/`).
- **FR-008**: System MUST add pattern `.specify/.runtime/` to ignore SpecKit
  runtime directory.
- **FR-009**: System MUST add negation pattern `!.specify/` to un-ignore the
  SpecKit configuration directory.
- **FR-010**: System MUST add negation pattern `!.specify/scripts/**` to un-ignore
  the scripts directory and its contents.
- **FR-011**: System MUST add negation pattern `!.specify/templates/**` to un-ignore
  the templates directory and its contents.
- **FR-012**: System MUST add negation pattern `!.specify/memory/**` to un-ignore
  the memory directory and its contents.
- **FR-013**: System MUST use a marker comment (e.g., `# SpecKit`) to group
  SpecKit-related gitignore entries.
- **FR-014**: System MUST be idempotent - running bootstrap multiple times should
  not duplicate gitignore entries.
- **FR-015**: System SHOULD warn user if existing gitignore patterns may conflict
  with SpecKit files.

### Key Entities

- **Gitignore File**: The `.gitignore` file at repository root that controls which
  files git ignores.
- **SpecKit Configuration Directory**: The `.specify/` directory containing
  SpecKit's working files.
- **Version-Controlled Subdirectories**: `memory/`, `templates/`, `scripts/` under
  `.specify/` that must be tracked.
- **Ignored Subdirectories**: `cache/`, `tmp/`, `logs/`, `runtime/` under
  `.specify/` that should not be tracked.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: After bootstrap in a git repository, all files in `.specify/memory/`,
  `.specify/templates/`, and `.specify/scripts/` are trackable by git (verified by
  `git check-ignore` returning non-zero).
- **SC-002**: After bootstrap, files created in `.specify/cache/`, `.specify/tmp/`,
  `.specify/logs/`, and `.specify/runtime/` are ignored by git (verified by
  `git check-ignore` returning zero).
- **SC-003**: The `.venv/` directory is ignored after bootstrap.
- **SC-004**: Running bootstrap twice produces identical `.gitignore` content (no
  duplicates).
- **SC-005**: Bootstrap completes successfully in non-git directories without
  errors related to gitignore.
- **SC-006**: Existing `.gitignore` entries are preserved when appending SpecKit
  patterns.

## Assumptions

- The bootstrap process has write access to the repository root directory.
- Users expect SpecKit to manage its own gitignore entries automatically.
- The `.specify/` directory structure follows a standard layout with known
  subdirectories.
- POSIX-compatible shell environment is available for gitignore manipulation.
