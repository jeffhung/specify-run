# Tasks: Script Versioning

**Input**: Design documents from `/specs/006-script-versioning/`
**Prerequisites**: plan.md, spec.md, research.md, quickstart.md

**Tests**: Included per plan.md (manual verification + shell script tests)

**Organization**: Tasks grouped by user story for independent implementation.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2)
- Include exact file paths in descriptions

## Path Conventions

- Single file script at repository root: `specify-run`
- Tests directory: `tests/`

---

## Phase 1: Setup

**Purpose**: No setup needed - modifying existing script

This feature modifies an existing single-file script. No project initialization
required.

**Checkpoint**: Ready for implementation

---

## Phase 2: User Story 1 - View Combined Version Information (Priority: P1) MVP

**Goal**: Display both specify-run and SpecKit versions when running
`./specify-run version` with NO side effects (no bootstrap, no hardenings)

**Independent Test**: Run `./specify-run version` and verify:
- Wrapper version displayed
- SpecKit version displayed (if installed)
- No .venv created in fresh environment
- No hardenings applied

### Implementation for User Story 1

- [x] T001 [US1] Add SPECIFYRUN_VERSION="0.6.0" to Configuration section in specify-run
- [x] T002 [US1] Add early-exit version logic after REQUIRED_GITIGNORE_PATTERNS in specify-run
- [x] T003 [US1] Verify version output shows "specify-run 0.6.0" followed by SpecKit version

### Tests for User Story 1

- [ ] T004 [P] [US1] Create version output test in tests/test_version.sh (skipped - manual verification sufficient)
- [x] T005 [US1] Verify no side effects: version command does not create .venv
- [x] T006 [US1] Verify other commands pass through unchanged (e.g., ./specify-run init --help)

**Checkpoint**: User Story 1 complete - version command displays versions with
no side effects

---

## Phase 3: User Story 2 - Version in Wrapper Script (Priority: P2)

**Goal**: Version variable is in configuration section for easy maintenance

**Independent Test**: Read script source and confirm SPECIFYRUN_VERSION is in
Configuration section alongside SPECKIT_GIT and SPECKIT_REF

### Implementation for User Story 2

- [x] T007 [US2] Verify version variable placement is correct in specify-run (already done in T001)
- [x] T008 [US2] Add comment "# Wrapper version (Semantic Versioning)" above variable in specify-run

**Checkpoint**: User Story 2 complete - version variable clearly labeled in
configuration section

---

## Phase 4: Polish & Validation

**Purpose**: Final validation and documentation

- [x] T009 Run quickstart.md Test 1: Version command (SpecKit installed)
- [x] T010 Run quickstart.md Test 2: Version command (fresh environment)
- [x] T011 Run quickstart.md Test 3: Other commands unchanged
- [x] T012 Run quickstart.md Test 4: Version format validation (semver pattern)
- [x] T013 Run quickstart.md Test 5: No side effects verification
- [x] T014 Update README.md to document `./specify-run version` command

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: N/A - no setup needed
- **Phase 2 (US1)**: Core implementation - can start immediately
- **Phase 3 (US2)**: Refinement - verifies T001 placement
- **Phase 4 (Polish)**: Depends on Phase 2 and 3 completion

### User Story Dependencies

- **User Story 1 (P1)**: No dependencies - core feature
- **User Story 2 (P2)**: Depends on US1 completion (verifies variable placement)

### Parallel Opportunities

- T004 (test file) can be created in parallel with T001-T002
- All tasks in Phase 4 (T009-T013) can run in parallel

---

## Parallel Example: User Story 1

```bash
# Implementation and test creation can run in parallel:
Task: "Add SPECIFYRUN_VERSION to Configuration section in specify-run"
Task: "Create version output test in tests/test_version.sh"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete T001: Add version variable
2. Complete T002: Add early-exit version logic (CRITICAL: before bootstrap)
3. Complete T003: Verify output
4. Complete T005: Verify no side effects
5. **STOP and VALIDATE**: Test `./specify-run version` independently
6. Deploy/demo if ready

### Full Delivery

1. Complete Phase 2 (User Story 1) -> MVP ready
2. Complete Phase 3 (User Story 2) -> Maintainability verified
3. Complete Phase 4 (Polish) -> All validations pass, docs updated

---

## Notes

- Single file modification (~15 lines added to specify-run)
- No external dependencies
- Version format: MAJOR.MINOR.PATCH (0.6.0)
- **CRITICAL**: Version logic must be EARLY EXIT - before any bootstrap/hardening
- Commit after T002 for minimal viable change
