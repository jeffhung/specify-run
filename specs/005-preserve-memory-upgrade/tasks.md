# Tasks: Preserve Memory on Upgrade

**Input**: Design documents from `/specs/005-preserve-memory-upgrade/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, quickstart.md

**Tests**: Manual integration tests only (per plan.md). No automated test tasks.

**Organization**: Tasks are grouped by user story to enable independent
implementation and testing.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2)
- Single-file project: all changes go directly into `specify-run`

## Path Conventions

- **Source**: `specify-run` (single bash script at repository root)
- **Spec docs**: `specs/005-preserve-memory-upgrade/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Add helper functions and constants needed by all user stories

- [x] T001 Add `detect_provisioned_agents()` function in `specify-run`
- [x] T002 Add `backup_memory()` function in `specify-run`
- [x] T003 [P] Add `restore_memory()` function in `specify-run`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Modify upgrade detection to prepare for re-bootstrap flow

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T004 Refactor upgrade block to separate CLI upgrade from project file handling in `specify-run`

**Checkpoint**: Foundation ready - user story implementation can now begin

---

## Phase 3: User Story 1 - Memory Files Preserved After Upgrade (Priority: P1) 🎯 MVP

**Goal**: Preserve all files in `.specify/memory/` during SpecKit upgrades

**Independent Test**: Modify `constitution.md`, upgrade SpecKit, verify content unchanged

### Implementation for User Story 1

- [ ] T005 [US1] Call `detect_provisioned_agents()` after CLI upgrade in `specify-run`
- [ ] T006 [US1] Call `backup_memory()` before first `specify init` in `specify-run`
- [ ] T007 [US1] Loop through provisioned agents calling `specify init --here --force --ai $agent` in `specify-run`
- [ ] T008 [US1] Call `restore_memory()` after all agents re-bootstrapped in `specify-run`
- [ ] T009 [US1] Handle edge case: no provisioned agents found (skip re-bootstrap) in `specify-run`
- [ ] T010 [US1] Handle edge case: no memory directory exists (skip backup) in `specify-run`
- [ ] T011 [US1] Restore memory on `specify init` failure in `specify-run`

**Checkpoint**: Memory files preserved during upgrade. Ready for manual testing.

---

## Phase 4: User Story 2 - Scripts and Templates Replaced on Upgrade (Priority: P1)

**Goal**: Replace `.specify/scripts/` and `.specify/templates/` with new versions

**Independent Test**: Modify a script file, upgrade SpecKit, verify overwritten

### Implementation for User Story 2

- [ ] T012 [US2] Verify `specify init --here --force` replaces scripts and templates (no code change needed, behavior from SpecKit)
- [ ] T013 [US2] Manual test: modify `.specify/scripts/bash/common.sh`, upgrade, verify replaced

**Checkpoint**: Scripts and templates replaced on upgrade. This is inherent behavior from `specify init --force`.

---

## Phase 5: User Story 3 - Clear Upgrade Messages (Priority: P2)

**Goal**: Display clear messages about preserved vs replaced files

**Independent Test**: Run upgrade and verify output messages

### Implementation for User Story 3

- [ ] T014 [US3] Add info message "Backed up .specify/memory/" in `backup_memory()` in `specify-run`
- [ ] T015 [US3] Add info message "Re-bootstrapping for $agent..." before each `specify init` in `specify-run`
- [ ] T016 [US3] Add info message "Preserved .specify/memory/ (project-specific files)" after restore in `specify-run`
- [ ] T017 [US3] Add info message "Replaced .specify/scripts/ and .specify/templates/" after all inits in `specify-run`
- [ ] T018 [US3] Add message for no-agents case "No SpecKit project files found - skipping re-bootstrap" in `specify-run`

**Checkpoint**: All upgrade messages display correctly

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Edge cases, validation, documentation

- [ ] T019 Run shellcheck on `specify-run`
- [ ] T020 Manual test: full upgrade flow with single agent (claude)
- [ ] T021 Manual test: upgrade flow with multiple agents (claude + copilot)
- [ ] T022 Manual test: upgrade with no provisioned agents
- [ ] T023 Manual test: upgrade with no existing memory directory
- [ ] T024 [P] Update README.md with upgrade behavior documentation
- [ ] T025 Run quickstart.md validation scenarios

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion
- **User Story 1 (Phase 3)**: Depends on Foundational
- **User Story 2 (Phase 4)**: Depends on User Story 1 (uses same upgrade flow)
- **User Story 3 (Phase 5)**: Depends on User Story 1 (adds messages to existing flow)
- **Polish (Phase 6)**: Depends on all user stories

### User Story Dependencies

- **User Story 1 (P1)**: Core functionality - must complete first
- **User Story 2 (P1)**: Verification only - no code changes beyond US1
- **User Story 3 (P2)**: Adds messages to US1 implementation

### Within Each User Story

- Helper functions before main flow integration
- Core path before edge cases
- Implementation before manual testing

### Parallel Opportunities

- T002 and T003 can run in parallel (independent helper functions)
- T014-T018 are all message additions that modify different code paths

---

## Parallel Example: Phase 1 Setup

```bash
# After T001 completes, launch T002 and T003 together:
Task: "Add backup_memory() function in specify-run"
Task: "Add restore_memory() function in specify-run"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T003)
2. Complete Phase 2: Foundational (T004)
3. Complete Phase 3: User Story 1 (T005-T011)
4. **STOP and VALIDATE**: Test memory preservation manually
5. Ready for demo/review

### Incremental Delivery

1. Complete Setup + Foundational → Upgrade refactored
2. Add User Story 1 → Memory preserved → Test manually
3. Add User Story 2 → Verify scripts/templates replaced (inherent)
4. Add User Story 3 → Clear messages added
5. Polish → Full validation

---

## Notes

- Single-file project: all code changes go to `specify-run`
- [P] tasks = independent helper functions
- [Story] label maps task to specific user story
- Manual testing per quickstart.md scenarios
- User Story 2 requires minimal code - behavior comes from `specify init --force`
- Commit after each logical group of tasks
