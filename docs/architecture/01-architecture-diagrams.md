# Sơ đồ kiến trúc hệ thống OpenClaw

## 1. Tổng quan kiến trúc hệ thống

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          OpenClaw System                                    │
│                                                                             │
│  ┌──────────┐    ┌──────────────────┐    ┌──────────────────────────────┐   │
│  │  CLI      │───▶│  Gateway Server  │◀───│  Channels (External)        │   │
│  │ (entry.ts)│    │  (WebSocket)     │    │  WhatsApp │ Telegram        │   │
│  └──────────┘    │  ws://127.0.0.1  │    │  Slack    │ Discord         │   │
│                  │  :18789          │    │  Signal   │ iMessage        │   │
│                  └────────┬─────────┘    │  MS Teams │ Google Chat     │   │
│                           │              └──────────────────────────────┘   │
│                           │                                                 │
│              ┌────────────▼────────────┐                                    │
│              │    Message Router        │                                    │
│              │  (routing/session-key)   │                                    │
│              └────────────┬────────────┘                                    │
│                           │                                                 │
│         ┌─────────────────▼──────────────────┐                              │
│         │          Agent System               │                              │
│         │                                     │                              │
│         │  ┌───────────┐   ┌──────────────┐  │                              │
│         │  │ Main Agent │   │ Sub-Agents   │  │                              │
│         │  │ (default)  │──▶│ (spawned)    │  │                              │
│         │  └─────┬──────┘   └──────────────┘  │                              │
│         │        │                            │                              │
│         │  ┌─────▼──────────────────────┐     │                              │
│         │  │  Pi Embedded Runner         │     │                              │
│         │  │  (LLM API call + tools)     │     │                              │
│         │  └─────┬──────────────────────┘     │                              │
│         │        │                            │                              │
│         │  ┌─────▼──────┐ ┌──────────────┐   │                              │
│         │  │ Tool System │ │ Skills System│   │                              │
│         │  │ (pi-tools)  │ │ (SKILL.md)   │   │                              │
│         │  └─────────────┘ └──────────────┘   │                              │
│         └────────────────────────────────────┘                              │
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Config System │  │ Session Store│  │ Memory System│  │ Plugin System│    │
│  │ (openclaw.json)│ │ (JSON files) │  │ (builtin/qmd)│  │ (plugins/)   │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2. Luồng khởi động ứng dụng (Application Startup Flow)

```
openclaw.mjs (Binary Entry)
    │
    ▼
entry.ts (Bootstrap)
    │  ├── Windows argv normalization
    │  ├── Node options management
    │  └── Respawn guard
    │
    ▼
src/index.ts (Main Init)
    │  ├── loadDotEnv()              ── Load .env file
    │  ├── normalizeEnv()            ── Standardize env vars
    │  ├── ensureOpenClawCliOnPath() ── Make CLI accessible
    │  ├── enableConsoleCapture()    ── Structured logging
    │  └── assertSupportedRuntime() ── Node ≥ 22.12.0
    │
    ▼
buildProgram() (CLI Program Builder)
    │  ├── Register all CLI commands
    │  ├── Parse arguments
    │  └── Execute matching command
    │
    ├──▶ "openclaw start"  ──▶ Gateway Server Start
    ├──▶ "openclaw chat"   ──▶ Interactive Chat
    ├──▶ "openclaw acp"    ──▶ ACP Bridge
    └──▶ "openclaw ..."    ──▶ Other Commands
```

## 3. Kiến trúc Gateway Server

```
┌─────────────────────────────────────────────────────────────┐
│                    Gateway Server                            │
│                  (server.impl.ts)                            │
│                                                              │
│  ┌──────────────┐                                            │
│  │ WebSocket     │◀── Client connections                     │
│  │ Server        │    (token/password auth)                  │
│  └──────┬───────┘                                            │
│         │                                                    │
│  ┌──────▼───────────────────────────────────────────┐        │
│  │              Request Dispatcher                   │        │
│  │                  (call.ts)                        │        │
│  └──┬───┬───┬───┬───┬───┬───┬───┬───────────────────┘        │
│     │   │   │   │   │   │   │   │                            │
│     ▼   ▼   ▼   ▼   ▼   ▼   ▼   ▼                           │
│  ┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐┌─────┐          │
│  │Chat││Chan││Cron││Hook││Meth││Sess││Ctrl││Model│          │
│  │    ││nels││    ││    ││ods ││ions││ UI ││Cata.│          │
│  └────┘└────┘└────┘└────┘└────┘└────┘└────┘└─────┘          │
│                                                              │
│  Sub-handlers:                                               │
│  ├── chat     : Message processing & agent invocation        │
│  ├── channels : Channel management & routing                 │
│  ├── cron     : Scheduled job execution                      │
│  ├── hooks    : Webhook integration                          │
│  ├── methods  : Gateway RPC methods                          │
│  ├── sessions : Session CRUD & history                       │
│  ├── ctrl     : Web dashboard / control UI                   │
│  └── models   : Model catalog & discovery                    │
└─────────────────────────────────────────────────────────────┘
```

