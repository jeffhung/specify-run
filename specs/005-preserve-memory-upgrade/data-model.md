# Data Model: Preserve Memory on Upgrade

**Feature**: 005-preserve-memory-upgrade
**Date**: 2026-01-17

## Overview

This feature extends the upgrade flow to handle project files. The data model
focuses on:

1. **Provisioned agents detection** (which AI agents have SpecKit installed)
2. **Memory backup/restore** (temporary storage during upgrade)
3. **Upgrade state transitions**

## Entities

### Provisioned Agents

AI agents that have SpecKit command files installed in the project.

**Detection logic**:
```bash
PROVISIONED_AGENTS=()

if ls "$ROOT/.claude/commands/speckit."*.md >/dev/null 2>&1; then
  PROVISIONED_AGENTS+=("claude")
fi

if ls "$ROOT/.github/prompts/speckit."*.prompt.md >/dev/null 2>&1; then
  PROVISIONED_AGENTS+=("copilot")
fi

# Extend for other agents as needed
```

**Agent mapping table**:

| Agent ID | Command Directory | File Pattern |
|----------|------------------|--------------|
| claude | `.claude/commands/` | `speckit.*.md` |
| copilot | `.github/prompts/` | `speckit.*.prompt.md` |
| gemini | `.gemini/` | `speckit.*.md` |
| cursor-agent | `.cursor/prompts/` | `speckit.*.md` |

### Memory Backup

Temporary backup of `.specify/memory/` during upgrade.

**Structure**:
```bash
MEMORY_BACKUP=""  # Empty if no backup needed

# Create backup
if [[ -d "$ROOT/.specify/memory" ]]; then
  MEMORY_BACKUP="$(mktemp -d)"
  cp -R "$ROOT/.specify/memory/." "$MEMORY_BACKUP/"
fi

# Restore backup
if [[ -n "$MEMORY_BACKUP" ]]; then
  rm -rf "$ROOT/.specify/memory"
  mv "$MEMORY_BACKUP" "$ROOT/.specify/memory"
fi
```

**Lifetime**: Created before first `specify init`, deleted after restore.

### Upgrade State

Extended upgrade states for project file handling.

**States**:
- `CLI_ONLY`: No provisioned agents found, only CLI upgraded
- `FULL_UPGRADE`: Agents found, CLI + project files upgraded

**Detection**:
```bash
if [[ ${#PROVISIONED_AGENTS[@]} -eq 0 ]]; then
  # CLI_ONLY upgrade
else
  # FULL_UPGRADE
fi
```

## State Transitions

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Upgrade Triggered                                   │
│                    (OLD_REF exists, STAMP missing)                          │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Prompt: upgrade     │
                         └─────────────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Upgrade CLI via pip │
                         └─────────────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Detect provisioned  │
                         │ agents              │
                         └─────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
             ┌────────────┐                  ┌────────────┐
             │ No agents  │                  │ Agents     │
             │ found      │                  │ found      │
             └────────────┘                  └────────────┘
                    │                               │
                    ▼                               ▼
             ┌────────────┐                  ┌────────────────┐
             │ Skip       │                  │ Backup memory  │
             │ re-bootstrap│                 │ (if exists)    │
             └────────────┘                  └────────────────┘
                    │                               │
                    │                               ▼
                    │                        ┌────────────────┐
                    │                        │ For each agent:│
                    │                        │ specify init   │
                    │                        │ --here --force │
                    │                        │ --ai $agent    │
                    │                        └────────────────┘
                    │                               │
                    │                               ▼
                    │                        ┌────────────────┐
                    │                        │ Restore memory │
                    │                        └────────────────┘
                    │                               │
                    ▼                               ▼
             ┌────────────────────────────────────────────────┐
             │ Touch STAMP                                    │
             │ Display upgrade message                        │
             │ Exit 75 (commit changes, then re-run)          │
             └────────────────────────────────────────────────┘
```

## File Categories

During upgrade, files are handled differently:

| Category | Location | Behavior |
|----------|----------|----------|
| Memory | `.specify/memory/` | Backed up → Restored (preserved) |
| Scripts | `.specify/scripts/` | Replaced by new version |
| Templates | `.specify/templates/` | Replaced by new version |
| Agent commands | `.claude/commands/`, etc. | Replaced by new version |
| Specs | `specs/` | Not touched (protected by SpecKit) |

## Validation Rules

1. **Agent detection uses file content**: Check for `speckit.*` files, not just
   directory existence.

2. **Memory backup is optional**: Only created if `.specify/memory/` exists.

3. **Restore on failure**: If any `specify init` fails, restore memory before
   exiting.

4. **Multiple agents supported**: Run `specify init` for each provisioned agent.

5. **Order matters for memory**: Backup once before first init, restore once
   after last init.
