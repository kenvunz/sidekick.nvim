# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`sidekick.nvim` is a Neovim plugin with two main subsystems:

1. **NES (Next Edit Suggestions)** — integrates Copilot LSP's ghost-text edit suggestions with diff rendering, hunk navigation, and Treesitter-based syntax highlighting.
2. **CLI Terminal** — an embedded AI CLI terminal (Claude, Copilot, Gemini, Codex, etc.) with context-aware prompts, session persistence (tmux/zellij), and picker integrations (snacks.nvim, telescope, fzf-lua).

## Commands

```bash
./scripts/test          # Run mini.test suite (auto-installs test deps via Lazy.nvim)
LAZY_OFFLINE=1 ./scripts/test  # Run tests offline (skips bootstrap download)
./scripts/docs          # Regenerate README.md sections from Lua annotations
stylua lua tests        # Format Lua source and tests
selene                  # Lint Lua files
```

To run a single spec file:

```bash
./scripts/test tests/diff_spec.lua
```

To inspect Neovim help from the CLI:

```bash
nvim --headless '+lua vim.cmd.help("nvim_buf_set_extmark"); print(table.concat(vim.api.nvim_buf_get_lines(0, vim.fn.line(".") - 1, vim.fn.line(".") + 50, false), "\n"))' +qa
```

## Architecture

```
lua/sidekick/
├── init.lua          # Public API: setup(), clear(), nes_jump_or_apply()
├── config.lua        # All defaults and Config class; docs are generated from this file
├── commands.lua      # CLI command routing
├── status.lua        # Copilot LSP statusline integration
├── util.lua          # notify, debounce, ref
├── text.lua          # Line/position utilities
├── treesitter.lua    # Function/class extraction for context
├── nes/
│   ├── init.lua      # NES orchestration: enable/disable/update/apply/jump; manages autocommands and debouncing
│   ├── diff.lua      # Diff rendering with inline word-level diffs and Treesitter highlighting
│   ├── edit.lua      # Edit application logic
│   └── ui.lua        # Visual rendering of NES suggestions
└── cli/
    ├── init.lua      # Public CLI API: show/toggle/hide/select/prompt/send
    ├── state.lua     # Terminal session state machine
    ├── terminal.lua  # Terminal window management
    ├── tool.lua      # Per-tool configuration
    ├── procs.lua     # Process detection
    ├── context/      # Context variable providers (file, selection, diagnostics, quickfix, textobject, etc.)
    ├── picker/       # snacks.nvim / telescope / fzf-lua picker adapters
    ├── session/      # tmux and zellij session persistence
    └── ui/           # Prompt and tool selection UI
```

## Testing

Tests use `mini.test` in `tests/`. The harness (`tests/minit.lua`) bootstraps Lazy.nvim and installs test dependencies (snacks.nvim, nvim-treesitter-textobjects, parsers for Python/Rust/JS/TS/Go/Lua).

- Prefer table-driven specs for combinatorial cases — see `tests/diff_spec.lua` and `tests/nes_spec.lua`.
- Use `assert.are.same`, `assert.is_true`, etc. (MiniTest assertions).
- Stub `Config.get_client` or `vim.lsp` methods directly; do **not** rely on `vim.lsp._set_clients`.
- Restore stubs in `after_each` hooks. For upvalue-based helpers, use `debug.setupvalue`.

## Key Conventions

- **NES enable guard**: Always respect the `Config.nes.enabled` callback; users can disable NES globally via `vim.g.sidekick_nes = false` or per-buffer via `vim.b[buf].sidekick_nes = false`.
- **Config documentation**: New config options must be documented in `lua/sidekick/config.lua` — docs are generated from annotations there via `./scripts/docs`. Do not edit generated sections of `README.md` directly.
- **Diff changes**: When modifying `lua/sidekick/nes/diff.lua`, update or add table-driven tests in `tests/diff_spec.lua`.
- **Status changes**: Extend `tests/status_spec.lua` when modifying status reporting.
- **ASCII preference**: Use ASCII unless surrounding context already uses Unicode (sign glyphs in user-facing configs are acceptable).
- **CI environment**: Tests may run headless with no network; avoid third-party fetches in specs — use stubs or fixtures.
- **Code style**: 2-space indents, 120-column width, sorted requires (enforced by `stylua.toml` / `selene.toml`).
