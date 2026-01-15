# Integration Guide

This document covers CI/CD and AI agent integration for `specify-run`.

For installation and usage, see [README.md](README.md).
For security properties, see [SECURITY.md](SECURITY.md).

---

## Overview

The `specify-run` script is designed for seamless integration with:

* **CI/CD systems**: GitHub Actions, GitLab CI, Jenkins, CircleCI, etc.
* **AI coding agents**: Claude Code, Cursor, GitHub Copilot Workspace, etc.
* **Editor automation**: Pre-commit hooks, build tasks, etc.

All integrations use the same canonical invocation: `./specify-run`

---

## GitHub Actions

### Basic workflow

```yaml
name: SpecKit

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  speckit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Run SpecKit
        run: ./specify-run
```

### With caching

For faster subsequent runs, cache the virtualenv:

```yaml
      - name: Cache virtualenv
        uses: actions/cache@v4
        with:
          path: .venv
          key: venv-${{ runner.os }}-${{ hashFiles('specify-run') }}

      - name: Run SpecKit
        run: ./specify-run
```

### Key points

* No `pip install` step required - the script handles installation
* No virtualenv activation required - the script handles execution
* Cache key uses `specify-run` hash to invalidate when version changes

---

## Other CI Systems

The pattern is the same for any CI system:

1. Check out the repository
2. Ensure Python 3.9+ is available
3. Run `./specify-run`

### GitLab CI

```yaml
speckit:
  image: python:3.11
  script:
    - ./specify-run
```

### CircleCI

```yaml
jobs:
  speckit:
    docker:
      - image: cimg/python:3.11
    steps:
      - checkout
      - run: ./specify-run
```

### Jenkins

```groovy
pipeline {
    agent any
    stages {
        stage('SpecKit') {
            steps {
                sh './specify-run'
            }
        }
    }
}
```

### Generic requirements

| Requirement | Notes |
|-------------|-------|
| Python 3.9+ | Must be in PATH |
| Git | For first-run installation |
| Network | Only needed on first run (or after cache clear) |
| Bash | Script requires bash shell |

---

## CI Troubleshooting

### Python not found

**Symptom**: `python: command not found` or `python3: command not found`

**Solution**: Ensure Python is installed and in PATH. Most CI systems provide a
Python setup action or Docker image.

```bash
# Override Python path if needed
PYTHON=python3.11 ./specify-run
```

### Permission denied

**Symptom**: `./specify-run: Permission denied`

**Solution**: The script needs execute permission. Either:

```bash
chmod +x ./specify-run && ./specify-run
```

Or run via bash:

```bash
bash ./specify-run
```

### Network timeout on first run

**Symptom**: Git clone fails during first run

**Solution**: The script clones SpecKit from GitHub on first run. Ensure:

* Network access to github.com
* No proxy blocking Git protocol
* Sufficient timeout for clone operation

### Cache issues

**Symptom**: Old version of SpecKit runs after updating `SPECKIT_REF`

**Solution**: Clear the virtualenv and cache:

```bash
rm -rf .venv
./specify-run
```

In CI, ensure cache key includes the script hash to auto-invalidate.

---

## Claude Code Integration

Claude Code (and similar AI coding agents) must be explicitly instructed to use
`./specify-run` instead of inferring tooling.

### Why explicit instructions matter

AI agents may:

* Guess that `speckit` is a global command
* Try to install packages directly with pip
* Activate virtualenvs manually

All of these break the deterministic execution model. Explicit instructions
prevent these issues.

### Setup

Create or update `.claude/CLAUDE.md` in your repository with instructions for
the agent. See the next section for a complete snippet.

### CLAUDE.md snippet

Copy this into your `.claude/CLAUDE.md` file:

````markdown
# SpecKit Usage

This repository uses `specify-run` for deterministic SpecKit execution.

## Mandatory rules

1. **Always use `./specify-run`** to run SpecKit commands
2. **Never run `speckit` directly** - there is no global installation
3. **Never run `pip install speckit`** - the script handles installation
4. **Never activate `.venv/` manually** - the script handles this

## Examples

```bash
# Correct
./specify-run
./specify-run init
./specify-run <subcommand>

# Incorrect - DO NOT DO THIS
speckit
pip install speckit
source .venv/bin/activate && speckit
```

## Why

The `specify-run` script ensures:
- Pinned SpecKit version (reproducible builds)
- No global dependencies (isolated environment)
- Consistent behavior across all machines
````

---

## Anti-Patterns

These patterns break the deterministic execution model. Agents and automation
**must avoid** them.

| Anti-Pattern | Why It's Wrong | Correct Alternative |
|--------------|----------------|---------------------|
| `speckit` | Relies on global install / PATH | `./specify-run` |
| `pip install speckit` | Bypasses version pinning | Let script handle install |
| `source .venv/bin/activate` | Manual activation is fragile | Script activates automatically |
| `python -m speckit` | Wrong Python, wrong packages | `./specify-run` |
| Modifying `.venv/` | Breaks reproducibility | Delete and re-run script |
| Caching `.venv/` without script hash | Stale cache after version change | Include script in cache key |

### How to detect violations

If an agent produces any of these commands, the CLAUDE.md instructions are
either missing or not being followed. Verify:

1. `.claude/CLAUDE.md` exists and contains the SpecKit rules
2. The agent is reading the instructions file
3. No conflicting instructions override the SpecKit rules
