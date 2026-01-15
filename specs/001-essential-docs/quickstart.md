# Quickstart: Validating Essential Documentation

**Feature**: 001-essential-docs
**Purpose**: Guide for validating documentation completeness against acceptance criteria

## Documentation Structure

Verify exactly 3 files exist in repository root:
- [ ] README.md (entry point for adopters)
- [ ] SECURITY.md (security reviewer documentation)
- [ ] INTEGRATION.md (CI/agent integration documentation)

## Validation Checklist

### User Story 1: First-Time Adopter Onboarding (P1)

**Test**: Give documentation to a developer unfamiliar with specify-run.

- [ ] Can explain what specify-run is within 2 minutes of reading README.md
- [ ] Successfully provisions SpecKit on first attempt following README.md
- [ ] Finds "why use this" justification within 1 minute in README.md

**How to validate**:
1. Find a colleague unfamiliar with specify-run
2. Share only the README.md
3. Time their understanding and first successful run
4. Target: < 10 minutes total

### User Story 2: Security-Conscious Reviewer (P2)

**Test**: Have security reviewer read SECURITY.md.

- [ ] Can list at least 3 security properties guaranteed
- [ ] Finds explicit version pinning and auditability documentation
- [ ] Can identify specific attacks prevented

**How to validate**:
1. Ask reviewer to list security guarantees after reading SECURITY.md
2. Expected answers include: no global state, explicit upgrades, auditable diffs,
   pinned source, visible tampering

### User Story 3: CI/CD Integration (P2)

**Test**: Copy GitHub Actions example from INTEGRATION.md to a workflow.

- [ ] Example found within 30 seconds in INTEGRATION.md
- [ ] Pipeline runs successfully without modification
- [ ] Troubleshooting guidance available in INTEGRATION.md

**How to validate**:
1. Create test repository
2. Copy example workflow from INTEGRATION.md verbatim
3. Run workflow
4. Verify SpecKit executes successfully

### User Story 4: Claude Code Integration (P2)

**Test**: Follow Claude Code integration instructions in INTEGRATION.md.

- [ ] Clear which file to create/edit (CLAUDE.md in adopter's project)
- [ ] CLAUDE.md snippet provided for copy-paste
- [ ] Configuration results in correct agent behavior
- [ ] Anti-patterns explicitly listed

**How to validate**:
1. Create CLAUDE.md in test project using snippet from INTEGRATION.md
2. Invoke Claude Code
3. Verify it uses ./specify-run for all SpecKit commands

### User Story 5: Version Upgrade Workflow (P3)

**Test**: Follow upgrade instructions in README.md.

- [ ] Step-by-step instructions present for upgrading SpecKit
- [ ] Step-by-step instructions present for upgrading specify-run script
- [ ] Upgrade visible as git commit
- [ ] Rollback possible within 2 minutes

**How to validate**:
1. Edit SPECKIT_REF to new version
2. Commit change
3. Run ./specify-run
4. Verify new version active
5. Revert commit, verify rollback works

## Topic Coverage Verification

### README.md Topics

- [ ] What is specify-run? (FR-001)
- [ ] How it works (FR-002)
- [ ] Why pinned execution (FR-003)
- [ ] Local development usage (FR-007)
- [ ] Upgrade SpecKit (FR-008)
- [ ] Upgrade specify-run script (FR-009)
- [ ] Why virtualenv (FR-010)
- [ ] Why pin SpecKit (FR-011)
- [ ] Why commit script (FR-012)
- [ ] Links to SECURITY.md and INTEGRATION.md

### SECURITY.md Topics

- [ ] Security hardening properties (FR-004)
- [ ] Threat model
- [ ] Attack examples prevented
- [ ] Supply-chain protections

### INTEGRATION.md Topics

- [ ] Claude Code integration with CLAUDE.md snippet (FR-005)
- [ ] GitHub Actions example (FR-006)
- [ ] Other CI systems guidance
- [ ] Troubleshooting
- [ ] Anti-patterns

## Success Criteria Verification

- [ ] SC-001: 10-minute onboarding achieved (README.md only)
- [ ] SC-002: All 12 required topics covered across 3 files
- [ ] SC-003: 3+ hardening properties identifiable in SECURITY.md
- [ ] SC-004: GitHub Actions example from INTEGRATION.md works as-is
- [ ] SC-005: Claude Code integration from INTEGRATION.md works first try
- [ ] SC-006: Documentation uses exactly 3 files: README.md + SECURITY.md + INTEGRATION.md
