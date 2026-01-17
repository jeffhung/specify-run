# Quickstart: Idempotent Security Hardenings

**Feature**: 004-idempotent-hardenings
**Date**: 2026-01-17

## What This Feature Does

After implementation, `./specify-run` will:

1. **Verify security hardenings** on every execution
2. **Detect and fix** missing `.gitignore` patterns
3. **Stop after fix** to allow standalone commits (exit 75)
4. **Fresh provisioning** completes in one pass (no stop)
5. **SpecKit upgrade** also stops for standalone commit

## Scenarios

### Fresh Provisioning (One Pass)

On a fresh clone with no `.venv/`:

```bash
$ ./specify-run init
[specify-run] About to create .venv/ and install SpecKit v0.0.90
Proceed? [Y/n]: y
[specify] Creating virtualenv at .venv
[specify] Installing SpecKit (v0.0.90)
[specify] Configured .gitignore
# Proceeds directly to: specify init
```

All steps complete in one pass. User can commit everything together.

### Normal Operation (Hardenings Correct)

When hardenings are in place:

```bash
$ ./specify-run
# Proceeds directly to SpecKit command
```

No prompts, no stops.

### Gitignore Remediation (Stop After Fix)

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
Please commit these changes, then re-run ./specify-run:
  git add .gitignore && git commit -m "fix: restore specify-run security hardenings"
# Exit code: 75
```

After committing:

```bash
$ git add .gitignore && git commit -m "fix: restore hardenings"
$ ./specify-run
# Now proceeds to SpecKit command
```

### Dirty Hardening Target (Blocked)

If `.gitignore` has uncommitted changes when remediation is needed:

```bash
$ echo "myfile" >> .gitignore  # User edit, not committed
$ ./specify-run
[specify-run] Security hardenings incomplete, but .gitignore has uncommitted changes.
Commit or revert your .gitignore changes first, then re-run.
# Exit code: 78
```

Other dirty files don't block remediation.

### SpecKit Upgrade (Stop After Fix)

When `SPECKIT_REF` is changed:

```bash
$ ./specify-run
[specify-run] About to upgrade SpecKit from v0.0.89 to v0.0.90
Proceed? [Y/n]: y
[specify] Installing SpecKit (v0.0.90)
[specify-run] SpecKit upgraded.
Please commit these changes, then re-run ./specify-run:
  git add -A && git commit -m "chore: upgrade SpecKit to v0.0.90"
# Exit code: 75
```

### Agentic Mode

For AI agents:

```bash
# Fresh provisioning
$ SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="bootstrap=y,gitignore=y" ./specify-run init
# Completes in one pass

# Remediation needed - first attempt
$ SPECIFYRUN_BY_AGENT=1 ./specify-run
[specify-run] Security hardenings incomplete.
Append `gitignore=y` to SPECIFYRUN_ANSWERS environment variable to proceed.
# Exit code: 75

# Retry with answer
$ SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="gitignore=y" ./specify-run
[specify-run] Security hardenings applied.
Please commit these changes, then re-run ./specify-run.
# Exit code: 75 (still needs retry after commit)

# After commit
$ SPECIFYRUN_BY_AGENT=1 ./specify-run
# Proceeds to SpecKit command
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success (specify command completed) |
| 75 | Retry needed (commit changes first, then re-run) |
| 78 | Configuration error (blocked by dirty target or non-interactive) |
| 130 | User declined |

## Answer Keys

| Key | Purpose |
|-----|---------|
| `bootstrap` | Consent for `.venv/` creation |
| `upgrade` | Consent for SpecKit upgrade |
| `gitignore` | Consent for gitignore hardening |

Combine in `SPECIFYRUN_ANSWERS`: `bootstrap=y,gitignore=y`

## Testing the Feature

After implementation, test these scenarios:

1. **Fresh provisioning**: Delete `.venv/`, run `./specify-run init`
2. **Normal operation**: Run on fully provisioned environment
3. **Remediation**: Remove a pattern, run, verify stop after fix
4. **Dirty target**: Edit `.gitignore`, remove pattern, verify blocked
5. **Other dirty files**: Edit `README.md`, remove pattern, verify proceeds
6. **Upgrade**: Change `SPECKIT_REF`, run, verify stop after upgrade
7. **Agentic mode**: Test with `SPECIFYRUN_BY_AGENT=1`
