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
`./specify-run version`

**Independent Test**: Run `./specify-run version` and verify both version
strings appear in output

### Implementation for User Story 1

- [ ] T001 [US1] Add SPECIFYRUN_VERSION="0.6.0" to Configuration section in specify-run
- [ ] T002 [US1] Add version display logic before exec statement in specify-run
- [ ] T003 [US1] Verify version output shows "specify-run 0.6.0" followed by SpecKit version

### Tests for User Story 1

- [ ] T004 [P] [US1] Create version output test in tests/test_version.sh
- [ ] T005 [US1] Verify other commands pass through unchanged (e.g., ./specify-run init --help)

**Checkpoint**: User Story 1 complete - version command displays both versions

---

## Phase 3: User Story 2 - Version in Wrapper Script (Priority: P2)

**Goal**: Version variable is in configuration section for easy maintenance

**Independent Test**: Read script source and confirm SPECIFYRUN_VERSION is in
Configuration section alongside SPECKIT_GIT and SPECKIT_REF

### Implementation for User Story 2

- [ ] T006 [US2] Verify version variable placement is correct in specify-run (already done in T001)
- [ ] T007 [US2] Add comment "# Wrapper version (Semantic Versioning)" above variable in specify-run

**Checkpoint**: User Story 2 complete - version variable clearly labeled in
configuration section

---

## Phase 4: Polish & Validation

**Purpose**: Final validation and documentation

- [ ] T008 Run quickstart.md Test 1: Version command verification
- [ ] T009 Run quickstart.md Test 2: Other commands unchanged
- [ ] T010 Run quickstart.md Test 3: Version format validation (semver pattern)
- [ ] T011 Update README.md to document `./specify-run version` command

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
- All tasks in Phase 4 (T008-T010) can run in parallel

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
2. Complete T002: Add version display logic
3. Complete T003: Verify output
4. **STOP and VALIDATE**: Test `./specify-run version` independently
5. Deploy/demo if ready

### Full Delivery

1. Complete Phase 2 (User Story 1) -> MVP ready
2. Complete Phase 3 (User Story 2) -> Maintainability verified
3. Complete Phase 4 (Polish) -> All validations pass, docs updated

---

## Notes

- Single file modification (~10 lines added to specify-run)
- No external dependencies
- Version format: MAJOR.MINOR.PATCH (0.6.0)
- Commit after T002 for minimal viable change
