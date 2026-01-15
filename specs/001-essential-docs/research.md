# Research: Essential Documentation

**Feature**: 001-essential-docs
**Date**: 2026-01-15

## Existing README Analysis

The current README.md already covers many required topics. This research
documents coverage gaps and enhancement opportunities.

### Current Coverage Assessment

| Required Topic (FR) | Current Status | Gap Analysis |
|---------------------|----------------|--------------|
| FR-001: What is specify-run | ✅ Covered | Opening paragraph explains purpose |
| FR-002: How it works | ✅ Covered | "How it works" section with 5-step flow |
| FR-003: Why pinned execution | ✅ Covered | "Why this exists" section |
| FR-004: Security hardening | ✅ Covered | "Security model" section |
| FR-005: Claude Code integration | ⚠️ Partial | Mentions file path but lacks actionable content |
| FR-006: GitHub Actions example | ✅ Covered | "CI usage" section with example |
| FR-007: Local development usage | ✅ Covered | "Usage" section |
| FR-008: Upgrade SpecKit | ✅ Covered | "SpecKit version pinning" section |
| FR-009: Upgrade specify-run script | ❌ Missing | No guidance on upgrading the script itself |
| FR-010: Why virtualenv | ⚠️ Partial | Mentioned but rationale not explicit |
| FR-011: Why pin SpecKit | ✅ Covered | "Why this exists" + reproducibility table |
| FR-012: Why commit script | ❌ Missing | Not explicitly explained |

### Decisions

#### Decision 1: Claude Code Integration Content

**Decision**: Add concrete CLAUDE.md snippet that adopters can copy to their project.

**Rationale**: Current documentation says "use .claude/instructions.md" but doesn't
show what content to put there. Users need copy-paste example.

**Alternatives considered**:
- Link to external docs: Rejected (adds friction, may become stale)
- Template file: Rejected (user clarification chose README-only approach)

#### Decision 2: Script Upgrade Documentation

**Decision**: Add section explaining how to upgrade the specify-run script itself.

**Rationale**: FR-009 requires this. Users need to know:
1. Where to get the latest script
2. How to replace it safely
3. Difference between upgrading SpecKit vs upgrading the script

**Alternatives considered**:
- Omit (rely on users figuring it out): Rejected (violates FR-009)
- Separate upgrade guide: Rejected (consolidation requirement FR-013)

#### Decision 3: Virtualenv Rationale

**Decision**: Add explicit explanation of why virtualenv is used.

**Rationale**: Technical users will ask "why not use system Python?" Need clear
answer: isolation prevents conflicts, enables reproducibility, allows pinning.

**Alternatives considered**:
- Assume users understand: Rejected (FR-010 requires explicit explanation)

#### Decision 4: Script Commitment Rationale

**Decision**: Add section explaining why script must be committed to repository.

**Rationale**: FR-012 requires this. Key points:
- Ensures all team members use same script version
- Visible in code review
- No external download step during CI
- Auditable history

**Alternatives considered**:
- Assume it's obvious: Rejected (explicit documentation required)

### Content Structure Recommendation

Enhance existing README with these additions/modifications:

1. **Add to "Why this exists"**: Brief mention of why script is committed
2. **Add new section "Why a virtualenv?"**: 3-4 bullet rationale
3. **Enhance "Editor / AI agent integration"**: Full CLAUDE.md snippet example
4. **Add new section "Upgrading the specify-run script"**: Step-by-step guide
5. **Add new section "Why commit this script?"**: Explicit rationale

### No NEEDS CLARIFICATION Items

All technical decisions can be made based on existing context and best practices.
No external research required for this documentation feature.
