# `specify-run` — deterministic SpecKit execution with safety

`specify-run` is a single-file wrapper script that provides **deterministic,
version-pinned SpecKit execution**. Copy it into any project to get:

* **Reproducible builds**: Same SpecKit version across all developers, CI, and time
* **Zero setup friction**: Works immediately after copying—no global install required
* **Supply-chain visibility**: Version changes are explicit commits, not silent upgrades
* **AI agent compatibility**: One canonical command that agents can follow reliably

```bash
./specify-run <command>
```

**See also**: [SECURITY.md](SECURITY.md) for threat model and hardening |
[INTEGRATION.md](INTEGRATION.md) for CI and Claude Code setup

---

## Why this exists

Global CLI installs cause:

* version drift between developers
* non-reproducible CI behavior
* hidden supply-chain risk
* fragile AI / editor automation

This repository solves that by:

* pinning SpecKit to a known version
* bootstrapping it automatically
* enforcing a single canonical entrypoint

The result is **deterministic, auditable, and agent-safe** tooling.

---

## How it works

The `./specify-run` script is a **single-file entrypoint** that:

1. Ensures Python is available
2. Creates a project-local virtualenv (`.venv/`) if missing
3. Installs SpecKit from a **pinned Git ref**
4. Caches installation state
5. Executes SpecKit from the pinned environment

No manual setup is required.

---

## Requirements

* Python **3.9+**
* Git
* Network access (first run only)

> No global Python packages are used.

---

## Usage

### Run SpecKit

```bash
./specify-run
```

### Initialize SpecKit (first time)

```bash
./specify-run init
```

### Any SpecKit command

```bash
./specify-run <subcommand> [args...]
```

---

## Canonical rule (important)

> **Never call `specify` directly.**
> **Never rely on `$PATH`.**
> **Never activate a virtualenv manually.**

The **only supported invocation** is:

```bash
./specify-run
```

This rule applies to:

* humans
* CI
* editors
* AI agents (Claude Code, etc.)

---

## Setup

Copy the `specify-run` script into your project:

```bash
curl -O https://raw.githubusercontent.com/.../specify-run
chmod +x specify-run
```

The script can be placed at repository root or any subdirectory where you want
SpecKit provisioned.

## First Run

On first run, the script automatically:

* creates `.venv/` in the script's directory
* installs the pinned SpecKit version
* caches the installation

```bash
./specify-run
```

Subsequent runs are fast and offline-safe. No manual installation of SpecKit
or its dependencies is required.

---

## SpecKit version pinning

The pinned SpecKit version is declared **inside the script**:

```bash
SPECKIT_GIT="https://github.com/github/spec-kit.git"
SPECKIT_REF="v0.3.2"
```

### Upgrade SpecKit

1. Edit `SPECKIT_REF`
2. Commit the change
3. Done

```diff
-SPECKIT_REF="v0.3.2"
+SPECKIT_REF="v0.4.0"
```

The next run will automatically upgrade.

### What happens during upgrade

When upgrading SpecKit, the script:

1. Installs the new SpecKit CLI version
2. Detects which AI agents are provisioned (claude, copilot, etc.)
3. Backs up `.specify/memory/` (project-specific files)
4. Re-bootstraps project files for each agent (`specify init --here --force`)
5. Restores `.specify/memory/`
6. Exits with code 75, prompting you to commit

| Directory | Behavior | Rationale |
| --------- | -------- | --------- |
| `.specify/memory/` | **Preserved** | Project-specific knowledge |
| `.specify/scripts/` | Replaced | SpecKit-managed tooling |
| `.specify/templates/` | Replaced | SpecKit-managed templates |
| `.claude/commands/` | Replaced | SpecKit-managed commands |

If no agents are provisioned, the script upgrades only the CLI and skips
re-bootstrapping.

### Upgrade the specify-run script

The `specify-run` script itself may receive updates (bug fixes, new features).

To upgrade the script:

1. Copy the latest version from the upstream repository
2. Replace your `specify-run` file
3. Commit the change

```bash
curl -o specify-run https://raw.githubusercontent.com/.../specify-run
chmod +x specify-run
git add specify-run && git commit -m "Upgrade specify-run script"
```

> After upgrading the script, delete `.venv/` to ensure a clean reinstall.

### Rollback

Both SpecKit versions and script changes are tracked in git. To rollback:

**Rollback SpecKit version:**

```bash
git revert <commit-that-changed-SPECKIT_REF>
rm -rf .venv
./specify-run
```

**Rollback script upgrade:**

```bash
git checkout <previous-commit> -- specify-run
rm -rf .venv
./specify-run
```

> Always delete `.venv/` after any rollback to ensure clean state.

---

## Reproducibility guarantees

| Property             | Guaranteed |
| -------------------- | ---------- |
| Same SpecKit version | ✅          |
| Same invocation path | ✅          |
| CI parity            | ✅          |
| Agent parity         | ✅          |
| No global state      | ✅          |

---

## Why a virtualenv?

The script creates a project-local `.venv/` directory because:

