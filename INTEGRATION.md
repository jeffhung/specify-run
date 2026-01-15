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