## 4. Luồng xử lý tin nhắn (Message Processing Flow)

```
Incoming Message (from Channel)
    │
    ▼
┌─────────────────────┐
│  Channel Adapter     │  WhatsApp/Telegram/Slack/Discord/...
│  (src/channels/)     │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Message Router      │
│  Determine:          │
│  ├── Agent ID        │  (which agent handles this?)
│  ├── Session Key     │  agent:<agentId>:<sessionKey>
│  └── Chat Type       │  DM / Group / Thread
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Session Manager     │
│  ├── Load session    │  (from JSON store)
│  ├── Check reset     │  (daily/idle triggers)
│  ├── Apply scope     │  (per-sender/global)
│  └── Load history    │  (conversation transcript)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Agent Scope         │
│  (agent-scope.ts)    │
│  ├── Resolve config  │
│  ├── Set workspace   │
│  ├── Load skills     │
│  └── Apply model     │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────────────────────────┐
│  Pi Embedded Runner                      │
│  (pi-embedded-runner/)                   │
│                                          │
│  1. Build System Prompt                  │
│     ├── Identity, tools, skills          │
│     ├── Memory context                   │
│     └── Workspace files (MEMORY.md)      │
│                                          │
│  2. Call LLM API                         │
│     ├── Anthropic / OpenAI / Google      │
│     ├── With tool definitions            │
│     └── Stream response                  │
│                                          │
│  3. Process Tool Calls                   │
│     ├── exec (shell commands)            │
│     ├── read/write/edit (files)          │
│     ├── web_search / web_fetch           │
│     ├── message (send to channels)       │
│     ├── sessions_spawn (sub-agents)      │
│     └── ... other tools                  │
│                                          │
│  4. Loop until complete                  │
│     (tool results → LLM → tools → ...)  │
│                                          │
│  5. Return final response                │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────┐
│  Response Delivery   │
│  ├── Save to session │
│  ├── Send via channel│
│  └── Update metrics  │
└─────────────────────┘
```

## 5. Hệ thống Agent và Sub-Agent

```
┌────────────────────────────────────────────────────────────────┐
│                     Agent System                                │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Agent Registry                         │   │
│  │  agents.list[] in openclaw.json                          │   │
│  │                                                           │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐               │   │
│  │  │ main     │  │ coder    │  │ writer   │  ... more      │   │
│  │  │ (default)│  │          │  │          │                │   │
│  │  └────┬─────┘  └──────────┘  └──────────┘               │   │
│  │       │                                                   │   │
│  └───────┼──────────────────────────────────────────────────┘   │
│          │                                                      │
│          │ sessions_spawn()                                     │
│          ▼                                                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │               Sub-Agent Registry                          │   │
│  │             (subagent-registry.ts)                        │   │
│  │                                                           │   │
│  │  ┌─────────────────┐  ┌─────────────────┐               │   │
│  │  │ Sub-Agent Run 1  │  │ Sub-Agent Run 2  │  ...          │   │
│  │  │                  │  │                  │               │   │
│  │  │ agentId: "coder" │  │ agentId: "writer"│               │   │
│  │  │ runId: "abc-123" │  │ runId: "def-456" │               │   │
│  │  │ task: "Fix bug"  │  │ task: "Write doc"│               │   │
│  │  │ status: running  │  │ status: done     │               │   │
│  │  └─────────────────┘  └─────────────────┘               │   │
│  │                                                           │   │
│  │  Lifecycle:                                               │   │
│  │  Register → Track → Execute → Announce → Archive          │   │
│  │                                                           │   │
│  │  Session Key Format:                                      │   │
│  │  agent:main:subagent:<runId>:<childKey>                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │               Per-Agent Configuration                     │   │
│  │                                                           │   │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐            │   │
│  │  │   Model    │ │  Skills    │ │  Tools     │            │   │
│  │  │  Override  │ │  Allowlist │ │  Policy    │            │   │
│  │  └────────────┘ └────────────┘ └────────────┘            │   │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐            │   │
│  │  │  Workspace │ │  Sandbox   │ │  Identity  │            │   │
│  │  │  Directory │ │  Settings  │ │  Config    │            │   │
│  │  └────────────┘ └────────────┘ └────────────┘            │   │
│  └──────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────┘
```

