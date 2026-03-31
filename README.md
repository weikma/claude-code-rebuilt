<div align="center">

# Claude Code Rebuilt

**A rebuilt, fully functional version of Anthropic's Claude Code CLI**

[![TypeScript](https://img.shields.io/badge/TypeScript-512K%2B_lines-3178C6?logo=typescript&logoColor=white)](#tech-stack)
[![Bun](https://img.shields.io/badge/Runtime-Bun-f472b6?logo=bun&logoColor=white)](#tech-stack)
[![React + Ink](https://img.shields.io/badge/UI-React_%2B_Ink-61DAFB?logo=react&logoColor=black)](#tech-stack)
[![Files](https://img.shields.io/badge/~1,900_files-source_only-grey)](#project-structure)

</div>

---

## Table of Contents

- [Background](#background)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Building](#building)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Notes](#notes)
- [Disclaimer](#disclaimer)

---

## Background

On March 31, 2026, the full source code of Anthropic's Claude Code was leaked via a source map file exposed in their npm registry. [The leaked source code](https://github.com/instructkr/claw-code) contained only the `src/` directory -- no build configuration, no dependency manifests, no type definitions for core modules, and no way to compile or run it.

This project reconstructs everything that was missing: `package.json`, `tsconfig.json`, build scripts, 185+ stub/type files, compatibility shims for internal-only packages, and a `bun:bundle` feature-flag runtime. The result is a complete, buildable, and runnable Claude Code terminal application. Internal-only Anthropic features (daemon workers, voice mode, computer-use, etc.) are disabled at build time via feature flags; the core interactive REPL, tool system, and Anthropic API integration remain fully functional.

---

## Tech Stack

| Category | Technology |
|---|---|
| Language | [TypeScript](https://www.typescriptlang.org/) (strict) |
| Runtime | [Bun](https://bun.sh) |
| Terminal UI | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
| CLI Parsing | [Commander.js](https://github.com/tj/commander.js) (extra-typings) |
| Schema Validation | [Zod](https://zod.dev) |
| Protocols | [MCP SDK](https://modelcontextprotocol.io) · LSP |
| API | [Anthropic SDK](https://docs.anthropic.com) |
| Auth | OAuth 2.0 · API Key · macOS Keychain |

---

## Getting Started

### 1. Install Bun

Claude Code runs on [Bun](https://bun.sh/) (v1.1+). If you don't have it:

```bash
curl -fsSL https://bun.sh/install | bash
```

### 2. Install dependencies

```bash
bun install
```

### 3. Set your API key

You need an [Anthropic API key](https://console.anthropic.com/), or you can use OAuth login (`/login` in the REPL):

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

### 4. Start the application

```bash
# Launch the interactive REPL
bun run start
```

That's it -- you should see the Claude Code terminal UI.

---

## Usage

```bash
# Print version
bun run start -- --version

# Show all CLI flags and subcommands
bun run start -- --help

# One-shot prompt (pipe-friendly, prints response and exits)
bun run start -- --print "explain this codebase"

# Minimal startup (skips hooks, plugins, auto-memory)
bun run start -- --bare

# Pass a system prompt
bun run start -- --system-prompt "You are a Go expert"

# Use a specific model
bun run start -- --model sonnet
```

---

## Building

Produce a single-file bundle:

```bash
# Build to dist/cli.js
bun run build

# Run the built artifact
bun dist/cli.js
bun dist/cli.js --help
```

---

## Project Structure

```
.
├── src/
│   ├── entrypoints/cli.tsx   # Process entrypoint
│   ├── main.tsx              # Commander CLI setup, REPL launch
│   ├── commands.ts           # Slash-command registry
│   ├── tools.ts              # Tool registry (Bash, Edit, Read, etc.)
│   ├── Tool.ts               # Base tool type definitions
│   ├── query.ts              # LLM query engine
│   ├── ink/                  # Vendored Ink terminal renderer
│   ├── components/           # React terminal UI components
│   ├── screens/              # Full-screen UIs (REPL, Doctor, Resume)
│   ├── services/             # API client, MCP, analytics, compaction
│   ├── hooks/                # React hooks
│   ├── utils/                # Utility functions
│   ├── types/                # Reconstructed type definitions
│   └── _external/            # Build compatibility layer
│       ├── preload.ts        # Runtime MACRO + bun:bundle shim
│       ├── globals.d.ts      # MACRO type declarations
│       └── shims/            # Stub packages for private deps
├── scripts/
│   └── build-external.ts     # Bun.build() with feature flags + defines
├── package.json
├── tsconfig.json
└── bunfig.toml               # Preload config + .md text loader
```

---

## How It Works

The original Claude Code source depends on Bun's `bun:bundle` module for compile-time feature flags and `MACRO.*` globals for build-time constants. This project provides:

1. **`bunfig.toml` + `preload.ts`** -- registers a Bun plugin that resolves `import { feature } from 'bun:bundle'` at runtime, and defines `MACRO.VERSION` and friends as globals.
2. **`scripts/build-external.ts`** -- a `Bun.build()` script that replaces `bun:bundle` via a plugin, injects `MACRO.*` via `define`, and marks private packages as external. All 90+ internal feature flags are disabled; only a handful of safe flags are enabled.
3. **Stub packages under `src/_external/shims/`** -- lightweight no-op modules for `@ant/*` internal packages and native NAPI addons that aren't publicly available.
4. **Reconstructed type files** -- `src/types/message.ts`, `src/types/tools.ts`, and other high-fanout modules that were missing from the leaked source.

---

## Notes

- Features gated behind disabled flags (voice, bridge, daemon, coordinator, assistant/Kairos, etc.) are not functional.
- The interactive REPL, `--print` mode, `--help`, and the full Commander option surface all work.
- Authentication (API key and OAuth), Anthropic API calls, tool execution, MCP server integration, and the Ink-based terminal UI are preserved from the original source.

---

## Disclaimer

**All original Claude Code source code is the intellectual property of [Anthropic, PBC](https://www.anthropic.com/).** This repository is based on source code that was unintentionally exposed and is provided here **strictly for research, educational, and archival purposes only**.

- This project carries **no license**. No permission is granted to use, modify, distribute, or create derivative works for any commercial purpose.
- This is an independent reconstruction effort and is **not affiliated with, endorsed by, or sponsored by Anthropic** in any way.
- If you are a representative of Anthropic and would like this repository removed, please open an issue or contact the maintainer ([@weikma](https://github.com/weikma)) directly.
