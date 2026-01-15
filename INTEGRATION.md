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
