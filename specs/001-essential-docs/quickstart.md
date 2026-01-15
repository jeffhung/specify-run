# Quickstart: Validating Essential Documentation

**Feature**: 001-essential-docs
**Purpose**: Guide for validating documentation completeness against acceptance criteria

## Validation Checklist

### User Story 1: First-Time Adopter Onboarding (P1)

**Test**: Give documentation to a developer unfamiliar with specify-run.

- [ ] Can explain what specify-run is within 2 minutes of reading
- [ ] Successfully provisions SpecKit on first attempt following instructions
- [ ] Finds "why use this" justification within 1 minute

**How to validate**:
1. Find a colleague unfamiliar with specify-run
2. Share only the README.md
3. Time their understanding and first successful run
4. Target: < 10 minutes total

### User Story 2: Security-Conscious Reviewer (P2)

**Test**: Have security reviewer read security section.

- [ ] Can list at least 3 security properties guaranteed
- [ ] Finds explicit version pinning and auditability documentation
- [ ] Can identify specific attacks prevented

**How to validate**:
1. Ask reviewer to list security guarantees after reading
2. Expected answers include: no global state, explicit upgrades, auditable diffs,
   pinned source, visible tampering

### User Story 3: CI/CD Integration (P2)

**Test**: Copy GitHub Actions example to a workflow.

- [ ] Example found within 30 seconds
- [ ] Pipeline runs successfully without modification
- [ ] Troubleshooting guidance available

**How to validate**:
1. Create test repository
2. Copy example workflow verbatim
3. Run workflow
4. Verify SpecKit executes successfully

### User Story 4: Claude Code Integration (P2)

**Test**: Follow Claude Code integration instructions.

- [ ] Clear which file to create/edit (CLAUDE.md in adopter's project)
- [ ] Configuration results in correct agent behavior
- [ ] Anti-patterns explicitly listed

**How to validate**:
1. Create CLAUDE.md in test project using documented snippet
2. Invoke Claude Code
3. Verify it uses ./specify-run for all SpecKit commands

### User Story 5: Version Upgrade Workflow (P3)

**Test**: Follow upgrade instructions for SpecKit.

- [ ] Step-by-step instructions present
- [ ] Upgrade visible as git commit
- [ ] Rollback possible within 2 minutes

**How to validate**:
1. Edit SPECKIT_REF to new version
2. Commit change
3. Run ./specify-run
4. Verify new version active
5. Revert commit, verify rollback works

## Topic Coverage Verification

Verify README.md contains all 12 required topics:

- [ ] What is specify-run? (FR-001)
- [ ] How it works (FR-002)
- [ ] Why pinned execution (FR-003)
- [ ] Security hardening (FR-004)
- [ ] Claude Code integration (FR-005)
- [ ] GitHub Actions example (FR-006)
- [ ] Local development usage (FR-007)
- [ ] Upgrade SpecKit (FR-008)
- [ ] Upgrade specify-run script (FR-009)
- [ ] Why virtualenv (FR-010)
- [ ] Why pin SpecKit (FR-011)
- [ ] Why commit script (FR-012)

## Success Criteria Verification

- [ ] SC-001: 10-minute onboarding achieved
- [ ] SC-002: All 12 topics covered
- [ ] SC-003: 3+ hardening properties identifiable
- [ ] SC-004: GitHub Actions example works as-is
- [ ] SC-005: Claude Code integration works first try
- [ ] SC-006: All content fits in README.md only
