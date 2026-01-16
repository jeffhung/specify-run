# Tasks: User Consent Prompting

**Input**: Design documents from `/specs/002-prompting/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, quickstart.md

**Tests**: No tests explicitly requested. Manual testing via quickstart.md.

**Organization**: Tasks grouped by user story for independent implementation.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- All changes in single file: `specify-run`

## Path Conventions

- **Single file**: All modifications to `specify-run` at repository root
- No new files created in source tree

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Add helper functions and constants used by all prompting logic

- [x] T001 Add exit code constants (EX_TEMPFAIL=75, EX_CONFIG=78) in specify-run
- [x] T002 Add `prompt_user()` function skeleton in specify-run
- [x] T003 Add `parse_answers()` function to parse SPECIFYRUN_ANSWERS in specify-run
- [x] T004 Add `is_interactive()` function using `[ -t 0 ]` check in specify-run

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core prompting infrastructure that all user stories depend on

**CRITICAL**: No user story work can begin until this phase is complete

- [x] T005 Implement `prompt_user()` to display message and read y/n in specify-run
- [x] T006 Implement `parse_answers()` to extract key=value pairs in specify-run
- [x] T007 ~~Add `--yes`/`-y` flag detection~~ N/A - using env vars only
- [x] T008 Add `SPECIFYRUN_BY_AGENT` environment variable detection in specify-run

**Checkpoint**: Foundation ready - prompting infrastructure complete

---

## Phase 3: User Story 1 - First-Run Bootstrap Consent (Priority: P1) MVP

**Goal**: Prompt before creating `.venv/` on first run

**Independent Test**: Run `./specify-run` in directory without `.venv/`; verify
prompt appears before any directory creation.

### Implementation for User Story 1

- [ ] T009 [US1] Add bootstrap prompt before `python -m venv` call in specify-run
- [ ] T010 [US1] Implement interactive mode: read y/n, proceed or exit 130 in specify-run
- [ ] T011 [US1] Implement SPECIFYRUN_ANSWERS bypass for bootstrap prompt in specify-run
- [ ] T012 [US1] Implement agentic mode with `bootstrap` key hint in specify-run
- [ ] T013 [US1] Exit 75 when agentic mode has no matching answer in specify-run
- [ ] T014 [US1] Exit 78 when non-interactive without `--yes` in specify-run

**Checkpoint**: First-run prompting fully functional

---

## Phase 4: User Story 2 - SpecKit Upgrade Consent (Priority: P2)

**Goal**: Prompt before upgrading SpecKit when version changes

**Independent Test**: Change `SPECKIT_REF`, run `./specify-run`; verify upgrade
prompt appears before pip install.

### Implementation for User Story 2

- [ ] T015 [US2] Add upgrade prompt before `pip install` when stamp mismatches in specify-run
- [ ] T016 [US2] Display old and new version in upgrade prompt in specify-run
- [ ] T017 [US2] Implement SPECIFYRUN_ANSWERS bypass for upgrade prompt in specify-run
- [ ] T018 [US2] Implement agentic mode with `upgrade` key hint in specify-run
- [ ] T019 [US2] Handle exit codes (130/75/78) for upgrade prompt in specify-run

**Checkpoint**: Upgrade prompting fully functional

---

## Phase 5: User Story 3 - AI Agent Invocation (Priority: P2)

**Goal**: Enable agentic mode via environment variables

**Independent Test**: Run with `SPECIFYRUN_BY_AGENT=1`; verify hint printed and
exit 75. Retry with `SPECIFYRUN_ANSWERS="bootstrap=y"`; verify proceeds.

### Implementation for User Story 3

- [ ] T020 [US3] Print answer key hint format in agentic mode in specify-run
- [ ] T021 [US3] Look up answer from SPECIFYRUN_ANSWERS before prompting in specify-run
- [ ] T022 [US3] Proceed automatically if matching answer found in specify-run
- [ ] T023 [US3] Ignore unrecognized keys silently in specify-run

**Checkpoint**: Agentic mode fully functional

---

## Phase 6: User Story 4 - Delegated Execution (Priority: P3)

**Goal**: Pass through to SpecKit without prompts when no modifications needed

**Independent Test**: Run `./specify-run init` after setup; verify no prompts
from specify-run (only SpecKit's prompts if any).

### Implementation for User Story 4

- [ ] T024 [US4] Skip all prompts when `.venv/` exists and stamp matches in specify-run
- [ ] T025 [US4] Ensure `exec "$SPECIFY" "$@"` happens without modification in specify-run

**Checkpoint**: Delegated execution works without interference

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Documentation and final validation

- [ ] T026 Update CLAUDE.md with SPECIFYRUN_BY_AGENT instructions
- [ ] T027 Update README.md with prompting behavior documentation
- [ ] T028 Run quickstart.md validation scenarios manually
- [ ] T029 Verify all exit codes match specification (0, 75, 78, 130)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-6)**: All depend on Foundational phase completion
  - US1 and US2 can run in parallel (different prompt triggers)
  - US3 depends on US1/US2 (uses same prompting infrastructure)
  - US4 is independent (no-op path)
- **Polish (Phase 7)**: Depends on all user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Phase 2 - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Phase 2 - Independent of US1
- **User Story 3 (P2)**: Builds on US1/US2 prompting, should follow them
- **User Story 4 (P3)**: Independent, can run after Phase 2

### Within Each User Story

- Core prompt logic first
- Mode-specific handling (interactive, --yes, agentic) after
- Exit code handling last

### Parallel Opportunities

- T001-T004 (Setup) can run in parallel
- T005-T008 (Foundational) are sequential (same file, related logic)
- US1 and US2 implementation can be interleaved
- T026-T027 (documentation) can run in parallel

---

## Parallel Example: Setup Phase

```bash
# These Setup tasks modify different functions:
Task: "Add exit code constants in specify-run"
Task: "Add prompt_user() function skeleton in specify-run"
Task: "Add parse_answers() function in specify-run"
Task: "Add is_interactive() function in specify-run"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T004)
2. Complete Phase 2: Foundational (T005-T008)
3. Complete Phase 3: User Story 1 (T009-T014)
4. **STOP and VALIDATE**: Test first-run prompting independently
5. Demo: `./specify-run` in clean directory shows prompt

### Incremental Delivery

1. Setup + Foundational → Prompting infrastructure ready
2. Add US1 → First-run bootstrap prompting works (MVP!)
3. Add US2 → Upgrade prompting works
4. Add US3 → Agentic mode works
5. Add US4 → Verify no-op path unaffected
6. Polish → Documentation complete

---

## Notes

- All tasks modify single file `specify-run` - coordinate edits carefully
- Test each checkpoint before proceeding
- Commit after each phase completion
- Exit codes are critical for agent integration - verify with `echo $?`
