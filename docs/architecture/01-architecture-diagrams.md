# Sơ đồ kiến trúc hệ thống OpenClaw

## 1. Tổng quan kiến trúc hệ thống

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          OpenClaw System                                    │
│                                                                             │
│  ┌───────────┐   ┌──────────────────┐   ┌──────────────────────────────┐  │
│  │   CLI      │──▶│  Gateway Server  │◀──│  Channels (31 extensions)    │  │
│  │ (entry.ts) │   │  (WebSocket+TLS) │   │  Discord  │ Telegram        │  │
│  └───────────┘   │  :18789          │   │  Slack    │ WhatsApp        │  │
│                  │                  │   │  Signal   │ iMessage        │  │
│  ┌───────────┐   │  HTTP Endpoints:  │   │  Matrix   │ Zalo            │  │
│  │  Web UI   │──▶│  /v1/chat/compl.  │   │  Twitch   │ Nostr           │  │
│  │ (ctrl UI) │   │  /v1/responses    │   │  Tlon     │ Line  ...       │  │
│  └───────────┘   └────────┬─────────┘   └──────────────────────────────┘  │
│                           │                                                │
│  ┌───────────┐   ┌───────▼────────┐   ┌──────────────────────────────┐    │
│  │  ACP      │──▶│ Message Router │──▶│         Agent System          │    │
│  │ (Agent    │   │ (routing/      │   │                               │    │
│  │  Control  │   │  session-key)  │   │  ┌──────────┐ ┌───────────┐  │    │
│  │  Protocol)│   └────────────────┘   │  │Main Agent│▶│Sub-Agents │  │    │
│  └───────────┘                        │  └────┬─────┘ └───────────┘  │    │
│                                       │       │                      │    │
│  ┌───────────┐                        │  ┌────▼──────────────────┐   │    │
│  │  Apps     │                        │  │  Pi Embedded Runner   │   │    │
│  │ iOS/macOS │                        │  │  (LLM API + tools)    │   │    │
│  │ Android   │                        │  └────┬──────────────────┘   │    │
│  └───────────┘                        │       │                      │    │
│                                       │  ┌────▼─────┐ ┌──────────┐  │    │
│                                       │  │Tool Sys. │ │Skills(53)│  │    │
│                                       │  │(pi-tools)│ │(SKILL.md)│  │    │
│                                       │  └──────────┘ └──────────┘  │    │
│                                       └──────────────────────────────┘    │
│                                                                           │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐  │
│  │  Config   │ │  Session  │ │  Memory   │ │  Plugin   │ │Extensions │  │
│  │ (openclaw │ │  Store    │ │  System   │ │  System   │ │  (31)     │  │
│  │  .json)   │ │           │ │(builtin/  │ │(plugin-sdk│ │(channels, │  │
│  │ 32 keys   │ │           │ │  qmd)     │ │ registry) │ │ providers)│  │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘ └───────────┘  │
└───────────────────────────────────────────────────────────────────────────┘
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
┌──────────────────────────────────────────────────────────────────┐
│                       Channel System                              │
│         Dynamic plugin-based channel loading (31 extensions)      │
│                                                                   │
│  ┌─────────────── Core (src/channels/) ───────────────────────┐  │
│  │  dock.ts         ── Channel registry interface              │  │
│  │  plugins/index.ts ── Plugin loader & discovery              │  │
│  │  registry.ts     ── Runtime registry with ordering          │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌─── Extension Channels (extensions/) ───────────────────────┐  │
│  │                                                             │  │
│  │  Messaging:                                                 │  │
│  │  ├── discord (discord.js)   ├── telegram (grammY)           │  │
│  │  ├── slack (Bolt)           ├── whatsapp (Baileys)          │  │
│  │  ├── signal                 ├── imessage (BlueBubbles)      │  │
│  │  ├── googlechat             ├── msteams                     │  │
│  │  ├── matrix                 ├── mattermost                  │  │
│  │  ├── line                   ├── feishu (Lark)               │  │
│  │  ├── tlon (Urbit)           ├── twitch                      │  │
│  │  ├── nostr                  ├── nextcloud-talk              │  │
│  │  ├── zalo                   └── lobster                     │  │
│  │                                                             │  │
│  │  Infrastructure:                                            │  │
│  │  ├── voice-call             ├── diagnostics-otel            │  │
│  │  ├── copilot-proxy          ├── llm-task                    │  │
│  │  └── memory-core / memory-lancedb                           │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌─────────────── Channel Plugin API ─────────────────────────┐  │
│  │  ├── sendText / sendMedia  ── Outbound messaging            │  │
│  │  ├── editMessage           ── Message editing               │  │
│  │  ├── react                 ── Emoji reactions               │  │
│  │  ├── typing                ── Typing indicators             │  │
│  │  ├── pairing               ── DM access pairing flow        │  │
│  │  ├── onboarding            ── Channel setup wizard          │  │
│  │  └── probeAccount          ── Health check & validation     │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌─────────────── Message Routing ────────────────────────────┐  │
│  │  ├── Agent binding (channel → agent routing)                │  │
│  │  ├── DM / Group / Thread / Topic routing                    │  │
│  │  ├── Mention gating & access control                        │  │
│  │  └── Per-group/per-topic agent override                     │  │
│  └─────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
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
│  │          Config Sections (32 top-level keys)         │   │
│  │                                                       │   │
│  │  Core:                                                │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌─────────┐        │   │
│  │  │ agents │ │ models │ │ tools  │ │channels │        │   │
│  │  └────────┘ └────────┘ └────────┘ └─────────┘        │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌─────────┐        │   │
│  │  │session │ │plugins │ │ hooks  │ │ memory  │        │   │
│  │  └────────┘ └────────┘ └────────┘ └─────────┘        │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌─────────┐        │   │
│  │  │browser │ │gateway │ │ skills │ │  cron   │        │   │
│  │  └────────┘ └────────┘ └────────┘ └─────────┘        │   │
│  │                                                       │   │
│  │  Infrastructure:                                      │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌─────────┐        │   │
│  │  │logging │ │  auth  │ │ update │ │  media  │        │   │
│  │  └────────┘ └────────┘ └────────┘ └─────────┘        │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌─────────┐        │   │
│  │  │  talk  │ │  web   │ │  meta  │ │   env   │        │   │
│  │  └────────┘ └────────┘ └────────┘ └─────────┘        │   │
│  │  ┌─────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │   │
│  │  │bindings │ │broadcast │ │approvals │ │discovery │  │   │
│  │  └─────────┘ └──────────┘ └──────────┘ └──────────┘  │   │
│  │  + ui, audio, commands, messages, nodeHost,           │   │
│  │    canvasHost, diagnostics, wizard                    │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  Type Definitions: 29 modular type files                    │
│  types.agents.ts, types.channels.ts, types.tools.ts, ...    │
└────────────────────────────────────────────────────────────┘
```

## 10. Cấu trúc thư mục dự án

```
openclaw/
├── src/                              # Source code chính (52 thư mục con)
│   ├── index.ts                      # Entry point + public API exports
│   ├── entry.ts                      # CLI bootstrapping & respawn
│   ├── agents/                       # Hệ thống Agent (core, 259+ files)
│   │   ├── agent-scope.ts            # Agent ID, config, workspace
│   │   ├── pi-tools.ts              # Tool creation & registration
│   │   ├── pi-tools.policy.ts       # Tool policy engine
│   │   ├── pi-embedded.ts           # Embedded agent runner (RPC)
│   │   ├── pi-embedded-runner/      # LLM execution loop
│   │   ├── system-prompt.ts         # System prompt builder
│   │   ├── subagent-registry.ts     # Sub-agent lifecycle
│   │   ├── subagent-announce.ts     # Sub-agent notifications
│   │   ├── tools/                   # Tool implementations
│   │   │   └── sessions-spawn-tool.ts
│   │   └── skills/                  # Skill loader & installation
│   ├── config/                       # Configuration system (80+ files)
│   │   ├── config.ts                # Main config loader
│   │   ├── io.ts                    # Config I/O
│   │   ├── paths.ts                 # Config path resolution
│   │   ├── schema.ts                # Master schema (55K lines)
│   │   ├── zod-schema.ts            # Zod validation (32 top-level keys)
│   │   ├── zod-schema.providers*.ts # Provider-specific schemas
│   │   ├── types.openclaw.ts        # Root OpenClawConfig type
│   │   └── types.*.ts               # 29 modular type files
│   ├── gateway/                      # Gateway server
│   │   ├── server.impl.ts           # Main implementation (21K lines)
│   │   ├── server/                  # WS, TLS, health, plugin HTTP
│   │   ├── client.ts                # Client management
│   │   ├── call.ts                  # API dispatcher
│   │   └── auth.ts                  # Authentication
│   ├── channels/                     # Channel system (dynamic loading)
│   │   ├── dock.ts                  # Channel registry interface
│   │   ├── plugins/index.ts         # Plugin-based channel loader
│   │   └── registry.ts             # Runtime registry
│   ├── acp/                          # Agent Control Protocol (13 files)
│   ├── plugins/                      # Plugin system (36+ files)
│   ├── plugin-sdk/                   # Plugin SDK (12K+ lines)
│   ├── hooks/                        # Hook system (14K lines)
│   ├── sessions/                     # Session management
│   ├── routing/                      # Message routing
│   ├── memory/                       # Memory system (builtin + QMD)
│   ├── media-understanding/          # Media analysis
│   ├── cli/                          # CLI infrastructure (250+ files)
│   ├── commands/                     # CLI commands (260+ files)
│   │   ├── agent/                   # Agent management
│   │   ├── channels/                # Channel management
│   │   ├── gateway-status/          # Gateway status
│   │   ├── models/                  # Model management
│   │   └── onboarding/              # Setup wizard
│   └── infra/                        # Infrastructure utils (40+ files)
├── extensions/                       # 31 extension packages
│   ├── discord/                     # Discord (discord.js)
│   ├── telegram/                    # Telegram (grammY)
│   ├── slack/                       # Slack (Bolt)
│   ├── whatsapp/                    # WhatsApp (Baileys)
│   ├── signal/                      # Signal
│   ├── imessage/ + bluebubbles/     # iMessage
│   ├── matrix/                      # Matrix protocol
│   ├── zalo/                        # Zalo Bot API
│   ├── msteams/                     # Microsoft Teams
│   ├── googlechat/                  # Google Chat
│   ├── mattermost/                  # Mattermost
│   ├── feishu/                      # Feishu/Lark
│   ├── line/                        # LINE
│   ├── tlon/                        # Tlon/Urbit
│   ├── twitch/                      # Twitch
│   ├── nostr/                       # Nostr protocol
│   ├── voice-call/                  # Voice call capability
│   ├── copilot-proxy/               # GitHub Copilot proxy
│   ├── diagnostics-otel/            # OpenTelemetry diagnostics
│   ├── memory-core/ + memory-lancedb/ # Memory backends
│   └── ...                          # + lobster, llm-task, etc.
├── skills/                           # 53 built-in skills
├── packages/                         # NPM workspaces (clawdbot, moltbot)
├── apps/                             # Native apps
│   ├── android/                     # Android (Gradle)
│   ├── ios/                         # iOS (Swift/Xcode)
│   ├── macos/                       # macOS (Swift/Xcode)
│   └── shared/                      # Shared OpenClawKit
├── docs/                             # Documentation
├── test/                             # Tests (vitest)
├── scripts/                          # Build & CI scripts
└── openclaw.mjs                      # CLI binary entry point
```
