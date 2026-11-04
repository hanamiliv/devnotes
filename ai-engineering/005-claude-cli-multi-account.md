---
type: howto
tags: [ai-engineering, claude, cli, terminal]
created: 2026-11-04
updated: 2026-11-04
---

# Run Multiple Claude CLI Accounts Concurrently

> **TL;DR** — Use the `CLAUDE_CONFIG_DIR` environment variable to isolate config directories per account (e.g. `CLAUDE_CONFIG_DIR=~/.claude-work claude`). This keeps credentials, sessions, and histories completely separate without overwriting your default login.

## Before You Start

- `claude` CLI tool installed.

## Steps

### 1. Define custom config directories

Assign separate folders for each account to store credentials and state independently instead of defaulting to `~/.claude`.

```bash
mkdir -p ~/.claude-work ~/.claude-personal
```

### 2. Initialize and sign in each profile

Run the CLI while setting `CLAUDE_CONFIG_DIR` for each environment to complete sign-in.

```bash
# Work account setup
CLAUDE_CONFIG_DIR=~/.claude-work claude

# Personal account setup (in another terminal tab)
CLAUDE_CONFIG_DIR=~/.claude-personal claude
```

### 3. Create shell aliases for quick switching

Add aliases to your shell configuration file (`~/.zshrc` or `~/.bashrc`) so you can launch specific profiles without typing full environment variables.

```bash
# Add to ~/.zshrc or ~/.bashrc
alias claude-work='CLAUDE_CONFIG_DIR=~/.claude-work claude'
alias claude-personal='CLAUDE_CONFIG_DIR=~/.claude-personal claude'
```

Apply the changes:

```bash
source ~/.zshrc
```

## Verify

Check that each alias resolves to its distinct configuration directory:

```bash
type claude-work
```

Expected output:

```text
claude-work is an alias for CLAUDE_CONFIG_DIR=~/.claude-work claude
```

Verify that settings files are created in separate directories:

```bash
ls -la ~/.claude-work/settings.json ~/.claude-personal/settings.json
```

## Gotchas

- Running `claude` without `CLAUDE_CONFIG_DIR` will still use the default `~/.claude` directory and will not affect any custom profile directories.
- Ensure custom config directory paths exist or are writable by the user, otherwise the CLI falls back or throws permission errors.

## Quick Reference

| Task | Command |
|---|---|
| Run Work Profile | `claude-work` |
| Run Personal Profile | `claude-personal` |
| One-off custom dir execution | `CLAUDE_CONFIG_DIR=~/.claude-custom claude` |

## Sources

- None (from experience)