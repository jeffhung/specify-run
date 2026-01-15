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
