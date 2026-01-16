# Quickstart: Gitignore Configuration for SpecKit

**Feature**: 003-gitignore-config  
**Date**: 2026-01-16

## What This Feature Does

When you run `./specify-run` in a git repository, it automatically configures
`.gitignore` to:

1. **Ignore** runtime artifacts (cache, tmp, logs, runtime, virtualenv)
2. **Preserve** version-controlled files (memory, templates, scripts)

## Expected Behavior

### First Bootstrap

```bash
$ ./specify-run
[specify-run] About to create .venv/ and install SpecKit v0.0.90
Proceed? [Y/n]: y
[specify] Creating virtualenv at .venv
[specify] Installing SpecKit (v0.0.90)
[specify] Configuring .gitignore
```

### Resulting .gitignore

```gitignore
# ... existing entries preserved ...

# specify-run
.venv/
!.specify/
!.specify/scripts/**
!.specify/templates/**
!.specify/memory/**

.specify/cache/
.specify/tmp/
.specify/logs/
.specify/.runtime/
```

## Verification

After bootstrap, verify configuration:

```bash
# Should NOT be ignored (exit code 1 = not ignored)
git check-ignore .specify/memory/test.md || echo "OK: memory tracked"
git check-ignore .specify/scripts/test.sh || echo "OK: scripts tracked"
git check-ignore .specify/templates/test.md || echo "OK: templates tracked"

# SHOULD be ignored (exit code 0 = ignored)
git check-ignore .specify/cache/test && echo "OK: cache ignored"
git check-ignore .specify/tmp/test && echo "OK: tmp ignored"
git check-ignore .specify/logs/test && echo "OK: logs ignored"
git check-ignore .venv/test && echo "OK: venv ignored"
```

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| No `.gitignore` exists | Creates new file with specify-run block |
| `.gitignore` exists | Appends block (preserves existing entries) |
| Block already exists | No changes (idempotent) |
| Not a git repository | Skips gitignore configuration silently |
| `.gitignore` read-only | Warns and continues bootstrap |