## 6. Hệ thống Tool (Tool System)

```
┌────────────────────────────────────────────────────────────┐
│                      Tool System                            │
│                    (pi-tools.ts)                            │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Tool Policy Engine                      │    │
│  │            (pi-tools.policy.ts)                      │    │
│  │                                                      │    │
│  │  Profiles: minimal │ coding │ messaging │ full       │    │
│  │  + allow[] / deny[] / alsoAllow[]                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌─────────── File Tools ──────────┐                        │
│  │ read │ write │ edit │ apply_patch│                        │
│  │ grep │ find  │ ls               │                        │
│  └─────────────────────────────────┘                        │
│                                                             │
│  ┌─────────── Exec Tools ──────────┐                        │
│  │ exec (shell commands + PTY)     │                        │
│  │ process (background management) │                        │
│  └─────────────────────────────────┘                        │
│                                                             │
│  ┌─────────── Web Tools ───────────┐                        │
│  │ web_search (Brave API)          │                        │
│  │ web_fetch  (URL extraction)     │                        │
│  │ browser   (browser control)     │                        │
│  └─────────────────────────────────┘                        │
│                                                             │
│  ┌─────────── Comms Tools ─────────┐                        │
│  │ message (send to channels)      │                        │
│  │ sessions_send (cross-session)   │                        │
│  │ sessions_spawn (sub-agents)     │                        │
│  │ agents_list / sessions_list     │                        │
│  └─────────────────────────────────┘                        │
│                                                             │
│  ┌─────────── Media Tools ─────────┐                        │
│  │ image (analysis)                │                        │
│  │ canvas (push/eval/snapshot)     │                        │
│  │ nodes (camera, screen, notif.)  │                        │
│  └─────────────────────────────────┘                        │
│                                                             │
│  ┌─────────── System Tools ────────┐                        │
│  │ cron (scheduled jobs)           │                        │
│  │ gateway (restart/config)        │                        │
│  │ session_status (usage/metrics)  │                        │
│  └─────────────────────────────────┘                        │
└────────────────────────────────────────────────────────────┘
```

## 7. Hệ thống Session (Session Management)

```
┌────────────────────────────────────────────────────────────┐
│                    Session System                           │
│                                                             │
│  Session Key Format:                                        │
│  agent:<agentId>:<sessionKey>                              │
│                                                             │
│  Examples:                                                  │
│  ├── agent:main:main              (default session)         │
│  ├── agent:main:dm:john           (per-peer DM)            │
│  ├── agent:coder:main             (coder agent, main)       │
│  └── agent:main:subagent:abc:task (sub-agent session)       │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Session Scopes                           │   │
│  │                                                       │   │
│  │  session.scope:                                       │   │
│  │  ├── "per-sender"  ── Isolated per message sender     │   │
│  │  └── "global"      ── Shared across all senders       │   │
│  │                                                       │   │
│  │  session.dmScope:                                     │   │
│  │  ├── "main"                    ── Single main session  │   │
│  │  ├── "per-peer"               ── One per contact      │   │
│  │  ├── "per-channel-peer"       ── Per channel+contact  │   │
│  │  └── "per-account-channel-peer" ── Full isolation     │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Session Reset Triggers                   │   │
│  │                                                       │   │
│  │  reset.mode:                                          │   │
│  │  ├── "daily"  ── Reset tại giờ cụ thể (atHour: 0-23) │   │
│  │  └── "idle"   ── Reset sau thời gian idle (minutes)   │   │
│  │                                                       │   │
│  │  Có thể cấu hình khác nhau theo channel:              │   │
│  │  resetByChannel: { "whatsapp": {...}, "slack": {...} } │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  Storage: ~/.openclaw/state/sessions/<agent>/store.json     │
└────────────────────────────────────────────────────────────┘
```

## 8. Hệ thống Channel (Kênh giao tiếp)

```
┌────────────────────────────────────────────────────────────┐
│                    Channel System                           │
│                  (src/channels/)                            │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ WhatsApp │  │ Telegram │  │  Slack   │  │ Discord  │   │
│  │ (Baileys)│  │ (grammY) │  │ (Bolt)   │  │(discord.js)│  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │              │              │              │         │
│  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐   │
│  │ Signal   │  │ iMessage │  │ MS Teams │  │ Google   │   │
│  │          │  │(BlueBubb)│  │          │  │ Chat     │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │              │              │              │         │
│       └──────────────┴──────┬───────┴──────────────┘         │
│                             │                                │
│                    ┌────────▼────────┐                        │
│                    │ Channel Adapter │                        │
│                    │ (Unified API)   │                        │
│                    │                 │                        │
│                    │ ├── sendMessage │                        │
│                    │ ├── editMessage │                        │
│                    │ ├── react       │                        │
│                    │ ├── typing      │                        │
│                    │ └── getHistory  │                        │
│                    └────────┬────────┘                        │
│                             │                                │
│                    ┌────────▼────────┐                        │
│                    │ Message Router  │                        │
│                    │                 │                        │
│                    │ DM / Group /    │                        │
│                    │ Thread routing  │                        │
│                    │ Mention gating  │                        │
│                    └─────────────────┘                        │
└────────────────────────────────────────────────────────────┘
```

