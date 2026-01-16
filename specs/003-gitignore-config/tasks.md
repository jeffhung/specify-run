# Tasks: Gitignore Configuration for SpecKit

**Input**: Design documents from `/specs/003-gitignore-config/`  
**Prerequisites**: plan.md, spec.md, research.md, data-model.md

**Tests**: Manual verification only (no automated test framework for bash scripts).

**Organization**: Tasks are grouped by user story. User Stories 1 & 2 are tightly
coupled (same gitignore block creation), so they're combined in Phase 3. User
Stories 3 & 4 are satisfied by the same implementation.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2)
- Include exact file paths in descriptions

## Path Conventions

- **Single file**: `specify-run` at repository root

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: No setup needed - modifying existing script

*No tasks - `specify-run` already exists*

**Checkpoint**: Ready to proceed directly to implementation

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Helper functions needed before gitignore logic

- [x] T001 Add `is_git_repo()` helper function in specify-run (after existing helpers around line 97)
- [x] T002 Add `configure_gitignore()` function skeleton in specify-run (after `is_agentic()`)
- [x] T003 Add `GITIGNORE_MARKER` constant in specify-run (in Configuration section around line 28)

**Checkpoint**: Foundation ready - gitignore implementation can begin

---

## Phase 3: User Stories 1 & 2 - Bootstrap Creates Gitignore with Negation Patterns (Priority: P1)

**Goal**: Create/append `.gitignore` with SpecKit patterns including negation patterns
to protect version-controlled directories from broader ignore rules.

**Independent Test**: Run `./specify-run` in fresh git repo, verify `.gitignore`
contains marker and all patterns. Verify `git check-ignore .specify/memory/test.md`
returns exit code 1 (not ignored).

### Implementation for User Stories 1 & 2

- [x] T004 [US1] Implement git repository detection using `git rev-parse --is-inside-work-tree` in `is_git_repo()` in specify-run
- [x] T005 [US1] Implement marker check logic (`grep -q "# specify-run"`) in `configure_gitignore()` in specify-run
- [x] T006 [US1] Implement gitignore block content with `.venv/` and negation patterns in `configure_gitignore()` in specify-run
- [x] T007 [US2] Add negation patterns `!.specify/`, `!.specify/scripts/**`, `!.specify/templates/**`, `!.specify/memory/**` in gitignore block in specify-run
- [x] T008 [US1] Implement append logic (create file if missing, append if marker absent) in `configure_gitignore()` in specify-run
- [x] T009 [US1] Call `configure_gitignore` after virtualenv creation in specify-run (around line 114)

**Checkpoint**: `.gitignore` created with negation patterns protecting version-controlled directories

---

## Phase 4: User Stories 3 & 4 - Ignore Runtime Artifacts (Priority: P2)

**Goal**: Ensure cache, tmp, logs, runtime directories are ignored while `specify-run`
script remains trackable.

**Independent Test**: Create test files in `.specify/cache/`, verify ignored. Verify
`specify-run` is not in `.gitignore`.

### Implementation for User Stories 3 & 4

- [x] T010 [US3] Add ignore patterns `.specify/cache/`, `.specify/tmp/`, `.specify/logs/`, `.specify/.runtime/` in gitignore block in specify-run
- [x] T011 [US4] Verify `specify-run` is NOT added to gitignore patterns (code review only - no code change)

**Checkpoint**: Runtime artifacts ignored, versioned files trackable

---

## Phase 5: Edge Cases & Error Handling

**Purpose**: Handle non-git repos, read-only files, conflicting patterns

- [ ] T012 Implement silent skip when `is_git_repo()` returns false in `configure_gitignore()` in specify-run
- [ ] T013 Add write error handling with warning message in `configure_gitignore()` in specify-run
- [ ] T014 Add info message when gitignore is configured: `info "Configured .gitignore"` in specify-run

**Checkpoint**: All edge cases handled gracefully

---

## Phase 6: Polish & Validation

**Purpose**: Final verification and documentation

- [ ] T015 [P] Manual verification: run `./specify-run` in fresh git repo, check `.gitignore` contents
- [ ] T016 [P] Manual verification: run `./specify-run` twice, verify idempotency (no duplicates)
- [ ] T017 [P] Manual verification: run `git check-ignore` on all paths per quickstart.md
- [ ] T018 Update CLAUDE.md Recent Changes section if needed

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: N/A - no setup tasks
- **Foundational (Phase 2)**: No dependencies - can start immediately
- **User Stories 1 & 2 (Phase 3)**: Depends on Phase 2 completion
- **User Stories 3 & 4 (Phase 4)**: Can be done in parallel with Phase 3 (same function)
- **Edge Cases (Phase 5)**: Depends on Phases 3 & 4
- **Polish (Phase 6)**: Depends on all implementation complete

### User Story Dependencies

- **User Stories 1 & 2 (P1)**: Combined - same gitignore block creation
- **User Stories 3 & 4 (P2)**: Combined - same pattern additions, no dependency on US1/2

### Within Each Phase

- T001-T003: Can run in parallel (different sections of file)
- T004-T009: Sequential (building on previous tasks)
- T010-T011: Can run in parallel with T004-T009
- T015-T018: All parallel (independent verification)

---

## Parallel Example: Foundational Phase

```bash
# These can be worked on in parallel (different sections):
Task: "Add is_git_repo() helper function in specify-run"
Task: "Add GITIGNORE_MARKER constant in specify-run"
```

---

## Implementation Strategy

### MVP First (User Stories 1 & 2)

1. Complete Phase 2: Foundational (helper functions)
2. Complete Phase 3: User Stories 1 & 2 (core gitignore creation)
3. **STOP and VALIDATE**: Test in fresh git repo
4. Proceed if working

### Full Implementation

1. Complete Foundational → Helpers ready
2. Complete User Stories 1 & 2 → Core gitignore working
3. Complete User Stories 3 & 4 → All patterns in place
4. Complete Edge Cases → Robust error handling
5. Complete Polish → Verified and documented

---

## Notes

- All implementation is in single file: `specify-run`
- No automated tests - manual verification via `git check-ignore`
- Marker `# specify-run` ensures idempotency
- Negation patterns must come before ignore patterns in the block
