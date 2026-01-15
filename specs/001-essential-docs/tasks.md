# Tasks: Essential Documentation

**Input**: Design documents from `/specs/001-essential-docs/`
**Prerequisites**: plan.md, spec.md, research.md

**Tests**: No automated tests (documentation feature - manual validation only)

**Organization**: Tasks are grouped by user story to enable independent
implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Deliverables**: README.md, SECURITY.md, INTEGRATION.md at repository root
- **Validation guide**: specs/001-essential-docs/quickstart.md

---

## Phase 1: Setup

**Purpose**: Review existing documentation and prepare for 3-file structure

- [x] T001 Review existing README.md structure and identify content to migrate
- [x] T002 Review research.md content migration plan

**Checkpoint**: Ready to begin content refactoring

---

## Phase 2: User Story 1 - First-Time Adopter Onboarding (Priority: P1)

**Goal**: Ensure developers understand what specify-run is, why they need it,
and how to install it within 10 minutes using README.md only.

**Independent Test**: Give README.md to unfamiliar developer, measure time to
first successful SpecKit run.

### Implementation for User Story 1

- [x] T003 [US1] Verify "What is specify-run" paragraph is clear in README.md
- [x] T004 [US1] Verify "How it works" section explains bootstrap flow in README.md
- [x] T005 [US1] Verify "Why this exists" section covers global install problems in README.md
- [x] T006 [US1] Add "Why a virtualenv?" section explaining isolation benefits in README.md
- [x] T007 [US1] Add "Why commit this script?" section with rationale in README.md
- [x] T008 [US1] Add links to SECURITY.md and INTEGRATION.md in README.md
- [x] T009 [US1] Remove security content from README.md (will move to SECURITY.md)
- [x] T010 [US1] Remove CI/agent content from README.md (will move to INTEGRATION.md)

**Checkpoint**: First-time adopter can understand and install in <10 minutes

---

## Phase 3: User Story 2 - Security-Conscious Reviewer (Priority: P2)

**Goal**: Security reviewers can identify hardening properties and understand
threat mitigations from SECURITY.md.

**Independent Test**: Security reviewer lists 3+ security properties after reading.

### Implementation for User Story 2

- [x] T011 [P] [US2] Create SECURITY.md with header and introduction
- [x] T012 [US2] Add threat model section to SECURITY.md
- [x] T013 [US2] Add hardening properties section to SECURITY.md
- [ ] T014 [US2] Add supply-chain protection section to SECURITY.md
- [ ] T015 [US2] Add concrete attack examples prevented to SECURITY.md
- [ ] T016 [US2] Add version pinning security benefits to SECURITY.md

**Checkpoint**: Security reviewer can identify 3+ hardening properties

---

## Phase 4: User Story 3 - CI/CD Integration (Priority: P2)

**Goal**: DevOps engineers can copy-paste GitHub Actions example from
INTEGRATION.md that works immediately.

**Independent Test**: Copy example to workflow, verify pipeline succeeds.

### Implementation for User Story 3

- [ ] T017 [P] [US3] Create INTEGRATION.md with header and introduction
- [ ] T018 [US3] Add GitHub Actions section with complete workflow example to INTEGRATION.md
- [ ] T019 [US3] Add other CI systems section (generic guidance) to INTEGRATION.md
- [ ] T020 [US3] Add CI troubleshooting section to INTEGRATION.md

**Checkpoint**: GitHub Actions example works when copy-pasted

---

## Phase 5: User Story 4 - Claude Code Integration (Priority: P2)

**Goal**: Teams using Claude Code can configure their agent correctly with
copy-paste CLAUDE.md snippet from INTEGRATION.md.

**Independent Test**: Follow instructions, verify agent uses ./specify-run
exclusively.

### Implementation for User Story 4

- [ ] T021 [US4] Add Claude Code integration section to INTEGRATION.md
- [ ] T022 [US4] Add complete CLAUDE.md snippet for adopters to INTEGRATION.md
- [ ] T023 [US4] Add anti-patterns section (what agents must NOT do) to INTEGRATION.md

**Checkpoint**: Claude Code integration works on first configuration attempt

---

## Phase 6: User Story 5 - Version Upgrade Workflow (Priority: P3)

**Goal**: Maintainers can safely upgrade SpecKit and the script itself with
clear procedures in README.md.

**Independent Test**: Follow upgrade instructions, verify new version active,
rollback in <2 minutes.

### Implementation for User Story 5

- [ ] T024 [US5] Verify SpecKit upgrade procedure is clear in README.md
- [ ] T025 [US5] Add "Upgrading the specify-run script" section in README.md
- [ ] T026 [US5] Add rollback procedure documentation in README.md

**Checkpoint**: Upgrade and rollback procedures work as documented

---

## Phase 7: Polish & Validation

**Purpose**: Final review and validation against acceptance criteria

- [ ] T027 Review README.md for consistency and flow
- [ ] T028 Review SECURITY.md for completeness
- [ ] T029 Review INTEGRATION.md for completeness
- [ ] T030 Verify all 12 required topics are covered across 3 files
- [ ] T031 Run quickstart.md validation checklist
- [ ] T032 Fix typo in README.md title ("sefety" → "safety")

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **User Story 1 (Phase 2)**: Depends on Setup; refactors README.md
- **User Stories 2-4 (Phases 3-5)**: Can start in parallel after US1 extracts content
  - US2 creates SECURITY.md (new file)
  - US3-4 create INTEGRATION.md (new file)
- **User Story 5 (Phase 6)**: Independent, modifies README.md
- **Polish (Phase 7)**: Depends on all user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Foundation - must complete first (extracts content)
- **User Story 2 (P2)**: Creates SECURITY.md - can start after T009 removes security from README
- **User Story 3 (P2)**: Creates INTEGRATION.md - can start after T010 removes CI from README
- **User Story 4 (P2)**: Extends INTEGRATION.md - depends on T017 creating the file
- **User Story 5 (P3)**: Independent - modifies README.md upgrade sections

### Parallel Opportunities

After US1 completes content extraction:
- US2 (SECURITY.md) and US3 (INTEGRATION.md) can run in parallel (different files)
- US4 depends on US3 creating INTEGRATION.md
- US5 can run in parallel with US2-4 (different file sections)

---

## Parallel Example: After User Story 1

```bash
# After T010 completes, launch in parallel:
Task: "Create SECURITY.md with header and introduction" (T011)
Task: "Create INTEGRATION.md with header and introduction" (T017)
Task: "Verify SpecKit upgrade procedure is clear in README.md" (T024)
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: User Story 1 (refactor README.md)
3. **STOP and VALIDATE**: Can a new user install in <10 minutes?
4. Commit MVP documentation

### Incremental Delivery

1. User Story 1 → Core onboarding works, content extracted
2. User Story 2 → SECURITY.md created, security approval unblocked
3. User Story 3 → INTEGRATION.md created, CI ready
4. User Story 4 → Claude Code section added to INTEGRATION.md
5. User Story 5 → Upgrade procedures documented in README.md
6. Polish → Full validation across all 3 files

---

## Notes

- US1 modifies README.md and extracts content for other files
- US2 creates new file: SECURITY.md
- US3-4 create/extend new file: INTEGRATION.md
- US5 modifies README.md (upgrade sections)
- Validation is manual (use quickstart.md checklist)
- Commit after each user story phase for incremental progress
