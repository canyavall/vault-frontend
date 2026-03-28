---
title: Claude Code Execution Engines
category: claude-engines
tags:
  - claude-code
  - execution-engines
  - pty-terminal
  - subprocess
  - architecture
  - integration
  - ipc-protocol
  - node-pty
  - websocket
  - interactive-terminal
description: >-
  The two Claude Code execution engines in Nori (PTY terminal and subprocess),
  when to use each, and why the Claude Code SDK must NOT be used.
when: >-
  When implementing Claude Code execution, integrating with the Terminal or Chat
  tabs, handling interactive CLI sessions with ANSI rendering, or deciding
  between engine architectures for code generation workflows.
rules:
  - packages/server/src/pty/**/*.ts
  - packages/server/src/pty/**/*.cjs
  - packages/server/src/subprocess/**/*.ts
  - packages/server/src/routes/pty.routes.ts
  - packages/server/src/routes/subprocess.routes.ts
required_knowledge: []
created: '2026-03-28T13:32:27.668Z'
updated: '2026-03-28T13:32:27.668Z'
---
# Claude Code Execution Engines

Nori runs Claude Code (the CLI tool) via two distinct engines. Choosing the wrong one breaks the use case. The SDK must NEVER be used. All rules below are MANDATORY.

---

## Decision Tree: Which Engine to Use?

```
Need full interactive terminal with ANSI rendering?
├── YES → Engine 1: PTY Terminal
│         (power-user "Terminal" tab, xterm.js frontend)
└── NO → Need structured JSON output, cost tracking, or tool visibility?
          ├── YES → Engine 2: Subprocess (`claude -p`)
          │         (Chat tab, knowledge generation, flow-chat)
          └── You want to use the SDK?
                    → STOP. Do NOT use the SDK. See "Why NOT the SDK" below.
```

---

## Engine 1: PTY Terminal (Interactive)

**When to use**: Full interactive Claude Code CLI sessions rendered in a terminal emulator (xterm.js). The user types, sees ANSI colours, uses arrow keys, Ctrl+C, etc.

**Entry points**:
- Worker: `packages/server/src/pty/pty-worker.cjs` — spawns `claude` via `node-pty`
- Manager: `packages/server/src/pty/pty-manager.ts` — Bun→Node IPC bridge
- Handler: `packages/server/src/pty/pty-handler.ts` — WebSocket message routing + ANSI intercepts
- Route: `packages/server/src/routes/pty.routes.ts` — WebSocket upgrade at `GET /api/pty`

**Architecture**: Bun cannot run `node-pty` on Windows without `ERR_SOCKET_CLOSED` crashes. Solution: Bun server forks a plain Node.js worker (`pty-worker.cjs`) via IPC. All PTY operations go through the worker.

```
Bun server
  └─ forks pty-worker.cjs (Node.js)
       └─ node-pty.spawn(claudePath, ['--session-id', sessionId], {
            name: 'xterm-256color',
            cwd: projectPath,
            useConpty: true,
            useConptyDll: false,
          })
```

**IPC protocol** (Bun parent ↔ Node worker):

| Direction | Message |
|-----------|---------|
| Parent → Worker | `{ cmd: 'spawn', id, projectPath, cols, rows, sessionId? }` |
| Parent → Worker | `{ cmd: 'write', id, data }` |
| Parent → Worker | `{ cmd: 'resize', id, cols, rows }` |
| Parent → Worker | `{ cmd: 'kill', id }` |
| Worker → Parent | `{ evt: 'spawned' \| 'data' \| 'exit' \| 'error', id, ... }` |

**WebSocket protocol** (server ↔ frontend):

| Direction | Message |
|-----------|---------|
| Client → Server | `{ type: 'spawn', project_path, cols?, rows?, session_id? }` |
| Client → Server | `{ type: 'input', data }` |
| Client → Server | `{ type: 'resize', cols, rows }` |
| Server → Client | `{ type: 'output', data }` (raw ANSI string) |
| Server → Client | `{ type: 'exit', code }` |

**Terminal capability intercepts** (`pty-handler.ts` intercepts these before forwarding to xterm.js):

| Sequence | Meaning | Response |
|----------|---------|----------|
| `?9001h` | Win32 input mode | STRIPPED — do NOT respond with `?9001l`; ConPTY handles VT natively |
| `>0q` | XTVERSION | `xterm(370)` |
| `0c` | DA1 (Primary Device Attributes) | `?1;2c` |
| `>Nc` | DA2 (Secondary Device Attributes) | `>41;370;0c` |
| `5n` | DSR (Device Status Report) | `0n` |
| `18t` | Window size report | Current `cols`/`rows` |

**Environment**: ALWAYS delete `CLAUDECODE` env var to prevent nested sessions. Strip Windows Terminal vars (`WT_SESSION`, `WT_PROFILE_ID`, `TERM_PROGRAM`) when not needed.

---

## Engine 2: Subprocess with `claude -p` (Structured)

**When to use**: Programmatic interactions where you need structured output — cost tracking, token counts, tool call visibility, or feeding results back into app logic. Used by: Chat tab, knowledge generation, flow-chat.

**Entry points**:
- Spawner: `packages/core/src/features/cc-session/cc-subprocess/actions/spawn-claude.ts`
- Parser: `packages/core/src/features/cc-session/cc-subprocess/actions/parse-stream-json.ts`
- Orchestrator: `packages/core/src/features/cc-session/cc-subprocess/cc-subprocess.ts`
- Types: `packages/shared/src/types/cc-subprocess.ts`

**Spawn pattern**:

