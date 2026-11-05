<!-- path: ai-engineering/000-fix-claude-code-npm-nvm-install.md -->
---
type: howto
tags: [ai-engineering, claude-code, npm, nvm, macos]
created: 2026-11-05
updated: 2026-11-05
---

# Fix Claude Code ENOTEMPTY Errors by Switching to the Native Installer

> **TL;DR** — `npm update -g @anthropic-ai/claude-code` under nvm can fail with `ENOTEMPTY` on a stuck package directory. Remove the directory by hand, then switch to Anthropic's native installer, which lives outside nvm/node_modules and avoids this class of error entirely.

## Before You Start

- macOS, Node installed via nvm.
- Claude Code currently installed as an npm global package.

## Steps

### 1. Diagnose the error

`npm update -g` (or `npm uninstall -g`) fails with something like:

```
npm error code ENOTEMPTY
npm error syscall rename
npm error path .../node_modules/@anthropic-ai/claude-code
npm error dest .../node_modules/@anthropic-ai/.claude-code-XXXXXXXX
npm error ENOTEMPTY: directory not empty, rename '...' -> '...'
```

This happens when a previous install/update was interrupted (killed terminal, sleep, network
drop), leaving a package directory in a state npm's rename-based update step can't clean up.
It is an npm/filesystem issue, not specific to Claude Code.

### 2. Remove the stuck package directory manually

npm's own `uninstall` can fail the same way, so delete it directly instead:

```bash
rm -rf "$(dirname "$(which node)")/../lib/node_modules/@anthropic-ai/claude-code"
```

Or use the exact path from the error message. Verify it's gone:

```bash
ls "$(dirname "$(which node)")/../lib/node_modules/@anthropic-ai/"
```

### 3. Install via the native installer instead of npm

The native installer puts the binary in `~/.local/bin`, independent of nvm or any Node
version, so switching Node versions can never break or collide with it again:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

### 4. Add `~/.local/bin` to PATH if prompted

The installer warns if it's missing:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

## Verify

```bash
which claude
claude --version
```

`which claude` should resolve to `~/.local/bin/claude`, not a path under `.nvm/versions/...`.

## Gotchas

- A leftover platform-specific package (e.g. `claude-code-darwin-arm64`) may remain under
  `node_modules/@anthropic-ai/` after removing `claude-code` — it's orphaned and harmless,
  safe to delete or ignore.
- If `rm -rf` itself hangs, close any shell/editor with that directory as its working
  directory, then retry; `sudo rm -rf` is a last resort (the path is usually user-owned).
- After switching installers, run `npm uninstall -g @anthropic-ai/claude-code` to avoid two
  copies of `claude` on PATH, in case npm's version is still listed as installed.

## Sources

- None (from experience)