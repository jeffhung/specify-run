# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository provides a deterministic, repository-pinned SpecKit entrypoint. There is no global installation - all invocations must go through the wrapper script.

## Commands

**Always use the wrapper script - never call `specify` directly or rely on `$PATH`:**

```bash
./specify-run              # Run SpecKit
./specify-run init         # Initialize SpecKit (first time)
./specify-run <command>    # Any SpecKit subcommand
```

**Reset environment if needed:**
```bash
rm -rf .venv && ./specify-run
```

**Use alternate Python version:**
```bash
PYTHON=python3.11 ./specify-run
```

## Architecture

- `specify-run` - Single-file bash entrypoint that bootstraps everything
- `.venv/` - Auto-created virtualenv (not committed)
- `.specify/` - SpecKit project files
- `.claude/commands/` - SpecKit slash commands for Claude Code

The wrapper script:
1. Creates `.venv/` if missing
2. Installs SpecKit from a pinned Git ref (configured in the script)
3. Caches installation state via stamp file
4. Executes SpecKit from the pinned environment

## SpecKit Version Pinning

The pinned version is declared inside `specify-run`:
```bash
SPECKIT_GIT="https://github.com/github/spec-kit.git"
SPECKIT_REF="v0.0.90"
```

To upgrade: edit `SPECKIT_REF` and commit. Next run auto-upgrades.

## Agentic Mode

When invoking `./specify-run` as an AI agent, set these environment variables:

```bash
SPECIFYRUN_BY_AGENT=1 ./specify-run
```

If a prompt requires consent, the script prints a hint and exits with code 75:
```
[specify-run] About to create .venv/ and install SpecKit v0.0.90
Append `bootstrap=y` to SPECIFYRUN_ANSWERS environment variable to proceed.
```

Retry with the answer:
```bash
SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="bootstrap=y" ./specify-run
```

For multiple prompts, combine answers: `SPECIFYRUN_ANSWERS="bootstrap=y,upgrade=y"`

Exit codes:
- 0: Success
- 75: Retry needed (provide answer via `SPECIFYRUN_ANSWERS`)
- 78: Configuration error (non-interactive without answers)
- 130: User declined

## Active Technologies
- Markdown (documentation only) + None (pure documentation) (001-essential-docs)
- Bash (POSIX-compatible with bashisms) + None (self-contained script) (002-prompting)
- N/A (stateless, uses stamp file for version tracking) (002-prompting)
- File-based (`.gitignore` at repository root) (003-gitignore-config)

## Recent Changes
- 001-essential-docs: Added Markdown (documentation only) + None (pure documentation)