```typescript
import { spawnClaudeSubprocess } from './actions/spawn-claude.js';

const handle = await spawnClaudeSubprocess(prompt, {
  projectPath: '/path/to/project',
  model: 'claude-opus-4-5',            // optional
  sessionId: 'existing-session-id',    // optional — resume session
  maxTurns: 5,                          // optional
  permissionMode: 'bypassPermissions', // optional
  appendSystemPrompt: '...',           // optional
  disallowedTools: 'Bash',            // optional — comma-separated
  allowedTools: 'Read,Write',         // optional — comma-separated
});

handle.onMessage((msg) => { /* CcSubprocessMessage */ });
handle.onStderr((text) => { /* raw stderr */ });
handle.onExit((code) => { /* exit code */ });
handle.kill(); // terminate early
```

**CLI args built** (exact order):

```
claude -p --output-format stream-json
  [--model <model>]
  [--session-id <id>]
  [--max-turns <n>]
  [--permission-mode <mode>]
  [--append-system-prompt <text>]
  [--disallowed-tools <tools>]    ← kebab-case
  [--allowedTools <tools>]        ← camelCase (matches Claude Code CLI's own inconsistency)
  <prompt>
```

**NDJSON message types from `--output-format stream-json`**:

| `type` field | Key fields | Meaning |
|---|---|---|
| `system` (subtype `init`) | `session_id`, `model`, `tools[]`, `mcp_servers[]`, `claude_code_version` | Session initialised |
| `assistant` | `message.content[]` (text/tool_use blocks), `message.usage`, `session_id` | Claude response |
| `result` (subtype `success`/`error`) | `result`, `session_id`, `total_cost_usd`, `duration_ms`, `num_turns`, `usage`, `is_error` | Final result |
| `rate_limit_event` | `rate_limit_info.status`, `rate_limit_info.resetsAt` | Rate limit hit |

**SSE events emitted** (by `parse-stream-json.ts` via `FlowEmitter`):

| SSE event | Triggered by |
|---|---|
| `cc:subprocess:init` | `system` init message |
| `cc:subprocess:text` | `assistant` text content block |
| `cc:subprocess:tool_use` | `assistant` tool_use content block |
| `cc:subprocess:message` | every `assistant` message (full) |
| `cc:subprocess:completed` | `result` with `is_error: false` |
| `cc:subprocess:error` | `result` with `is_error: true` |

**Result shape** (`CcSubprocessData`):

```typescript
{
  result: string,          // final text answer
  session_id: string,
  total_cost_usd: number,
  duration_ms: number,
  num_turns: number,
  usage: { input_tokens: number, output_tokens: number },
}
```

---

## Why NOT the Claude Code SDK

The `@anthropic-ai/claude-code` npm package (also called Claude Agent SDK) MUST NOT be used in Nori. This is a hard architectural decision.

| Reason | Detail |
|--------|--------|
| **Authentication mismatch** | Requires a linked Claude Code subscription. An Anthropic API key from platform.claude.com alone is insufficient — it silently fails. |
| **Minimal system prompt** | The SDK does NOT use Claude Code's full system prompt. All Claude Code capabilities that depend on the full prompt are degraded or unavailable. |
| **No filesystem access by default** | Runs in a temp directory with zero tools enabled. Must be explicitly reconfigured to match what `claude -p` provides out of the box. |
| **Proprietary license** | The Claude Agent SDK ships with a proprietary license that restricts redistribution and usage scenarios incompatible with Nori's distribution model. |
| **No benefit over `claude -p`** | Spawning `claude -p --output-format stream-json` achieves identical results with zero npm dependencies added. Unix philosophy: use the CLI. |
| **CLI-first architecture** | Nori delegates to the Claude Code CLI installed on the user's machine. This keeps capabilities in sync with the user's Claude Code version automatically — no SDK version pinning needed. |

---

## Anti-Patterns

```typescript
// NEVER — SDK usage
import { ClaudeCode } from '@anthropic-ai/claude-code';
const sdk = new ClaudeCode({ apiKey: process.env.ANTHROPIC_API_KEY });
// Auth fails silently; minimal system prompt; no filesystem; proprietary license

// NEVER — Parsing JSON from PTY output
// PTY streams raw ANSI escape sequences. JSON.parse will always fail on them.
pty.onData((raw) => JSON.parse(raw));

// NEVER — Feeding subprocess output into xterm.js
// subprocess output is NDJSON lines, not a VT100 stream.
const handle = await spawnClaudeSubprocess(prompt, opts);
terminal.write(handle.stdout); // xterm.js will render garbage

// NEVER — Running node-pty directly from Bun
// node-pty's write pipe crashes on Windows under Bun (ERR_SOCKET_CLOSED).
// Always go through pty-manager.ts → pty-worker.cjs (Node.js).
import pty from 'node-pty';
pty.spawn('claude', []); // broken on Windows/Bun

// NEVER — Forgetting to delete CLAUDECODE env var
// Causes undefined behaviour in nested Claude Code invocations.
spawn('claude', ['-p', prompt], {
  env: process.env, // WRONG — must delete env.CLAUDECODE
});
```

---

## Comparison Table

| | PTY Terminal | Subprocess (`claude -p`) | SDK |
|---|---|---|---|
| Output format | Raw ANSI/VT stream | Structured NDJSON | SDK objects |
| Use case | Interactive terminal UI | Programmatic / flow-driven | FORBIDDEN |
| Cost tracking | No | Yes (`total_cost_usd`) | — |
| Tool call visibility | No (embedded in ANSI) | Yes (`tool_use` blocks) | — |
| Session resume | Yes (`--session-id`) | Yes (`--session-id`) | — |
| Windows compat | Node.js worker required | Native `child_process` | — |
| Frontend transport | WebSocket → xterm.js | SSE events from server | — |
| Defined in | `packages/server/src/pty/` | `packages/core/src/features/cc-session/cc-subprocess/` | Never add |
