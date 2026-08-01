# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Build (debug is default)
make
# Build with optimizations
make CMAKE_BUILD_TYPE=RelWithDebInfo
# Run without installing
VIMRUNTIME=./runtime ./build/bin/nvim
# Use Lua modules at runtime instead of precompiled bytecode
VIMRUNTIME=./runtime ./build/bin/nvim --luamod-dev
# Clean build
make distclean && make
# Build with ASAN/UBSAN
CMAKE_EXTRA_FLAGS="-DCMAKE_C_COMPILER=clang -DENABLE_ASAN_UBSAN=1" make
```

## Test System

Three test types, run via `make`:

```bash
# All tests
make test

# Functional tests (integration tests driven by RPC, spawns real nvim processes)
make functionaltest
# Single functional test
make functionaltest TEST_FILE=test/functional/example_spec.lua
# Filter within a test file
make functionaltest TEST_FILE=test/functional/api/buffer_spec.lua TEST_FILTER='nvim_buf_set_text'
# Debug with GDB
GDB=1 TEST_FILE=test/functional/api/buffer_spec.lua make functionaltest

# Unit tests (LuaJIT FFI calling compiled C)
make unittest
make unittest TEST_FILE=test/unit/foo_spec.lua

# Old tests (legacy Vim tests)
make oldtest
make oldtest TEST_FILE=test_syntax.vim

# Valgrind
VALGRIND=1 make test

# Parallel (CI uses this)
cmake --build build --target functionaltest-parallel -j2
```

### Zig builds

```bash
zig build functionaltest -- test/functional/autocmd/bufenter_spec.lua
zig build unittest
zig build oldtest
```

### Available env vars (see `runtime/doc/dev_test.txt` for full list)
- `TEST_FILE` — path to a specific test file
- `TEST_FILTER` — Lua pattern to filter tests within a file
- `TEST_FILTER_OUT` — exclude tests matching pattern
- `TEST_TAG` — run tests with specific tag
- `NVIM_TEST_TRACE_ON_ERROR=1` — trace on unit test error
- `TEST_SKIP_FRAGILE=1` — skip fragile tests
- `TEST_ARGS="--repeat=100 --no-keep-going"` — extra busted args

## Lint & Format

```bash
make lint       # All linters (C, Lua, sh, commit, doc, queries, etc.)
make lintc      # C only
make lintlua    # Lua only
make lintdoc    # Documentation validation
make lintcommit # Commit message format
make format     # Format all (C via uncrustify, Lua, queries)
```

## Code Generation & Docs

```bash
make doc        # Regenerate :help docs from C/Lua docstrings
```

Doc generation pipeline lives in `src/gen/`:
- `gen_vimdoc.lua` — main doc generator (parses C/Lua → vimdoc)
- `cdoc_parser.lua` / `luacats_parser.lua` — docstring parsers
- `gen_eval_files.lua` — generates `lua.txt`, `api.txt`, `vimfn.txt`, `options.txt`

## Include Management

```bash
cmake --preset iwyu
cmake --build build
make iwyu  # auto-fix includes
```

## Project Architecture

**Source layout:**
- `src/nvim/` — Core C code: editor, buffers, windows, screen rendering, etc.
- `src/nvim/api/` — Public API functions (vim, buffer, window, tabpage, ui, command, autocmd, extmark)
- `src/nvim/eval/` — Vimscript expression evaluator and type system
- `src/nvim/viml/parser/` — VimL parser
- `src/nvim/event/` — Event loop (libuv-based)
- `src/nvim/tui/` — Terminal UI (tui.c + terminfo/termkey)
- `src/nvim/os/` — OS abstractions (file I/O, processes, signals, shell)
- `src/nvim/lua/` — Lua/C bindings (treesitter, executor, converter, stdlib)
- `src/nvim/msgpack_rpc/` — MessagePack RPC channel server/client
- `src/gen/` — Code generation scripts (docs, API dispatch, UI events, options, keycodes, terminfo)
- `runtime/lua/vim/` — Lua standard library
- `runtime/lua/vim/_core/` — Core Lua modules (precompiled to bytecode; use `--luamod-dev` to bypass)
- `runtime/lua/vim/lsp/` — LSP client (buf.lua, client.lua, completion.lua, handlers.lua, etc.)
- `runtime/lua/vim/treesitter/` — Tree-sitter integration (languagetree.lua, highlighter.lua, query.lua)
- `runtime/lua/vim/diagnostic/` — Diagnostic framework
- `runtime/plugin/` — Built-in opt-out plugins
- `runtime/doc/` — Help files (many auto-generated from docstrings)
- `scripts/` — Utility scripts (release, vim-patch, lint)
- `cmake.deps/` — Third-party dependency build system
- `test/functional/` — Integration tests
- `test/unit/` — Unit tests (LuaJIT FFI)
- `test/old/testdir/` — Legacy Vim tests

**Key architectural patterns:**

- **New functionality should go in Lua, not C.** See PRs #37757, #37831 for examples.
- **UI is compositor-based.** External UIs communicate via msgpack RPC; the TUI is the built-in default. UI events flow through `ui.c` → compositor → external channels or TUI.
- **Event loop** built on libuv (`src/nvim/event/loop.c`), drives async I/O, timers, job control, RPC.
- **Tree-sitter** provides syntax parsing (used for highlighting, indentation, folding, motions).
- **LSP client** is implemented entirely in Lua under `runtime/lua/vim/lsp/`.
- **Vimscript eval** (`src/nvim/eval/`) implements the Vim expression evaluator, type system (`typval`), and built-in functions.

**Important development notes:**
- `git config blame.ignoreRevsFile .git-blame-ignore-revs` — cleaner git blame
- Use **clangd** for C code navigation (config at nvim-lspconfig/clangd)
- Commit messages follow **conventional commits**: `type(scope): subject` with `Problem:`/`Solution:` body
- Vim runtime files (Vimscript) are maintained upstream by Vim; Lua runtime files by Neovim
- `$NVIM_LOG_FILE` for debug logging
- Vim patches follow a special workflow — see `runtime/doc/dev_vimpatch.txt`
