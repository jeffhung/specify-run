# Research: Preserve Memory on Upgrade

**Feature**: 005-preserve-memory-upgrade
**Date**: 2026-01-17

## Research Topics

### 1. SpecKit Upgrade Mechanism

**Finding**: Pip upgrade only updates the SpecKit CLI. Project files installed
by SpecKit (`.specify/`, agent command files) require a separate command.

**Source**: https://github.com/github/spec-kit/blob/main/docs/upgrade.md

**Re-bootstrap command**:
```bash
specify init --here --force --ai <agent>
```

**Files updated by re-bootstrap**:
- Agent command files (`.claude/commands/`, `.github/prompts/`, etc.)
- `.specify/scripts/`
- `.specify/templates/`
- `.specify/memory/` (including `constitution.md`) - **OVERWRITTEN**

**Files protected**:
- `specs/` directory (excluded from template packages)
- Source code and git history

### 2. Memory Files During Re-bootstrap

**Finding**: Memory files in `.specify/memory/` ARE overwritten by
`specify init --force`. Tested and confirmed:

```bash
# Before init
echo "# My Custom Constitution" > .specify/memory/constitution.md

# After specify init --here --force --ai claude
cat .specify/memory/constitution.md
# Output: [PROJECT_NAME] Constitution (template content)
```

**Decision**: `specify-run` must backup `.specify/memory/` before running
`specify init --force`, then restore after.

### 3. Multiple AI Agents Support

**Finding**: Running `specify init --here --force --ai <agent>` multiple times
with different agents is safe. Each agent's command directory is **additive**:

```bash
specify init --here --force --ai claude   # Creates .claude/commands/
specify init --here --force --ai copilot  # Creates .github/prompts/, preserves .claude/
```

**Tested behavior**:
- `.claude/commands/` preserved when running with `--ai copilot`
- `.github/prompts/` preserved when running with `--ai claude`
- `.specify/` is shared and overwritten each time

**Decision**: During upgrade, run `specify init` once for each AI agent that
has SpecKit command files installed.

### 4. Detecting SpecKit-Provisioned AI Agents

**Finding**: SpecKit command files have a distinctive `speckit.` prefix:
- Claude: `.claude/commands/speckit.*.md`
- Copilot: `.github/prompts/speckit.*.prompt.md`
- Gemini: `.gemini/speckit.*.md` (pattern assumed)
- Cursor: `.cursor/prompts/speckit.*.md` (pattern assumed)

**Decision**: Detect provisioned agents by checking for `speckit.*` files in
agent command directories, not just directory existence.

**Agent detection logic**:
```bash
PROVISIONED_AGENTS=()

if ls "$ROOT/.claude/commands/speckit."*.md >/dev/null 2>&1; then
  PROVISIONED_AGENTS+=("claude")
fi

if ls "$ROOT/.github/prompts/speckit."*.prompt.md >/dev/null 2>&1; then
  PROVISIONED_AGENTS+=("copilot")
fi

# Add more agents as needed
```

### 5. Upgrade Flow for Multiple Agents

**Decision**: Backup memory once, run `specify init` for each agent, restore
memory once at the end.

**Implementation**:
```bash
# Step 1: Detect all provisioned agents
PROVISIONED_AGENTS=()
# ... detection logic ...

# Step 2: Backup memory once (if agents found and memory exists)
MEMORY_BACKUP=""
if [[ ${#PROVISIONED_AGENTS[@]} -gt 0 ]] && [[ -d "$ROOT/.specify/memory" ]]; then
  MEMORY_BACKUP="$(mktemp -d)"
  cp -R "$ROOT/.specify/memory/." "$MEMORY_BACKUP/"
  info "Backed up .specify/memory/"
fi

# Step 3: Re-bootstrap for each agent
for agent in "${PROVISIONED_AGENTS[@]}"; do
  info "Re-bootstrapping for $agent..."
  "$SPECIFY" init --here --force --ai "$agent"
done

# Step 4: Restore memory once
if [[ -n "$MEMORY_BACKUP" ]]; then
  rm -rf "$ROOT/.specify/memory"
  mv "$MEMORY_BACKUP" "$ROOT/.specify/memory"
  info "Preserved .specify/memory/ (project-specific files)"
fi
```

