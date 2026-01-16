# Data Model: User Consent Prompting

**Feature**: 002-prompting
**Date**: 2026-01-16

## Overview

This feature adds no persistent data structures. All state is transient within
a single script invocation.

## Environment Variables (Input)

| Variable | Type | Description |
|----------|------|-------------|
| `SPECIFYRUN_BY_AGENT` | Flag (0/1) | Enables agentic mode when set to `1` |
| `SPECIFYRUN_ANSWERS` | String | Comma-separated `key=value` pairs for pre-supplied answers |

## Prompt Keys (Constants)

| Key | Trigger Condition |
|-----|-------------------|
| `bootstrap` | `.venv/` does not exist |
| `upgrade` | `.venv/` exists but stamp file mismatches `SPECKIT_REF` |

## Exit Codes (Output)

| Code | Constant | Meaning |
|------|----------|---------|
| 0 | EX_OK | Success, action completed |
| 1 | - | General/unexpected error |
| 75 | EX_TEMPFAIL | No answer provided (agent should retry with answer) |
| 78 | EX_CONFIG | Prompt required but not authorized to proceed |
| 130 | SIGINT+128 | User explicitly declined |

## State Transitions

```
┌─────────────┐
│ Script Start│
└──────┬──────┘
       │
       ▼
┌──────────────────┐
│ Check .venv/     │
│ exists?          │
└───────┬──────────┘
        │
   ┌────┴────┐
   │         │
   ▼         ▼
┌──────┐  ┌────────────┐
│ Yes  │  │ No         │
└──┬───┘  └─────┬──────┘
   │            │
   ▼            ▼
┌──────────┐  ┌─────────────────┐
│ Check    │  │ Prompt:         │
│ stamp    │  │ key="bootstrap" │
│ matches? │  └────────┬────────┘
└────┬─────┘           │
     │                 ▼
┌────┴────┐      ┌───────────┐
│         │      │ Confirmed?│
▼         ▼      └─────┬─────┘
┌────┐ ┌──────┐        │
│Yes │ │ No   │   ┌────┴────┐
└─┬──┘ └──┬───┘   ▼         ▼
  │       │    ┌────┐    ┌──────┐
  │       ▼    │ Yes│    │ No   │
  │  ┌────────────┐└──┬──┘└──┬───┘
  │  │ Prompt:    │   │      │
  │  │ key=       │   ▼      ▼
  │  │ "upgrade"  │ Create  Exit
  │  └─────┬──────┘ .venv/  (130/75/78)
  │        │
  │        ▼
  │   ┌───────────┐
  │   │ Confirmed?│
  │   └─────┬─────┘
  │    ┌────┴────┐
  │    ▼         ▼
  │ ┌────┐    ┌──────┐
  │ │ Yes│    │ No   │
  │ └──┬─┘    └──┬───┘
  │    │         │
  │    ▼         ▼
  │  Upgrade   Exit
  │  SpecKit   (130/75/78)
  │    │
  ▼    ▼
┌─────────────┐
│ Exec SpecKit│
└─────────────┘
```
