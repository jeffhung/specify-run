# Research: Gitignore Configuration for SpecKit

**Feature**: 003-gitignore-config  
**Date**: 2026-01-16

## Research Topics

### 1. Gitignore Pattern Ordering and Negation

**Decision**: Use negation patterns (`!path`) after ignore patterns to un-ignore
specific directories.

**Rationale**: Git processes `.gitignore` patterns top-to-bottom. Later patterns
override earlier ones. Negation patterns must come after any broader patterns that
might match the same paths. The `!` prefix explicitly un-ignores previously ignored
paths.

**Alternatives Considered**:
- Rely on users not having conflicting patterns: Rejected - fragile and
  unpredictable
- Use `.gitignore` in subdirectories: Rejected - violates single-file principle

### 2. Idempotent Gitignore Modification

**Decision**: Use a marker comment (`# specify-run`) to identify the specify-run
block and check for its presence before appending.

**Rationale**: Checking for the marker prevents duplicate entries on repeated
bootstrap runs. Using `# specify-run` distinguishes from other SpecKit tools that
may use `# SpecKit` markers.

**Alternatives Considered**:
- Use `# SpecKit`: Rejected - may conflict with other SpecKit tooling
- Check each pattern individually: Rejected - complex and error-prone
- Overwrite entire file: Rejected - destroys user entries

### 3. Git Repository Detection

**Decision**: Use `git rev-parse --is-inside-work-tree` to detect if the directory
is under git version control.

**Rationale**: This is the standard, reliable way to detect a git repository. It
handles edge cases like bare repositories and nested git directories correctly.

**Alternatives Considered**:
- Check for `.git` directory: Rejected - doesn't handle worktrees or submodules
- Check for `git` command availability: Insufficient - directory may not be a repo

### 4. File Write Safety

**Decision**: Use atomic append with existence check; warn on read-only files.

**Rationale**: The script should not fail catastrophically if `.gitignore` is
read-only or in a protected location. Warning and continuing allows the bootstrap
to complete.

**Alternatives Considered**:
- Fail hard on write errors: Rejected - prevents bootstrap completion
- Silent skip: Rejected - user unaware of incomplete configuration

### 5. Pattern Specificity

**Decision**: Use the exact patterns from the clarification session:
```
!.specify/
!.specify/scripts/**
!.specify/templates/**
!.specify/memory/**

.specify/cache/
.specify/tmp/
.specify/logs/
.specify/.runtime/
```

**Rationale**: These patterns were explicitly confirmed by the user. The `**`
suffix ensures all contents of the directories are included. The `.runtime/`
hidden directory naming follows existing convention.

**Alternatives Considered**:
- Broader patterns like `.specify/*`: Rejected - less precise control
- Individual file patterns: Rejected - maintenance burden

## Unresolved Items

None. All technical questions resolved.