## 9. Hệ thống Configuration (Cấu hình)

```
┌────────────────────────────────────────────────────────────┐
│              Configuration System                           │
│                (src/config/)                                │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Config Loading Flow                      │   │
│  │                                                       │   │
│  │  1. Load openclaw.json                                │   │
│  │     └── Search: workspace → ~/.openclaw/ → defaults   │   │
│  │                                                       │   │
│  │  2. Validate with Zod Schema                          │   │
│  │     └── zod-schema.ts (comprehensive validation)      │   │
│  │                                                       │   │
│  │  3. Merge with defaults                               │   │
│  │     └── Missing values → default values               │   │
│  │                                                       │   │
│  │  4. Apply environment overrides                       │   │
│  │     └── env vars > config file values                 │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Config Sections                          │   │
│  │                                                       │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐    │   │
│  │  │ agents  │ │ models  │ │  tools  │ │channels │    │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘    │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐    │   │
│  │  │ session │ │ plugins │ │  hooks  │ │ memory  │    │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘    │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐    │   │
│  │  │ browser │ │ gateway │ │ sandbox │ │ logging │    │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘    │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  Type Definitions: 30+ modular type files                   │
│  types.agents.ts, types.channels.ts, types.tools.ts, ...    │
└────────────────────────────────────────────────────────────┘
```

## 10. Cấu trúc thư mục dự án

```
openclaw/
├── src/                              # Source code chính
│   ├── index.ts                      # Entry point export
│   ├── entry.ts                      # CLI bootstrapping
│   ├── agents/                       # Hệ thống Agent (core)
│   │   ├── agent-scope.ts            # Agent ID, config, workspace
│   │   ├── pi-tools.ts              # Tool creation & registration
│   │   ├── pi-tools.policy.ts       # Tool policy engine
│   │   ├── pi-embedded.ts           # Embedded agent runner (RPC)
│   │   ├── pi-embedded-runner/      # LLM execution loop
│   │   ├── system-prompt.ts         # System prompt builder
│   │   ├── subagent-registry.ts     # Sub-agent lifecycle
│   │   ├── subagent-announce.ts     # Sub-agent notifications
│   │   └── skills/                  # Skill loader
│   ├── config/                       # Configuration system
│   │   ├── config.ts                # Main config loader
│   │   ├── types.ts                 # Root type definitions
│   │   ├── zod-schema.ts            # Validation schema
│   │   └── types.*.ts               # Modular type files (30+)
│   ├── gateway/                      # WebSocket gateway server
│   │   ├── server.impl.ts           # Main server implementation
│   │   ├── server/                  # Sub-handlers
│   │   ├── client.ts                # Client management
│   │   ├── call.ts                  # API dispatcher
│   │   ├── auth.ts                  # Authentication
│   │   └── session-utils.ts         # Session CRUD
│   ├── channels/                     # Messaging channels
│   │   ├── whatsapp/                # WhatsApp (Baileys)
│   │   ├── telegram/                # Telegram (grammY)
│   │   ├── slack/                   # Slack (Bolt)
│   │   ├── discord/                 # Discord (discord.js)
│   │   ├── signal/                  # Signal
│   │   ├── imessage/                # iMessage (BlueBubbles)
│   │   ├── msteams/                 # Microsoft Teams
│   │   └── googlechat/              # Google Chat
│   ├── acp/                          # Agent Client Protocol
│   ├── plugins/                      # Plugin system
│   ├── hooks/                        # Hook system
│   ├── sessions/                     # Session management
│   ├── routing/                      # Message routing
│   ├── memory/                       # Memory system
│   ├── media-understanding/          # Media analysis
│   ├── cli/                          # CLI infrastructure
│   ├── commands/                     # CLI commands
│   └── infra/                        # Infrastructure utils
├── skills/                           # 54 built-in skills
├── extensions/                       # Channel extensions
├── packages/                         # NPM workspaces
├── apps/                             # Desktop/mobile apps
├── docs/                             # Documentation
└── openclaw.mjs                      # CLI binary entry point
```