* **Isolation**: SpecKit and its dependencies never conflict with system Python or other projects
* **Reproducibility**: The exact same environment is recreated on every machine
* **No sudo required**: Everything installs in user space, no root privileges needed
* **Clean removal**: Delete `.venv/` to completely remove all installed packages

> The virtualenv is an implementation detail. You never need to activate it manually.

---

## Why commit this script?

Commit the `specify-run` script to your repository. Do **not** commit `.venv/`.

Benefits of committing the script:

* **Team consistency**: All developers use the exact same bootstrap logic
* **Code review**: Changes to tooling are visible in pull requests
* **CI simplicity**: No external download step required during builds
* **Auditability**: Full history of when and why the script changed
* **No network dependency**: Works offline after initial clone

> The script is intentionally simple and readable. Review it before adopting.

---

## Directory layout

```text
your-repo/
├── specify-run      # canonical entrypoint (this script)
├── .venv/           # auto-created, not committed
├── .specify/        # SpecKit project files
├── .claude/         # agent instructions
└── README.md
```

---

## Troubleshooting

### Python not found

Set an explicit Python interpreter:

```bash
PYTHON=python3.11 ./specify-run
```

### Reset environment

```bash
rm -rf .venv
./specify-run
```

---

## Prompting behavior

The script prompts for consent before modifying the filesystem:

| Scenario | Prompt shown |
| -------- | ------------ |
| First run (no `.venv/`) | "About to create .venv/ and install SpecKit" |
| Version upgrade | "About to upgrade SpecKit from X to Y" |
| Already set up | No prompt (pass-through to SpecKit) |

### Non-interactive mode (CI)

Provide pre-authorized answers via environment variable:

```bash
SPECIFYRUN_ANSWERS="bootstrap=y" ./specify-run
```

### Agentic mode (AI agents)

AI agents should set `SPECIFYRUN_BY_AGENT=1`. If consent is needed, the script
prints a hint and exits with code 75:

```bash
SPECIFYRUN_BY_AGENT=1 ./specify-run
# Output: Append `bootstrap=y` to SPECIFYRUN_ANSWERS environment variable to proceed.
# Exit code: 75

SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="bootstrap=y" ./specify-run
# Proceeds with installation
```

### Exit codes

| Code | Name | Meaning |
| ---- | ---- | ------- |
| 0 | Success | SpecKit executed successfully |
| 75 | Retry needed | Preparation incomplete; take action then re-run |
| 78 | Config error | Cannot proceed due to configuration issue |
| 130 | Declined | User declined a prompt |

Exit code 75 (EX_TEMPFAIL) indicates `specify-run` stopped before reaching
SpecKit. The requested `specify` operation has **not** run. Common scenarios:

* Consent needed → provide answer via `SPECIFYRUN_ANSWERS`, then re-run
* Security hardenings applied → commit changes, then re-run
* SpecKit upgraded → commit changes, then re-run

Exit code 78 (EX_CONFIG) indicates a blocking configuration issue:

* `.gitignore` has uncommitted changes and hardenings are incomplete
* Non-interactive mode without required answers

---

## Security hardenings

The script enforces security hardenings on every execution:

### What gets verified

* `.gitignore` contains required patterns (`.venv/`, `.specify/cache/`, etc.)
* SpecKit-managed files are properly excluded from version control

### Behavior

| Scenario | Action |
| -------- | ------ |
| Hardenings correct | Proceed silently to SpecKit |
| Hardenings missing (fresh) | Apply during bootstrap, no stop |
| Hardenings missing (existing) | Prompt, fix, stop with exit 75 |
| `.gitignore` has uncommitted changes | Block with exit 78 |

### Stop-after-fix

When fixing drifted hardenings on an existing environment, the script stops
after applying fixes and instructs you to commit:

```bash
./specify-run
# [specify-run] Security hardenings applied.
# Please commit these changes, then re-run ./specify-run:
#   git add .gitignore && git commit -m "fix: restore specify-run security hardenings"
# Exit code: 75
```

This ensures hardening changes are committed separately from SpecKit operations.

### Agentic mode

```bash
# First attempt - gets hint
SPECIFYRUN_BY_AGENT=1 ./specify-run
# Append `gitignore=y` to SPECIFYRUN_ANSWERS environment variable to proceed.
# Exit code: 75

# Retry with answer
SPECIFYRUN_BY_AGENT=1 SPECIFYRUN_ANSWERS="gitignore=y" ./specify-run
# Applies fix, exits 75 with commit instructions
```

### Answer keys

| Key | Purpose |
| --- | ------- |
| `bootstrap` | Consent for `.venv/` creation and SpecKit install |
| `upgrade` | Consent for SpecKit version upgrade |
| `gitignore` | Consent for `.gitignore` hardening |

Combine multiple: `SPECIFYRUN_ANSWERS="bootstrap=y,gitignore=y"`

---

## Design principles (for maintainers)

* **Entry point > environment**
* **Declaration > discovery**
* **Pinned > global**
* **Automation > documentation**
* **Agents are first-class users**

This script is intentionally boring.
That is a feature.

---

## License

This script is part of this repository and follows the repository license.

