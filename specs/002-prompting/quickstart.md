# Quickstart: User Consent Prompting

**Feature**: 002-prompting
**Date**: 2026-01-16

## Overview

After this feature is implemented, `specify-run` will prompt for confirmation
before creating or modifying the `.venv/` directory.

## Interactive Usage (Human)

### First Run

```bash
./specify-run
```

Output:
```
[specify-run] About to create .venv/ and install SpecKit v0.0.90
Proceed? [Y/n]: y
[specify] Creating virtualenv at .venv
[specify] Installing SpecKit (v0.0.90)
```

### Declining

```bash
./specify-run
```

Output:
```
[specify-run] About to create .venv/ and install SpecKit v0.0.90
Proceed? [Y/n]: n
```
Exit code: 130

### Upgrade

After editing `SPECKIT_REF` in the script:

```bash
./specify-run
```

Output:
```
[specify-run] About to upgrade SpecKit from v0.0.90 to v0.1.0
Proceed? [Y/n]: y
[specify] Installing SpecKit (v0.1.0)
```

## CI/Automation Usage

Use `SPECIFYRUN_ANSWERS` to skip prompts:

```bash
SPECIFYRUN_ANSWERS="bootstrap=y" ./specify-run
```

Or with subcommand:

```bash
SPECIFYRUN_ANSWERS="bootstrap=y" ./specify-run init
```

## AI Agent Usage (Agentic Mode)

### First Attempt (No Answer Provided)

```bash
SPECIFYRUN_BY_AGENT=1 ./specify-run
```

Output:
```
[specify-run] About to create .venv/ and install SpecKit v0.0.90
Append `bootstrap=y` to SPECIFYRUN_ANSWERS environment variable to proceed.
```
Exit code: 75 (EX_TEMPFAIL)

### Retry with Answer

```bash
SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="bootstrap=y" ./specify-run
```

Output:
```
[specify] Creating virtualenv at .venv
[specify] Installing SpecKit (v0.0.90)
```
Exit code: 0

### Upgrade with Answer

```bash
SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="upgrade=y" ./specify-run
```

### Multiple Answers

If a workflow requires multiple prompts (unlikely but supported):

```bash
SPECIFYRUN_ANSWERS="bootstrap=y,upgrade=y"
```

## Exit Code Reference

| Code | Meaning | Agent Action |
|------|---------|--------------|
| 0 | Success | Continue |
| 75 | Answer required | Retry with `SPECIFYRUN_ANSWERS` |
| 78 | Config error (non-interactive without answers) | Set `SPECIFYRUN_ANSWERS` |
| 130 | User declined | Stop, report to user |

## Environment Variables

| Variable | Description |
|----------|-------------|
| `SPECIFYRUN_BY_AGENT=1` | Enable agentic mode |
| `SPECIFYRUN_ANSWERS="key=value,..."` | Pre-supplied prompt answers |

## Prompt Keys

| Key | When Prompted |
|-----|---------------|
| `bootstrap` | First run, creating `.venv/` |
| `upgrade` | SpecKit version changed |
