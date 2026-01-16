# Research: User Consent Prompting

**Feature**: 002-prompting
**Date**: 2026-01-16

## Terminal Detection

**Decision**: Use `[ -t 0 ]` to detect interactive terminal (stdin is TTY).

**Rationale**: POSIX-standard method, works across bash/zsh/sh. Returns true
only when stdin is connected to a terminal device.

**Alternatives considered**:
- `$TERM` check: Unreliable, may be set in non-interactive contexts
- `$PS1` check: Only works in interactive shells, not subshells
- `tty` command: Extra process spawn, same result as `[ -t 0 ]`

## User Input Reading

**Decision**: Use `read -r -p "prompt" var` with timeout for interactive mode.

**Rationale**: `-r` prevents backslash interpretation, `-p` displays prompt
inline. Bash-specific but acceptable per Technical Context.

**Alternatives considered**:
- `echo` + `read`: Works but less elegant, two statements
- Here-doc prompts: Overkill for simple y/n confirmation

## Exit Codes (BSD sysexits)

**Decision**: Use BSD sysexits.h conventions for semantic exit codes.

| Code | Name | Usage |
|------|------|-------|
| 0 | EX_OK | Success |
| 1 | General error | Script bugs, unexpected failures |
| 75 | EX_TEMPFAIL | No answer provided (timeout, agent needs retry) |
| 78 | EX_CONFIG | Non-interactive/agentic hit prompt without auth |
| 130 | SIGINT+128 | User declined (Ctrl+C or explicit "no") |

**Rationale**: These codes are well-documented, used by sendmail/postfix/etc.,
and provide semantic meaning that automation can interpret.

**Alternatives considered**:
- Custom codes (10, 11, 12): No standard meaning, harder to remember
- All errors as 1: Loses semantic information for automation

## Environment Variable Parsing

**Decision**: Parse `SPECIFYRUN_ANSWERS` as comma-separated `key=value` pairs.

**Example**: `SPECIFYRUN_ANSWERS="bootstrap=y,upgrade=n"`

**Rationale**: Simple format, no special characters needed, easy to construct
programmatically.

**Parsing approach**:
```bash
IFS=',' read -ra pairs <<< "$SPECIFYRUN_ANSWERS"
for pair in "${pairs[@]}"; do
  key="${pair%%=*}"
  value="${pair#*=}"
  # store in associative array or check inline
done
```

**Alternatives considered**:
- JSON: Requires jq or complex parsing, violates no-dependency constraint
- Newline-separated: Harder to set in single env var
- URL-encoded: Unnecessary complexity for simple keys

## Prompt Keys

**Decision**: Use short, predictable keys matching action names.

| Key | Trigger |
|-----|---------|
| `bootstrap` | First-run `.venv/` creation |
| `upgrade` | SpecKit version change |

**Rationale**: Keys match the action being authorized. Agents can hardcode
these keys in their instructions.

## Prompt Message Format

**Decision**: Use consistent format with action, explanation, and hint.

**Interactive mode**:
```
[specify-run] About to create .venv/ and install SpecKit v0.0.90
Proceed? [Y/n]:
```

**Agentic mode** (no matching answer):
```
[specify-run] About to create .venv/ and install SpecKit v0.0.90
Append `bootstrap=y` to SPECIFYRUN_ANSWERS environment variable to proceed.
```

**Rationale**: Clear prefix identifies source, action is explicit, hint is
actionable for agents.

## Flag Parsing

**Decision**: Check for `--yes` or `-y` before other arguments.

**Approach**: Early argument scan, shift if found, continue with remaining args.

```bash
YES_FLAG=0
for arg in "$@"; do
  case "$arg" in
    --yes|-y) YES_FLAG=1; shift ;;
  esac
done
```

**Rationale**: Simple, no external getopt needed, maintains single-file goal.

**Alternatives considered**:
- `getopt`/`getopts`: More robust but adds complexity for single flag
- Environment variable only: Less discoverable than CLI flag
