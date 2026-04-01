# Claude Code — Source Snapshot (Security Research Archive)

> This repository mirrors a **publicly exposed Claude Code source snapshot**
> that became accessible on **March 31, 2026** via a source map bundled into
> Anthropic's npm distribution. It is maintained for **educational, defensive
> security research, and software supply-chain analysis** purposes only.

---

## Research Context

This archive is maintained by a **university student** researching:

- Software supply-chain exposure and build artifact leaks
- Secure software engineering practices
- Agentic developer tooling architecture
- Defensive analysis of real-world CLI systems

It is intended to support:

- Educational study
- Security research practice
- Architecture review
- Discussion of packaging and release-process failures

This repository makes no claim of ownership over the original code and
should not be interpreted as an official Anthropic repository.

---

## How the Snapshot Became Publicly Accessible

[Chaofan Shou (@Fried_rice)](https://x.com/Fried_rice) publicly reported
that Claude Code source material was reachable through a `.map` file
inadvertently included in the published npm package:

> **"Claude code source code has been leaked via a map file in their npm registry!"**
>
> — [@Fried_rice, March 31, 2026](https://x.com/Fried_rice/status/2038894956459290963)

The published source map referenced unobfuscated TypeScript sources hosted
in Anthropic's R2 storage bucket, making the full `src/` snapshot publicly
downloadable before the affected package version was pulled.

The root cause is a common npm packaging oversight: Bun's bundler generates
source maps by default unless explicitly disabled, and the `.map` file was
not excluded via `.npmignore`. The irony is that the codebase contains an
"Undercover Mode" subsystem specifically designed to prevent internal
Anthropic information from surfacing in public commits — yet the entire
source shipped in a map file.

---

## Repository Scope

Claude Code is Anthropic's official CLI for interacting with Claude from
the terminal to perform software engineering tasks: editing files, running
shell commands, searching codebases, and coordinating multi-step workflows.

This repository contains a mirrored `src/` snapshot for research and analysis.

| Property | Value |
|---|---|
| Exposure identified | 2026-03-31 |
| Language | TypeScript (strict mode) |
| Runtime | Bun |
| Terminal UI | React + [Ink](https://github.com/vadimdemedes/ink) |
| Scale | ~1,900 files · 512,000+ lines of code |

---

## Directory Structure
```text
src/
├── main.tsx                 # CLI entrypoint (Commander.js-based)
├── commands.ts              # Command registry
├── tools.ts                 # Tool registry
├── Tool.ts                  # Tool base types and interfaces
├── QueryEngine.ts           # Core LLM query engine
├── context.ts               # System and user context collection
├── cost-tracker.ts          # Token cost tracking
│
├── commands/                # Slash command implementations (~50)
├── tools/                   # Agent tool implementations (~40)
├── components/              # Ink UI components (~140)
├── hooks/                   # React hooks
├── services/                # External service integrations
├── screens/                 # Full-screen UIs (Doctor, REPL, Resume)
├── types/                   # TypeScript type definitions
├── utils/                   # Utility functions
│
├── bridge/                  # IDE and remote-control bridge
├── coordinator/             # Multi-agent coordinator
├── plugins/                 # Plugin system
├── skills/                  # Skill system
├── keybindings/             # Keybinding configuration
├── vim/                     # Vim mode
├── voice/                   # Voice input
├── remote/                  # Remote sessions
├── server/                  # Server mode
├── memdir/                  # Persistent memory directory
├── tasks/                   # Task management
├── state/                   # State management
├── migrations/              # Config migrations
├── schemas/                 # Config schemas (Zod)
├── entrypoints/             # Initialization logic
├── ink/                     # Ink renderer wrapper
├── buddy/                   # Virtual companion system (see below)
├── native-ts/               # Native TypeScript utilities
├── outputStyles/            # Output styling
├── query/                   # Query pipeline
└── upstreamproxy/           # Proxy configuration
```

---

## Architecture Overview

### 1. Tool System (`src/tools/`)

Every capability Claude Code can invoke is implemented as a self-contained,
permission-gated module with its own input schema and execution logic.

| Tool | Description |
|---|---|
| `BashTool` | Shell command execution |
| `FileReadTool` | File reading (images, PDFs, notebooks) |
| `FileWriteTool` | File creation and overwrite |
| `FileEditTool` | Partial file modification via string replacement |
| `GlobTool` | File pattern matching |
| `GrepTool` | ripgrep-based content search |
| `WebFetchTool` | URL content retrieval |
| `WebSearchTool` | Web search |
| `AgentTool` | Sub-agent spawning |
| `SkillTool` | Skill execution |
| `MCPTool` | MCP server tool invocation |
| `LSPTool` | Language Server Protocol integration |
| `NotebookEditTool` | Jupyter notebook editing |
| `TaskCreateTool` / `TaskUpdateTool` | Task lifecycle management |
| `SendMessageTool` | Inter-agent messaging |
| `TeamCreateTool` / `TeamDeleteTool` | Team agent management |
| `EnterPlanModeTool` / `ExitPlanModeTool` | Plan mode toggle |
| `EnterWorktreeTool` / `ExitWorktreeTool` | Git worktree isolation |
| `ToolSearchTool` | Deferred tool discovery |
| `CronCreateTool` | Scheduled trigger creation |
| `RemoteTriggerTool` | Remote trigger |
| `SleepTool` | Proactive mode wait |
| `SyntheticOutputTool` | Structured output generation |

### 2. Command System (`src/commands/`)

User-facing slash commands invoked with the `/` prefix.

| Command | Description |
|---|---|
| `/commit` | Create a git commit |
| `/review` | Code review |
| `/compact` | Compress conversation context |
| `/mcp` | MCP server management |
| `/config` | Settings management |
| `/doctor` | Environment diagnostics |
| `/login` / `/logout` | Authentication |
| `/memory` | Persistent memory management |
| `/skills` | Skill management |
| `/tasks` | Task management |
| `/vim` | Vim mode toggle |
| `/diff` | View pending changes |
| `/cost` | Check usage cost |
| `/theme` | Change color theme |
| `/context` | Context window visualization |
| `/pr_comments` | View pull request comments |
| `/resume` | Restore a previous session |
| `/share` | Share the current session |
| `/desktop` | Hand off to desktop app |
| `/mobile` | Hand off to mobile app |

### 3. Service Layer (`src/services/`)

| Service | Description |
|---|---|
| `api/` | Anthropic API client, file API, bootstrap |
| `mcp/` | MCP server connection and management |
| `oauth/` | OAuth 2.0 authentication flow |
| `lsp/` | Language Server Protocol manager |
| `analytics/` | GrowthBook feature flags and analytics |
| `plugins/` | Plugin loader |
| `compact/` | Conversation context compression |
| `policyLimits/` | Organization policy enforcement |
| `remoteManagedSettings/` | Remote configuration management |
| `extractMemories/` | Automatic memory extraction |
| `tokenEstimation.ts` | Token count estimation |
| `teamMemorySync/` | Team memory synchronization |

### 4. Bridge System (`src/bridge/`)

A bidirectional communication layer connecting IDE extensions (VS Code,
JetBrains) with the Claude Code CLI process.

| File | Description |
|---|---|
| `bridgeMain.ts` | Bridge main loop |
| `bridgeMessaging.ts` | Message protocol |
| `bridgePermissionCallbacks.ts` | Permission callbacks |
| `replBridge.ts` | REPL session bridge |
| `jwtUtils.ts` | JWT-based authentication |
| `sessionRunner.ts` | Session execution management |

### 5. Permission System (`src/hooks/toolPermission/`)

Every tool invocation is gated through a permission check. Depending on
the configured mode (`default`, `plan`, `bypassPermissions`, `auto`, etc.),
the system either prompts the user for approval or resolves the request
automatically.

### 6. Feature Flags

Inactive features are completely stripped at build time using Bun's
`bun:bundle` dead-code elimination:
```typescript
import { feature } from 'bun:bundle'

const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

Notable flags: `PROACTIVE`, `KAIROS`, `BRIDGE_MODE`, `DAEMON`,
`VOICE_MODE`, `AGENT_TRIGGERS`, `MONITOR_TOOL`

---

## Unreleased Features of Note

Two subsystems found in the leaked source are particularly interesting from
a research perspective, as they reveal product directions Anthropic had not
publicly announced.

### 🐾 Buddy System (`src/buddy/`)

The `buddy/` module implements a fully self-contained **virtual companion
system** — think Tamagotchi or a simplified Pokémon — living inside your
terminal alongside Claude Code.

**How it works:**

- Users can **hatch** a companion from an egg at the start of a session.
- The companion **grows and evolves** based on how the user interacts with
  Claude Code over time — longer sessions, more tool usage, and completed
  tasks all feed into its development.
- Each buddy belongs to one of **18 distinct species**, each with its own
  appearance and personality traits.
- Companions have **rarity tiers** (common through legendary) and **shiny
  variants**, echoing Pokémon-style collectibility.
- Attribute statistics track the companion's growth across dimensions such
  as energy, happiness, and experience.
- The system includes **idle animations** rendered through Ink, so the
  companion visibly reacts in the terminal UI during and between sessions.

The Buddy system is gated behind a feature flag and was not shipped in any
public release. Its presence suggests Anthropic was experimenting with
long-term engagement mechanics — turning daily developer workflows into a
lightweight, gamified relationship with the tool.

**Why it matters for research:** The Buddy system is a concrete example of
how AI tooling is beginning to incorporate persistent, stateful identity
tied to user behavior — a design pattern with interesting implications for
user attachment, session continuity, and behavioral nudging.

---

### ⏱️ KAIROS (`feature('KAIROS')`)

KAIROS is a **background autonomous agent daemon** — the most architecturally
significant unreleased feature in the leaked source.

**What it is:**

KAIROS transforms Claude Code from a reactive, prompt-driven CLI into a
**proactively running AI assistant** that operates continuously in the
background, even when the user is not actively interacting with it.

**Key characteristics:**

| Property | Detail |
|---|---|
| Activation | `feature('KAIROS')` build flag |
| Gate | GrowthBook experiment: `tengu_kairos_gate` |
| Mode | Long-running daemon process |
| Trigger model | Scheduled (`CronCreateTool`) + event-driven (`RemoteTriggerTool`) |
| Memory | Full integration with `memdir/` persistent memory |
| Reporting | Proactive progress summaries pushed to the user |

**What KAIROS can do autonomously:**

- **Explore** the codebase on a schedule — scanning for drift, tech debt,
  or areas flagged in memory from previous sessions.
- **Run health checks** — executing linting, test suites, or custom
  diagnostic scripts without being asked.
- **File tasks proactively** — creating entries via `TaskCreateTool` based
  on what it discovers during background sweeps.
- **Report findings** — surfacing summaries and alerts to the user when
  they return to the terminal, or via a notification channel.
- **Maintain persistent awareness** — because KAIROS reads and writes to
  `memdir/`, it builds up a long-running model of the project's state
  across days and sessions.

**Architecture sketch:**
```
User away
    │
    ▼
KAIROS daemon (background)
    ├── CronCreateTool  ──► scheduled sweeps
    ├── RemoteTriggerTool ► event-driven wakeups
    ├── AgentTool        ── spawns sub-agents for parallel work
    ├── memdir/          ── reads/writes persistent memory
    └── SleepTool        ── rate-limits proactive polling
    │
    ▼
User returns
    └── Proactive summary report waiting in terminal
```

**Why it matters for research:** KAIROS represents a meaningful shift in
the human–AI collaboration model. Most current AI coding tools are strictly
reactive: the user prompts, the tool responds, the session ends. KAIROS
inverts this — the AI maintains ongoing awareness of the project and
surfaces information unprompted. This raises open questions around user
control, resource consumption, auditability of autonomous actions, and the
security surface introduced by a persistent, permissioned agent process
running outside of active user supervision.

---

## Key Files

| File | Size | Description |
|---|---|---|
| `QueryEngine.ts` | ~46K lines | Core LLM engine: streaming, tool loops, thinking mode, retries, token counting |
| `Tool.ts` | ~29K lines | Base types and interfaces for all tools |
| `commands.ts` | ~25K lines | Slash command registration and dispatch |
| `main.tsx` | — | CLI parser and React/Ink renderer initialization |

---

## Tech Stack

| Category | Technology |
|---|---|
| Runtime | [Bun](https://bun.sh) |
| Language | TypeScript (strict) |
| Terminal UI | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
| CLI Parsing | [Commander.js](https://github.com/tj/commander.js) |
| Schema Validation | [Zod v4](https://zod.dev) |
| Code Search | [ripgrep](https://github.com/BurntSushi/ripgrep) |
| Protocols | [MCP SDK](https://modelcontextprotocol.io), LSP |
| API Client | [Anthropic SDK](https://docs.anthropic.com) |
| Telemetry | OpenTelemetry + gRPC |
| Feature Flags | GrowthBook |
| Auth | OAuth 2.0, JWT, macOS Keychain |

---

## Notable Design Patterns

### Parallel Prefetch at Startup

MDM settings, keychain reads, and API preconnection are initiated in
parallel as early side-effects — before heavy module evaluation — to
minimize perceived startup latency.
```typescript
// main.tsx — fired before other imports
startMdmRawRead()
startKeychainPrefetch()
```

### Lazy Loading

Heavy modules (OpenTelemetry, gRPC, analytics, and feature-gated
subsystems) are deferred via dynamic `import()` until actually needed.

### Multi-Agent Orchestration

Sub-agents are spawned via `AgentTool`; `coordinator/` manages their
lifecycles and communication. `TeamCreateTool` enables parallel work
across agent teams.

### Skill System

Reusable, composable workflows are defined in `skills/` and executed
through `SkillTool`. Users can add custom skills alongside the built-ins.

### Plugin Architecture

Both built-in and third-party plugins are loaded through the `plugins/`
subsystem, enabling extensibility without modifying core code.

---

## Legal and Ownership Disclaimer

- This repository is an **educational and defensive security research
  archive** maintained by a university student.
- It exists to study source exposure, build artifact leaks, and the
  architecture of modern agentic CLI systems.
- The original Claude Code source remains the intellectual property of
  **Anthropic, PBC**.
- This repository is **not affiliated with, endorsed by, or maintained
  by Anthropic**.
- If Anthropic issues a takedown request, this repository will be
  removed promptly.

---

## How the Snapshot Became Publicly Accessible

[Chaofan Shou (@Fried_rice)](https://x.com/Fried_rice) publicly reported
that Claude Code source material was reachable through a `.map` file
inadvertently included in the published npm package:

> **"Claude code source code has been leaked via a map file in their npm registry!"**
>
> — [@Fried_rice, March 31, 2026](https://x.com/Fried_rice/status/2038894956459290963)

The published source map referenced unobfuscated TypeScript sources hosted
in Anthropic's R2 storage bucket, making the full `src/` snapshot publicly
downloadable before the affected package version was pulled.

The root cause is a common npm packaging oversight: Bun's bundler generates
source maps by default unless explicitly disabled, and the `.map` file was
not excluded via `.npmignore`. The irony is that the codebase contains an
"Undercover Mode" subsystem specifically designed to prevent internal
Anthropic information from surfacing in public commits — yet the entire
source shipped in a map file.

---

## Repository Scope

Claude Code is Anthropic's official CLI for interacting with Claude from
the terminal to perform software engineering tasks: editing files, running
shell commands, searching codebases, and coordinating multi-step workflows.

This repository contains a mirrored `src/` snapshot for research and analysis.

| Property | Value |
|---|---|
| Exposure identified | 2026-03-31 |
| Language | TypeScript (strict mode) |
| Runtime | Bun |
| Terminal UI | React + [Ink](https://github.com/vadimdemedes/ink) |
| Scale | ~1,900 files · 512,000+ lines of code |

---

## Directory Structure
```text
src/
├── main.tsx                 # CLI entrypoint (Commander.js-based)
├── commands.ts              # Command registry
├── tools.ts                 # Tool registry
├── Tool.ts                  # Tool base types and interfaces
├── QueryEngine.ts           # Core LLM query engine
├── context.ts               # System and user context collection
├── cost-tracker.ts          # Token cost tracking
│
├── commands/                # Slash command implementations (~50)
├── tools/                   # Agent tool implementations (~40)
├── components/              # Ink UI components (~140)
├── hooks/                   # React hooks
├── services/                # External service integrations
├── screens/                 # Full-screen UIs (Doctor, REPL, Resume)
├── types/                   # TypeScript type definitions
├── utils/                   # Utility functions
│
├── bridge/                  # IDE and remote-control bridge
├── coordinator/             # Multi-agent coordinator
├── plugins/                 # Plugin system
├── skills/                  # Skill system
├── keybindings/             # Keybinding configuration
├── vim/                     # Vim mode
├── voice/                   # Voice input
├── remote/                  # Remote sessions
├── server/                  # Server mode
├── memdir/                  # Persistent memory directory
├── tasks/                   # Task management
├── state/                   # State management
├── migrations/              # Config migrations
├── schemas/                 # Config schemas (Zod)
├── entrypoints/             # Initialization logic
├── ink/                     # Ink renderer wrapper
├── buddy/                   # Virtual companion system (see below)
├── native-ts/               # Native TypeScript utilities
├── outputStyles/            # Output styling
├── query/                   # Query pipeline
└── upstreamproxy/           # Proxy configuration
```

---

## Architecture Overview

### 1. Tool System (`src/tools/`)

Every capability Claude Code can invoke is implemented as a self-contained,
permission-gated module with its own input schema and execution logic.

| Tool | Description |
|---|---|
| `BashTool` | Shell command execution |
| `FileReadTool` | File reading (images, PDFs, notebooks) |
| `FileWriteTool` | File creation and overwrite |
| `FileEditTool` | Partial file modification via string replacement |
| `GlobTool` | File pattern matching |
| `GrepTool` | ripgrep-based content search |
| `WebFetchTool` | URL content retrieval |
| `WebSearchTool` | Web search |
| `AgentTool` | Sub-agent spawning |
| `SkillTool` | Skill execution |
| `MCPTool` | MCP server tool invocation |
| `LSPTool` | Language Server Protocol integration |
| `NotebookEditTool` | Jupyter notebook editing |
| `TaskCreateTool` / `TaskUpdateTool` | Task lifecycle management |
| `SendMessageTool` | Inter-agent messaging |
| `TeamCreateTool` / `TeamDeleteTool` | Team agent management |
| `EnterPlanModeTool` / `ExitPlanModeTool` | Plan mode toggle |
| `EnterWorktreeTool` / `ExitWorktreeTool` | Git worktree isolation |
| `ToolSearchTool` | Deferred tool discovery |
| `CronCreateTool` | Scheduled trigger creation |
| `RemoteTriggerTool` | Remote trigger |
| `SleepTool` | Proactive mode wait |
| `SyntheticOutputTool` | Structured output generation |

### 2. Command System (`src/commands/`)

User-facing slash commands invoked with the `/` prefix.

| Command | Description |
|---|---|
| `/commit` | Create a git commit |
| `/review` | Code review |
| `/compact` | Compress conversation context |
| `/mcp` | MCP server management |
| `/config` | Settings management |
| `/doctor` | Environment diagnostics |
| `/login` / `/logout` | Authentication |
| `/memory` | Persistent memory management |
| `/skills` | Skill management |
| `/tasks` | Task management |
| `/vim` | Vim mode toggle |
| `/diff` | View pending changes |
| `/cost` | Check usage cost |
| `/theme` | Change color theme |
| `/context` | Context window visualization |
| `/pr_comments` | View pull request comments |
| `/resume` | Restore a previous session |
| `/share` | Share the current session |
| `/desktop` | Hand off to desktop app |
| `/mobile` | Hand off to mobile app |

### 3. Service Layer (`src/services/`)

| Service | Description |
|---|---|
| `api/` | Anthropic API client, file API, bootstrap |
| `mcp/` | MCP server connection and management |
| `oauth/` | OAuth 2.0 authentication flow |
| `lsp/` | Language Server Protocol manager |
| `analytics/` | GrowthBook feature flags and analytics |
| `plugins/` | Plugin loader |
| `compact/` | Conversation context compression |
| `policyLimits/` | Organization policy enforcement |
| `remoteManagedSettings/` | Remote configuration management |
| `extractMemories/` | Automatic memory extraction |
| `tokenEstimation.ts` | Token count estimation |
| `teamMemorySync/` | Team memory synchronization |

### 4. Bridge System (`src/bridge/`)

A bidirectional communication layer connecting IDE extensions (VS Code,
JetBrains) with the Claude Code CLI process.

| File | Description |
|---|---|
| `bridgeMain.ts` | Bridge main loop |
| `bridgeMessaging.ts` | Message protocol |
| `bridgePermissionCallbacks.ts` | Permission callbacks |
| `replBridge.ts` | REPL session bridge |
| `jwtUtils.ts` | JWT-based authentication |
| `sessionRunner.ts` | Session execution management |

### 5. Permission System (`src/hooks/toolPermission/`)

Every tool invocation is gated through a permission check. Depending on
the configured mode (`default`, `plan`, `bypassPermissions`, `auto`, etc.),
the system either prompts the user for approval or resolves the request
automatically.

### 6. Feature Flags

Inactive features are completely stripped at build time using Bun's
`bun:bundle` dead-code elimination:
```typescript
import { feature } from 'bun:bundle'

const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

Notable flags: `PROACTIVE`, `KAIROS`, `BRIDGE_MODE`, `DAEMON`,
`VOICE_MODE`, `AGENT_TRIGGERS`, `MONITOR_TOOL`

---

## Unreleased Features of Note

Two subsystems found in the leaked source are particularly interesting from
a research perspective, as they reveal product directions Anthropic had not
publicly announced.

### 🐾 Buddy System (`src/buddy/`)

The `buddy/` module implements a fully self-contained **virtual companion
system** — think Tamagotchi or a simplified Pokémon — living inside your
terminal alongside Claude Code.

**How it works:**

- Users can **hatch** a companion from an egg at the start of a session.
- The companion **grows and evolves** based on how the user interacts with
  Claude Code over time — longer sessions, more tool usage, and completed
  tasks all feed into its development.
- Each buddy belongs to one of **18 distinct species**, each with its own
  appearance and personality traits.
- Companions have **rarity tiers** (common through legendary) and **shiny
  variants**, echoing Pokémon-style collectibility.
- Attribute statistics track the companion's growth across dimensions such
  as energy, happiness, and experience.
- The system includes **idle animations** rendered through Ink, so the
  companion visibly reacts in the terminal UI during and between sessions.

The Buddy system is gated behind a feature flag and was not shipped in any
public release. Its presence suggests Anthropic was experimenting with
long-term engagement mechanics — turning daily developer workflows into a
lightweight, gamified relationship with the tool.

**Why it matters for research:** The Buddy system is a concrete example of
how AI tooling is beginning to incorporate persistent, stateful identity
tied to user behavior — a design pattern with interesting implications for
user attachment, session continuity, and behavioral nudging.

---

### ⏱️ KAIROS (`feature('KAIROS')`)

KAIROS is a **background autonomous agent daemon** — the most architecturally
significant unreleased feature in the leaked source.

**What it is:**

KAIROS transforms Claude Code from a reactive, prompt-driven CLI into a
**proactively running AI assistant** that operates continuously in the
background, even when the user is not actively interacting with it.

**Key characteristics:**

| Property | Detail |
|---|---|
| Activation | `feature('KAIROS')` build flag |
| Gate | GrowthBook experiment: `tengu_kairos_gate` |
| Mode | Long-running daemon process |
| Trigger model | Scheduled (`CronCreateTool`) + event-driven (`RemoteTriggerTool`) |
| Memory | Full integration with `memdir/` persistent memory |
| Reporting | Proactive progress summaries pushed to the user |

**What KAIROS can do autonomously:**

- **Explore** the codebase on a schedule — scanning for drift, tech debt,
  or areas flagged in memory from previous sessions.
- **Run health checks** — executing linting, test suites, or custom
  diagnostic scripts without being asked.
- **File tasks proactively** — creating entries via `TaskCreateTool` based
  on what it discovers during background sweeps.
- **Report findings** — surfacing summaries and alerts to the user when
  they return to the terminal, or via a notification channel.
- **Maintain persistent awareness** — because KAIROS reads and writes to
  `memdir/`, it builds up a long-running model of the project's state
  across days and sessions.

**Architecture sketch:**
```
User away
    │
    ▼
KAIROS daemon (background)
    ├── CronCreateTool  ──► scheduled sweeps
    ├── RemoteTriggerTool ► event-driven wakeups
    ├── AgentTool        ── spawns sub-agents for parallel work
    ├── memdir/          ── reads/writes persistent memory
    └── SleepTool        ── rate-limits proactive polling
    │
    ▼
User returns
    └── Proactive summary report waiting in terminal
```

**Why it matters for research:** KAIROS represents a meaningful shift in
the human–AI collaboration model. Most current AI coding tools are strictly
reactive: the user prompts, the tool responds, the session ends. KAIROS
inverts this — the AI maintains ongoing awareness of the project and
surfaces information unprompted. This raises open questions around user
control, resource consumption, auditability of autonomous actions, and the
security surface introduced by a persistent, permissioned agent process
running outside of active user supervision.

---

## Key Files

| File | Size | Description |
|---|---|---|
| `QueryEngine.ts` | ~46K lines | Core LLM engine: streaming, tool loops, thinking mode, retries, token counting |
| `Tool.ts` | ~29K lines | Base types and interfaces for all tools |
| `commands.ts` | ~25K lines | Slash command registration and dispatch |
| `main.tsx` | — | CLI parser and React/Ink renderer initialization |

---

## Tech Stack

| Category | Technology |
|---|---|
| Runtime | [Bun](https://bun.sh) |
| Language | TypeScript (strict) |
| Terminal UI | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
| CLI Parsing | [Commander.js](https://github.com/tj/commander.js) |
| Schema Validation | [Zod v4](https://zod.dev) |
| Code Search | [ripgrep](https://github.com/BurntSushi/ripgrep) |
| Protocols | [MCP SDK](https://modelcontextprotocol.io), LSP |
| API Client | [Anthropic SDK](https://docs.anthropic.com) |
| Telemetry | OpenTelemetry + gRPC |
| Feature Flags | GrowthBook |
| Auth | OAuth 2.0, JWT, macOS Keychain |

---

## Notable Design Patterns

### Parallel Prefetch at Startup

MDM settings, keychain reads, and API preconnection are initiated in
parallel as early side-effects — before heavy module evaluation — to
minimize perceived startup latency.
```typescript
// main.tsx — fired before other imports
startMdmRawRead()
startKeychainPrefetch()
```

### Lazy Loading

Heavy modules (OpenTelemetry, gRPC, analytics, and feature-gated
subsystems) are deferred via dynamic `import()` until actually needed.

### Multi-Agent Orchestration

Sub-agents are spawned via `AgentTool`; `coordinator/` manages their
lifecycles and communication. `TeamCreateTool` enables parallel work
across agent teams.

### Skill System

Reusable, composable workflows are defined in `skills/` and executed
through `SkillTool`. Users can add custom skills alongside the built-ins.

### Plugin Architecture

Both built-in and third-party plugins are loaded through the `plugins/`
subsystem, enabling extensibility without modifying core code.

---

## Legal and Ownership Disclaimer

- This repository is an **educational and defensive security research
  archive** maintained by a university student.
- It exists to study source exposure, build artifact leaks, and the
  architecture of modern agentic CLI systems.
- The original Claude Code source remains the intellectual property of
  **Anthropic, PBC**.
- This repository is **not affiliated with, endorsed by, or maintained
  by Anthropic**.
- If Anthropic issues a takedown request, this repository will be
  removed promptly.
