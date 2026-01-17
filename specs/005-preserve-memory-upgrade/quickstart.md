# Quickstart: Preserve Memory on Upgrade

**Feature**: 005-preserve-memory-upgrade
**Date**: 2026-01-17

## What This Feature Does

After implementation, SpecKit upgrades will:

1. **Preserve memory files** in `.specify/memory/` across upgrades
2. **Replace scripts and templates** with new version
3. **Support multiple AI agents** (claude, copilot, etc.)
4. **Display clear messages** about what was preserved vs replaced

## Scenarios

### Normal Upgrade (Single Agent)

When `SPECKIT_REF` is changed in a project with one AI agent:

```bash
$ ./specify-run
[specify-run] About to upgrade SpecKit from v0.0.90 to v0.0.91
Proceed? [Y/n]: y
[specify] Installing SpecKit CLI (v0.0.91)
[specify] Backed up .specify/memory/
[specify] Re-bootstrapping for claude...
[specify] Preserved .specify/memory/ (project-specific files)
[specify] Replaced .specify/scripts/ and .specify/templates/
[specify-run] SpecKit upgraded to v0.0.91.
Please commit these changes, then re-run ./specify-run:
  git add -A && git commit -m "chore: upgrade SpecKit to v0.0.91"
# Exit code: 75
```

After committing:

```bash
$ git add -A && git commit -m "chore: upgrade SpecKit to v0.0.91"
$ ./specify-run
# Proceeds to SpecKit command
```

### Upgrade with Multiple Agents

Project provisioned with both Claude and Copilot:

```bash
$ ./specify-run
[specify-run] About to upgrade SpecKit from v0.0.90 to v0.0.91
Proceed? [Y/n]: y
[specify] Installing SpecKit CLI (v0.0.91)
[specify] Backed up .specify/memory/
[specify] Re-bootstrapping for claude...
[specify] Re-bootstrapping for copilot...
[specify] Preserved .specify/memory/ (project-specific files)
[specify] Replaced .specify/scripts/ and .specify/templates/
[specify-run] SpecKit upgraded to v0.0.91.
Please commit these changes, then re-run ./specify-run:
  git add -A && git commit -m "chore: upgrade SpecKit to v0.0.91"
# Exit code: 75
```

### Upgrade with No Provisioned Agents

Project has SpecKit CLI installed but never ran `specify init`:

```bash
$ ./specify-run
[specify-run] About to upgrade SpecKit from v0.0.90 to v0.0.91
Proceed? [Y/n]: y
[specify] Installing SpecKit CLI (v0.0.91)
[specify] No SpecKit project files found - skipping re-bootstrap
[specify-run] SpecKit CLI upgraded to v0.0.91.
Run './specify-run init --ai <agent>' to provision project files.
# Exit code: 75
```

User can then provision:

```bash
$ ./specify-run init --ai claude
# Provisions fresh project files
```

### Fresh Install (No Change)

Fresh provisioning behavior is unchanged:

```bash
$ ./specify-run init --ai claude
[specify-run] About to create .venv/ and install SpecKit v0.0.91
Proceed? [Y/n]: y
[specify] Creating virtualenv at .venv
[specify] Installing SpecKit (v0.0.91)
[specify] Configured .gitignore
# Proceeds to: specify init --ai claude
# All files created from templates (nothing to preserve)
```

### Agentic Mode

For AI agents:

```bash
# Upgrade with known agents
$ SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="upgrade=y" ./specify-run
[specify] Installing SpecKit CLI (v0.0.91)
[specify] Backed up .specify/memory/
[specify] Re-bootstrapping for claude...
[specify] Preserved .specify/memory/ (project-specific files)
[specify-run] SpecKit upgraded to v0.0.91.
Please commit these changes, then re-run ./specify-run:
  git add -A && git commit -m "chore: upgrade SpecKit to v0.0.91"
# Exit code: 75

# After commit
$ SPECIFYRUN_BY_AGENT=1 ./specify-run
# Proceeds to SpecKit command
```

## What Gets Preserved

| Directory | Behavior | Rationale |
|-----------|----------|-----------|
| `.specify/memory/` | **Preserved** | Project-specific knowledge |
| `.specify/scripts/` | Replaced | SpecKit-managed tooling |
| `.specify/templates/` | Replaced | SpecKit-managed templates |
| `.claude/commands/` | Replaced | SpecKit-managed commands |
| `.github/prompts/` | Replaced | SpecKit-managed prompts |
| `specs/` | Protected | User content (by SpecKit) |

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success (specify command completed) |
| 75 | Retry needed (commit changes first, then re-run) |
| 78 | Configuration error |
| 130 | User declined |

## Answer Keys

| Key | Purpose |
|-----|---------|
| `bootstrap` | Consent for `.venv/` creation |
| `upgrade` | Consent for SpecKit upgrade |
| `gitignore` | Consent for gitignore hardening |

## Testing the Feature

After implementation, test these scenarios:

1. **Normal upgrade**: Change `SPECKIT_REF`, run, verify memory preserved
2. **Multiple agents**: Provision with claude + copilot, upgrade, verify both
   agent directories updated
3. **No agents**: Delete agent command files, upgrade, verify CLI-only upgrade
4. **Memory content**: Modify constitution.md, upgrade, verify unchanged
5. **Scripts replaced**: Modify a script, upgrade, verify overwritten
6. **Agentic mode**: Test with `SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="upgrade=y"`
