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
