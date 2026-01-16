# Data Model: Gitignore Configuration for SpecKit

**Feature**: 003-gitignore-config  
**Date**: 2026-01-16

## Overview

This feature operates on a single file (`.gitignore`) with no persistent data
model. The "entities" are conceptual groupings of gitignore patterns.

## Entities

### Gitignore Block

A contiguous block of lines in `.gitignore` managed by `specify-run`.

**Structure**:
```
# specify-run
<negation patterns>
<blank line>
<ignore patterns>
```

**Attributes**:
- Marker: `# specify-run` (identifies block start)
- Negation patterns: Lines starting with `!`
- Ignore patterns: Lines specifying paths to ignore

### Pattern Categories

| Category | Patterns | Purpose |
|----------|----------|---------|
| Virtualenv | `.venv/` | Exclude Python virtual environment |
| Un-ignore SpecKit | `!.specify/` | Ensure .specify visible |
| Un-ignore scripts | `!.specify/scripts/**` | Version control scripts |
| Un-ignore templates | `!.specify/templates/**` | Version control templates |
| Un-ignore memory | `!.specify/memory/**` | Version control memory |
| Ignore cache | `.specify/cache/` | Exclude cache files |
| Ignore tmp | `.specify/tmp/` | Exclude temporary files |
| Ignore logs | `.specify/logs/` | Exclude log files |
| Ignore runtime | `.specify/.runtime/` | Exclude runtime data |

## State Transitions

### Gitignore State

```
[No .gitignore] --bootstrap--> [.gitignore with specify-run block]
[.gitignore without block] --bootstrap--> [.gitignore with block appended]
[.gitignore with block] --bootstrap--> [No change (idempotent)]
```

## Validation Rules

1. Marker `# specify-run` must appear exactly once (or zero times before first
   bootstrap)
2. Negation patterns must precede ignore patterns within the block
3. No duplicate patterns within the block
