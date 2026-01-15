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

- **Primary deliverable**: README.md at repository root
- **Validation guide**: specs/001-essential-docs/quickstart.md

---

## Phase 1: Setup

**Purpose**: Review existing documentation and prepare for enhancements

- [ ] T001 Review existing README.md structure and identify insertion points
- [ ] T002 Review research.md gap analysis for required additions

**Checkpoint**: Ready to begin content enhancements

---

## Phase 2: User Story 1 - First-Time Adopter Onboarding (Priority: P1)

**Goal**: Ensure developers understand what specify-run is, why they need it,
and how to install it within 10 minutes.

**Independent Test**: Give docs to unfamiliar developer, measure time to first
successful SpecKit run.

### Implementation for User Story 1

- [ ] T003 [US1] Verify "What is specify-run" paragraph is clear in README.md
- [ ] T004 [US1] Verify "How it works" section explains bootstrap flow in README.md
- [ ] T005 [US1] Verify "Why this exists" section covers global install problems in README.md
- [ ] T006 [US1] Add "Why a virtualenv?" section explaining isolation benefits in README.md
- [ ] T007 [US1] Add "Why commit this script?" section with rationale in README.md

**Checkpoint**: First-time adopter can understand and install in <10 minutes

---

## Phase 3: User Story 2 - Security-Conscious Reviewer (Priority: P2)

**Goal**: Security reviewers can identify hardening properties and understand
threat mitigations.

**Independent Test**: Security reviewer lists 3+ security properties after reading.

### Implementation for User Story 2

- [ ] T008 [US2] Verify "Security model" section lists explicit properties in README.md
- [ ] T009 [US2] Add concrete attack examples prevented by this design in README.md
- [ ] T010 [US2] Verify supply-chain protection documentation in README.md

**Checkpoint**: Security reviewer can identify 3+ hardening properties

---

## Phase 4: User Story 3 - CI/CD Integration (Priority: P2)

**Goal**: DevOps engineers can copy-paste GitHub Actions example that works
immediately.

**Independent Test**: Copy example to workflow, verify pipeline succeeds.

### Implementation for User Story 3

- [ ] T011 [US3] Verify GitHub Actions example is complete and correct in README.md
- [ ] T012 [US3] Add CI troubleshooting tips (Python version, permissions) in README.md

**Checkpoint**: GitHub Actions example works when copy-pasted

---

## Phase 5: User Story 4 - Claude Code Integration (Priority: P2)

**Goal**: Teams using Claude Code can configure their agent correctly with
copy-paste instructions.

**Independent Test**: Follow instructions, verify agent uses ./specify-run
exclusively.

### Implementation for User Story 4

- [ ] T013 [US4] Enhance "Editor / AI agent integration" section in README.md
- [ ] T014 [US4] Add complete CLAUDE.md snippet that adopters can copy in README.md
- [ ] T015 [US4] Add anti-patterns section (what agents must NOT do) in README.md

**Checkpoint**: Claude Code integration works on first configuration attempt

---

## Phase 6: User Story 5 - Version Upgrade Workflow (Priority: P3)

**Goal**: Maintainers can safely upgrade SpecKit and the script itself with
clear procedures.

**Independent Test**: Follow upgrade instructions, verify new version active,
rollback in <2 minutes.

### Implementation for User Story 5

- [ ] T016 [US5] Verify SpecKit upgrade procedure is clear in README.md
- [ ] T017 [US5] Add "Upgrading the specify-run script" section in README.md
- [ ] T018 [US5] Add rollback procedure documentation in README.md

**Checkpoint**: Upgrade and rollback procedures work as documented

---

## Phase 7: Polish & Validation

**Purpose**: Final review and validation against acceptance criteria

- [ ] T019 Review full README.md for consistency and flow
- [ ] T020 Verify all 12 required topics are covered per research.md checklist
- [ ] T021 Run quickstart.md validation checklist
- [ ] T022 Fix typo in README.md title ("sefety" → "safety")

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **User Stories (Phases 2-6)**: Can start after Setup
  - Stories can proceed in parallel (different sections of README.md)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Phase 7)**: Depends on all user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: No dependencies - core onboarding content
- **User Story 2 (P2)**: Independent - security section
- **User Story 3 (P2)**: Independent - CI section
- **User Story 4 (P2)**: Independent - agent integration section
- **User Story 5 (P3)**: Independent - upgrade procedures section

### Parallel Opportunities

All user story phases can be worked on in parallel since they modify
different sections of README.md:

- US1: Overview, virtualenv, commit rationale sections
- US2: Security model section
- US3: CI usage section
- US4: Editor/AI agent integration section
- US5: Upgrade sections

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: User Story 1
3. **STOP and VALIDATE**: Can a new user install in <10 minutes?
4. Commit MVP documentation

### Incremental Delivery

1. User Story 1 → Core onboarding works
2. User Story 2 → Security approval unblocked
3. User Story 3 → CI integration ready
4. User Story 4 → AI agent integration ready
5. User Story 5 → Maintenance procedures documented
6. Polish → Full validation complete

---

## Notes

- All tasks modify README.md at repository root
- No code changes required
- Validation is manual (use quickstart.md checklist)
- Commit after each user story phase for incremental progress