### 6. Agent Command Directory Mapping

**Reference table** (from `specify init --help`):

| Agent | Command Directory | File Pattern |
|-------|------------------|--------------|
| claude | `.claude/commands/` | `speckit.*.md` |
| copilot | `.github/prompts/` | `speckit.*.prompt.md` |
| gemini | `.gemini/` | `speckit.*.md` (TBD) |
| cursor-agent | `.cursor/prompts/` | `speckit.*.md` (TBD) |
| windsurf | `.windsurf/` | TBD |
| codex | `.codex/` | TBD |
| qwen | `.qwen/` | TBD |

**Note**: Only claude and copilot patterns are confirmed. Others need
verification when support is needed.

### 7. Edge Case: No Provisioned Agents Found

**Scenario**: Upgrade triggered but no SpecKit command files found in any
agent directory.

**Possible causes**:
1. User installed SpecKit CLI but never ran `specify init`
2. User deleted SpecKit command files manually
3. User abandoned SpecKit usage

**Decision**: Skip re-bootstrap, just upgrade CLI. User can provision later.

**Rationale**: If we fail with an error telling user to run
`./specify-run init --ai <agent>`, that command would trigger the upgrade
path again and hit the same error. Instead, gracefully skip re-bootstrap.

**Implementation**:
```bash
if [[ ${#PROVISIONED_AGENTS[@]} -eq 0 ]]; then
  info "No SpecKit project files found - skipping re-bootstrap"
  touch "$STAMP"
  echo "[specify-run] SpecKit CLI upgraded to $SPECKIT_REF."
  echo "Run './specify-run init --ai <agent>' to provision project files."
  exit "$EX_TEMPFAIL"
fi
```

### 8. Edge Case: Missing Memory Directory

**Scenario**: `.specify/memory/` does not exist before upgrade.

**Decision**: Skip backup, let `specify init` create memory from templates.

**Rationale**: If memory doesn't exist, there's nothing to preserve. Fresh
templates from the new version are appropriate.

### 9. Failure Recovery

**Decision**: Restore memory backup if any `specify init` fails.

**Implementation**:
```bash
for agent in "${PROVISIONED_AGENTS[@]}"; do
  if ! "$SPECIFY" init --here --force --ai "$agent"; then
    if [[ -n "$MEMORY_BACKUP" ]]; then
      rm -rf "$ROOT/.specify/memory"
      mv "$MEMORY_BACKUP" "$ROOT/.specify/memory"
      info "Restored .specify/memory/ after failed upgrade"
    fi
    die "SpecKit upgrade failed during $agent re-bootstrap"
  fi
done
```

### 10. Upgrade Messages

**Messages** (per FR-007, User Story 3):

**Normal upgrade with agents**:
```
[specify] Installing SpecKit CLI (v0.0.91)
[specify] Backed up .specify/memory/
[specify] Re-bootstrapping for claude...
[specify] Re-bootstrapping for copilot...
[specify] Preserved .specify/memory/ (project-specific files)
[specify] Replaced .specify/scripts/ and .specify/templates/
[specify-run] SpecKit upgraded to v0.0.91.
Please commit these changes, then re-run ./specify-run:
  git add -A && git commit -m "chore: upgrade SpecKit to v0.0.91"
```

**Upgrade with no provisioned agents**:
```
[specify] Installing SpecKit CLI (v0.0.91)
[specify] No SpecKit project files found - skipping re-bootstrap
[specify-run] SpecKit CLI upgraded to v0.0.91.
Run './specify-run init --ai <agent>' to provision project files.
```

## Summary

Key implementation decisions:
1. Pip upgrade only updates CLI; `specify init --here --force --ai X` updates
   project files
2. Backup/restore `.specify/memory/` around `specify init --force`
3. Detect provisioned agents by checking for `speckit.*` command files
4. Run `specify init` for each provisioned agent during upgrade
5. Memory backup/restore happens once (before first init, after last init)
6. Skip re-bootstrap if no provisioned agents found (don't fail)
7. Display clear messages about preservation vs replacement
8. Restore memory on any init failure
