# Quickstart: Idempotent Security Hardenings

**Feature**: 004-idempotent-hardenings
**Date**: 2026-01-17

## What This Feature Does

After this feature is implemented, `./specify-run` will verify security
hardenings (`.gitignore` patterns) on every execution, not just during initial
setup. If patterns are missing or modified, the script will:

1. Detect the deviation
2. Explain what's wrong
3. Prompt for consent to fix
4. Apply fixes (if working copy is clean)
5. Stop and ask you to commit before continuing

## Normal Operation (Hardenings Correct)

When hardenings are in place, you see no change:

```bash
$ ./specify-run
# Proceeds directly to SpecKit command
```

## Remediation Flow (Hardenings Missing)

If someone accidentally removes a `.gitignore` pattern:

```bash
$ ./specify-run
[specify-run] Security hardenings incomplete.
Missing .gitignore patterns:
  - .venv/
  - .specify/cache/
[specify-run] About to restore missing .gitignore patterns
Proceed? [Y/n]: y
[specify-run] Security hardenings applied.
Please commit these changes before re-running:
  git add .gitignore && git commit -m "fix: restore specify-run security hardenings"
```

After committing:

```bash
$ git add .gitignore && git commit -m "fix: restore specify-run security hardenings"
$ ./specify-run
# Now proceeds to SpecKit command
```

## Dirty Working Copy (Remediation Blocked)

If you have uncommitted changes when remediation is needed:

```bash
$ echo "test" > newfile.txt  # Create uncommitted change
$ ./specify-run
[specify-run] Security hardenings incomplete, but working copy is dirty.
Commit or stash your changes first, then re-run to apply security fixes.
Alternatively, use a separate worktree/branch for re-hardening.
```

The script stops without making any changes.

## Agentic Mode

For AI agents:

```bash
# First attempt - gets hint
$ SPECIFYRUN_BY_AGENT=1 ./specify-run
[specify-run] Security hardenings incomplete.
Append `gitignore=y` to SPECIFYRUN_ANSWERS environment variable to proceed.
# Exit code: 75

# Retry with answer
$ SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="gitignore=y" ./specify-run
[specify-run] Security hardenings applied.
Please commit these changes before re-running.
# Exit code: 0
```

Note: The same `gitignore` key works for both initial provision (during
bootstrap) and remediation (fixing missing patterns). This is intentional—both
are the same idempotent operation.

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success (including after applying fixes) |
| 75 | Retry needed (agentic mode, provide answer) |
| 78 | Configuration error (non-interactive without answers) |
| 130 | User declined fix |

## Testing the Feature

After implementation, test these scenarios:

1. **Clean state**: Remove the gitignore block, commit, run `./specify-run`
2. **Dirty state**: Make changes, remove pattern, run without committing
3. **Agentic mode**: Test with `SPECIFYRUN_BY_AGENT=1`
4. **Partial missing**: Remove just one pattern, verify only it's reported
