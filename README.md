# `specify-run` — deterministic SpecKit execution with sefety

This repository uses a **self-bootstrapping, repository-pinned SpecKit entrypoint**.

There is **no global installation** and **no `$PATH` dependency**.

All humans, CI jobs, editors, and AI agents **must** invoke SpecKit via:

```bash
./specify-run <command>
```

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

## Installation & Provisioning

There is **nothing to install manually**.

On first run:

```bash
./specify-run
```

The script will:

* create `.venv/`
* install the pinned SpecKit version
* cache the install

Subsequent runs are fast and offline-safe.

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

## Security model

This design intentionally avoids:

* global binaries
* hidden PATH resolution
* invisible upgrades

Security properties:

* SpecKit source is pinned and reviewable
* Upgrades are explicit git diffs
* Tooling behavior is deterministic
* Tampering is visible in version control

For higher security:

* pin to a commit SHA instead of a tag
* vendor SpecKit instead of installing from Git

---

## CI usage

Example (GitHub Actions):

```yaml
- name: Run SpecKit
  run: ./specify-run
```

No environment activation required.

---

## Editor / AI agent integration

### Claude Code

Claude Code is instructed to use `./specify-run` exclusively via:

```text
.claude/instructions.md
```

Agents must **never infer tooling**.

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

