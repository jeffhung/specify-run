# Research: Essential Documentation

**Feature**: 001-essential-docs
**Date**: 2026-01-15

## Documentation Structure Decision

Per user clarification, documentation uses a persona-based 3-file structure:
- **README.md**: Entry point for adopters
- **SECURITY.md**: Security reviewer documentation
- **INTEGRATION.md**: CI/agent integration documentation

All files in repository root with uppercase names.

## Existing Content Analysis

### Current README.md Coverage

| Required Topic (FR) | Current Status | Target File | Gap Analysis |
|---------------------|----------------|-------------|--------------|
| FR-001: What is specify-run | ✅ Covered | README.md | Keep in README |
| FR-002: How it works | ✅ Covered | README.md | Keep in README |
| FR-003: Why pinned execution | ✅ Covered | README.md | Keep in README |
| FR-004: Security hardening | ✅ Covered | SECURITY.md | Move to SECURITY.md, expand |
| FR-005: Claude Code integration | ⚠️ Partial | INTEGRATION.md | Move to INTEGRATION.md, add snippet |
| FR-006: GitHub Actions example | ✅ Covered | INTEGRATION.md | Move to INTEGRATION.md |
| FR-007: Local development usage | ✅ Covered | README.md | Keep in README |
| FR-008: Upgrade SpecKit | ✅ Covered | README.md | Keep in README |
| FR-009: Upgrade specify-run script | ❌ Missing | README.md | Add to README |
| FR-010: Why virtualenv | ⚠️ Partial | README.md | Expand in README |
| FR-011: Why pin SpecKit | ✅ Covered | README.md | Keep in README |
| FR-012: Why commit script | ❌ Missing | README.md | Add to README |

## Content Migration Plan

### README.md Refactoring

**Keep and enhance**:
- Opening paragraph (FR-001)
- "How it works" section (FR-002)
- "Why this exists" section (FR-003)
- "Usage" section (FR-007)
- "SpecKit version pinning" section (FR-008, FR-011)
- "Troubleshooting" section

**Add new content**:
- "Why a virtualenv?" section (FR-010) - explicit rationale
- "Upgrading the specify-run script" section (FR-009)
- "Why commit this script?" section (FR-012)
- Links to SECURITY.md and INTEGRATION.md

**Move to other files**:
- "Security model" section → SECURITY.md
- "CI usage" section → INTEGRATION.md
- "Editor / AI agent integration" section → INTEGRATION.md

### SECURITY.md (New File)

Content from existing README "Security model" section plus:
- Expanded threat model
- Concrete attack examples prevented
- Supply-chain protection details
- Comparison to global install risks

### INTEGRATION.md (New File)

Content from existing README plus:
- GitHub Actions section (expanded from current)
- Claude Code section with complete CLAUDE.md snippet
- Other CI systems (generic guidance)
- Troubleshooting CI-specific issues
- Anti-patterns section

## Decisions

### Decision 1: Claude Code Integration Content

**Decision**: Add concrete CLAUDE.md snippet in INTEGRATION.md.

**Rationale**: Users need copy-paste example for their project's CLAUDE.md.

### Decision 2: Script Upgrade Documentation

**Decision**: Add "Upgrading the specify-run script" section in README.md.

**Rationale**: FR-009 requires this. Distinct from upgrading SpecKit.

### Decision 3: Virtualenv Rationale

**Decision**: Add explicit "Why a virtualenv?" section in README.md.

**Rationale**: FR-010 requires explicit explanation (isolation, reproducibility).

### Decision 4: Script Commitment Rationale

**Decision**: Add "Why commit this script?" section in README.md.

**Rationale**: FR-012 requires this (team consistency, auditability, CI).

### Decision 5: Content Extraction

**Decision**: Extract security and integration content to separate files.

**Rationale**: FR-013 requires persona-based structure. Security reviewers
and DevOps engineers have distinct needs.

## No NEEDS CLARIFICATION Items

All technical decisions resolved. Documentation structure clarified by user.
