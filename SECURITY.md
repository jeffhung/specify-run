# Security Model

This document describes the security properties of `specify-run` and the
threats it mitigates.

For installation and usage, see [README.md](README.md).
For CI and agent integration, see [INTEGRATION.md](INTEGRATION.md).

---

## Overview

The `specify-run` script provides a security-hardened approach to running
SpecKit by eliminating common attack vectors associated with global CLI tools
and hidden dependency resolution.

---

## Threat Model

### Threats Addressed

| Threat | Attack Vector | Mitigation |
|--------|---------------|------------|
| Version drift | Different developers run different CLI versions | Pinned `SPECKIT_REF` in script |
| PATH hijacking | Malicious binary shadows legitimate tool | No PATH dependency; explicit path execution |
| Silent upgrades | Upstream changes behavior without notice | Upgrades require explicit commit |
| Supply-chain injection | Compromised package registry | Git-based installation from known source |
| Environment pollution | Global packages conflict or leak | Project-local virtualenv isolation |

### Threats NOT Addressed

This design does not protect against:

* Compromised Git repository (mitigate: pin to SHA, vendor dependencies)
* Malicious code in SpecKit itself (mitigate: audit before adoption)
* Local machine compromise (out of scope)

---

## Hardening Properties

The `specify-run` design provides the following security properties:

| Property | Description |
|----------|-------------|
| No global state | Script uses project-local `.venv/`; no system-wide side effects |
| Explicit upgrades | Version changes require editing `SPECKIT_REF` and committing |
| Auditability | All tooling changes visible in git history |
| Deterministic behavior | Same script + same ref = same behavior across machines |
| No hidden resolution | No PATH lookup, no implicit package manager calls |

### Verification

Security reviewers can verify these properties by:

1. Reading the script source (single file, <200 lines)
2. Checking `SPECKIT_REF` value against known-good versions
3. Reviewing git history for script changes
