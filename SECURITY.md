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

---

## Supply-Chain Protection

### How specify-run reduces supply-chain risk

Traditional global CLI installation:

```text
pip install speckit        # Pulls from PyPI
                           # Version may change silently
                           # No visibility into what was installed
```

With specify-run:

```text
SPECKIT_REF="v0.3.2"       # Explicit version in committed script
                           # Installed from Git, not package registry
                           # Change requires pull request review
```

### Hardening options

For higher security environments:

| Option | How | Trade-off |
|--------|-----|-----------|
| Pin to SHA | `SPECKIT_REF="abc123..."` | Immune to tag mutation; harder to read |
| Vendor dependencies | Copy SpecKit source into repo | Full control; maintenance burden |
| Private mirror | Host internal Git mirror | Air-gapped; infrastructure overhead |

---

## Attack Examples Prevented

### Example 1: PATH Hijacking

**Scenario**: Attacker places malicious `speckit` binary in a directory that
appears earlier in `$PATH`.

**With global install**: User runs `speckit` and executes attacker's binary.

**With specify-run**: User runs `./specify-run` which uses explicit path to
virtualenv Python. PATH is never consulted for tool resolution.

### Example 2: Dependency Confusion

**Scenario**: Attacker publishes malicious package with similar name to PyPI.

**With global install**: `pip install speckit` might pull wrong package or
malicious dependency.

**With specify-run**: Installation uses Git URL directly. No PyPI resolution
occurs.

### Example 3: Silent Version Bump

**Scenario**: Upstream maintainer pushes breaking change or compromised version.

**With global install**: Next `pip install --upgrade` silently pulls new version.

**With specify-run**: Version change requires editing `SPECKIT_REF` and passing
code review. Team is aware of every version change.

---

## Version Pinning Security Benefits

The `SPECKIT_REF` variable in the script provides:

| Benefit | Description |
|---------|-------------|
| Immutability | Once committed, the version cannot change without a new commit |
| Traceability | `git blame` shows who changed the version and when |
| Rollback | Revert to previous version with `git revert` |
| Review gate | Version changes go through normal PR review process |
| Incident response | If vulnerability found, `git log` shows which commits are affected |

### Recommended practices

1. **Use tags for readability**: `SPECKIT_REF="v0.3.2"`
2. **Pin to SHA for critical systems**: `SPECKIT_REF="a1b2c3d4..."`
3. **Document version changes**: Include reason in commit message
4. **Subscribe to upstream releases**: Monitor for security advisories
