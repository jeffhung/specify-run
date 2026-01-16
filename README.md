# `specify-run` — deterministic SpecKit execution with safety

This repository uses a **self-bootstrapping, repository-pinned SpecKit entrypoint**.

There is **no global installation** and **no `$PATH` dependency**.

All humans, CI jobs, editors, and AI agents **must** invoke SpecKit via:

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

