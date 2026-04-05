# Hướng dẫn cấu hình openclaw.json

File `openclaw.json` là file cấu hình chính của OpenClaw. File này thường nằm tại `~/.openclaw/openclaw.json` hoặc trong thư mục workspace của dự án.

## Mục lục

File `openclaw.json` có **32 top-level keys**. Dưới đây là tài liệu chi tiết cho các section chính:

1. [Meta](#1-meta)
2. [Environment Variables](#2-environment-variables-env)
3. [Agents](#3-agents)
4. [Models](#4-models)
5. [Tools](#5-tools)
6. [Channels](#6-channels) (bao gồm [Telegram](#cấu-hình-chi-tiết-telegram-channelstelegram) và [Zalo](#cấu-hình-chi-tiết-zalo-channelszalo))
7. [Session](#7-session)
8. [Plugins](#8-plugins)
9. [Hooks](#9-hooks)
10. [Memory](#10-memory)
11. [Browser](#11-browser)
12. [Gateway](#12-gateway)
13. [Sandbox](#13-sandbox)
14. [Logging](#14-logging)
15. [Skills](#15-skills)
16. [Cron](#16-cron)
17. [Talk (TTS)](#17-talk-tts)
18. [Auth](#18-auth)
19. [Discovery](#19-discovery)
20. [Các section khác](#20-các-section-khác)

---

## 1. Meta

Thông tin metadata về file cấu hình.

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `meta.lastTouchedVersion` | `string` | Phiên bản OpenClaw cuối cùng chỉnh sửa config |
| `meta.lastTouchedAt` | `string` | Thời điểm chỉnh sửa cuối (ISO 8601) |

```json
{
  "meta": {
    "lastTouchedVersion": "1.2.0",
    "lastTouchedAt": "2026-03-18T10:00:00Z"
  }
}
```

---

## 2. Environment Variables (`env`)

Cấu hình biến môi trường cho toàn bộ hệ thống.

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `env.shellEnv.enabled` | `boolean` | Bật load env từ shell |
| `env.shellEnv.timeoutMs` | `number` | Timeout cho shell env loading (ms) |
| `env.vars` | `Record<string, string>` | Biến môi trường tùy ý |

```json
{
  "env": {
    "vars": {
      "ANTHROPIC_API_KEY": "sk-ant-...",
      "OPENAI_API_KEY": "sk-...",
      "BRAVE_API_KEY": "BSA..."
    }
  }
}
```

---

## 3. Agents

Cấu hình hệ thống agent - bao gồm cài đặt mặc định và danh sách agent cụ thể.

### 3.1 Agent Defaults (`agents.defaults`)

Cài đặt mặc định áp dụng cho tất cả agent nếu không được override.

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `model` | `string \| {primary, fallbacks[]}` | Model AI sử dụng | - |
| `workspace` | `string` | Thư mục workspace mặc định | `"."` |
| `repoRoot` | `string` | Repository root cho system prompt | - |
| `skipBootstrap` | `boolean` | Bỏ qua tạo bootstrap.md | `false` |
| `bootstrapMaxChars` | `number` | Max ký tự bootstrap files | `20000` |
| `contextTokens` | `number` | Context window cap (tokens) | - |
| `userTimezone` | `string` | IANA timezone cho user | - |
| `timeFormat` | `"auto" \| "12" \| "24"` | Định dạng thời gian | `"auto"` |
| `memorySearch` | `object` | Cấu hình tìm kiếm bộ nhớ | - |
| `thinkingDefault` | `"off" \| "minimal" \| "low" \| "medium" \| "high" \| "xhigh"` | Mức thinking mặc định | - |
| `verboseDefault` | `"off" \| "on" \| "full"` | Mức verbose mặc định | `"off"` |
| `blockStreamingDefault` | `"off" \| "on"` | Block streaming mặc định | `"off"` |
| `blockStreamingBreak` | `"text_end" \| "message_end"` | Streaming boundary | - |
| `humanDelay` | `object` | Cấu hình delay giả lập người (mode, minMs, maxMs) | - |
| `heartbeat` | `object` | Cấu hình heartbeat (every, model, session, target, activeHours) | - |
| `identity` | `object` | Cấu hình danh tính agent | - |
| `groupChat` | `object` | Cấu hình chat nhóm | - |
| `maxConcurrent` | `number` | Số request đồng thời tối đa | `1` |
| `timeoutSeconds` | `number` | Global run timeout (giây) | - |
| `mediaMaxMb` | `number` | Max media size (MB) | - |
| `subagents.maxConcurrent` | `number` | Max sub-agent đồng thời | `1` |
| `subagents.archiveAfterMinutes` | `number` | Thời gian archive sub-agent | `60` |
| `subagents.model` | `string \| {primary, fallbacks[]}` | Model cho sub-agents | - |
| `subagents.thinking` | `string` | Thinking level cho sub-agents | - |
| `sandbox` | `object` | Cấu hình sandbox (mode, workspaceAccess, docker, browser) | - |
| `contextPruning` | `object` | Cấu hình prune old tool results (mode, ttl, keepLastAssistants) | - |
| `compaction` | `object` | Cấu hình compaction (mode, reserveTokensFloor, memoryFlush) | - |
| `cliBackends` | `Record<string, CliBackendConfig>` | CLI backends cho text-only fallback | - |
| `tools` | `object` | Cấu hình tool cho agent | - |

### 3.2 Agent List (`agents.list[]`)

Danh sách các agent cụ thể. Mỗi agent kế thừa từ defaults và có thể override.

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|--------|
| `id` | `string` | **Có** | ID duy nhất (lowercase, a-z0-9, dấu gạch ngang, max 64 ký tự) |
| `default` | `boolean` | Không | Đánh dấu agent mặc định |
| `name` | `string` | Không | Tên hiển thị |
| `workspace` | `string` | Không | Thư mục workspace riêng |
| `agentDir` | `string` | Không | Thư mục agent tùy chỉnh |
| `model` | `string \| {primary, fallbacks[]}` | Không | Override model AI |
| `skills` | `string[]` | Không | Allowlist skills |
| `memorySearch` | `object` | Không | Cấu hình memory search |
| `humanDelay` | `object` | Không | Cấu hình delay giả lập |
| `heartbeat` | `object` | Không | Cấu hình heartbeat |
| `identity` | `object` | Không | Cấu hình danh tính |
| `groupChat` | `object` | Không | Cấu hình chat nhóm |
| `subagents.allowAgents` | `string[]` | Không | Agents được phép spawn (`"*"` = tất cả) |
| `subagents.model` | `string \| {primary, fallbacks[]}` | Không | Model cho sub-agents |
| `sandbox` | `object` | Không | Cấu hình sandbox |
| `tools` | `object` | Không | Cấu hình tool |

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-20250514",
      "subagents": {
        "archiveAfterMinutes": 120
      }
    },
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Assistant",
        "model": {
          "primary": "anthropic/claude-opus-4-20250514",
          "fallbacks": ["anthropic/claude-sonnet-4-20250514"]
        },
        "skills": ["github", "canvas"],
        "subagents": {
          "allowAgents": ["coder", "writer"]
        }
      },
      {
        "id": "coder",
        "name": "Code Assistant",
        "workspace": "/home/user/projects",
        "model": "anthropic/claude-sonnet-4-20250514",
        "skills": ["github"]
      },
      {
        "id": "writer",
        "name": "Content Writer",
        "model": "anthropic/claude-sonnet-4-20250514"
      }
    ]
  }
}
```

---

## 4. Models

Cấu hình nhà cung cấp model AI và profiles.

### 4.1 Providers (`models.providers`)

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `[provider_id].apiKey` | `string` | API key |
| `[provider_id].baseUrl` | `string` | Base URL cho API |
| `[provider_id].headers` | `Record<string, string>` | HTTP headers tùy chỉnh |
| `[provider_id].models[]` | `array` | Danh sách model khả dụng |
| `[provider_id].models[].id` | `string` | Model ID |
| `[provider_id].models[].cost` | `object` | Chi phí (input, output, cacheRead, cacheWrite) |

### 4.2 Profiles (`models.profiles[]`)

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `id` | `string` | Profile ID |
| `default` | `boolean` | Profile mặc định |
| `provider` | `string` | Provider ID |
| `model` | `string` | Model ID |

```json
{
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": "sk-ant-...",
        "models": [
          {
            "id": "claude-opus-4-20250514",
            "cost": { "input": 15, "output": 75, "cacheRead": 1.5, "cacheWrite": 18.75 }
          }
        ]
      },
      "openai": {
        "apiKey": "sk-...",
        "baseUrl": "https://api.openai.com/v1"
      }
    },
    "profiles": [
      { "id": "default", "default": true, "provider": "anthropic", "model": "claude-sonnet-4-20250514" },
      { "id": "smart", "provider": "anthropic", "model": "claude-opus-4-20250514" }
    ]
  }
}
```

---

## 5. Tools

Cấu hình hệ thống tool - quyền truy cập, chính sách, và các công cụ chuyên biệt.

### 5.1 Tool Policy

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `tools.profile` | `"minimal" \| "coding" \| "messaging" \| "full"` | Profile tool preset | `"coding"` |
| `tools.allow` | `string[]` | Danh sách tool được phép | - |
| `tools.alsoAllow` | `string[]` | Thêm tool vào danh sách cho phép | - |
| `tools.deny` | `string[]` | Danh sách tool bị cấm | - |

### 5.2 Exec Tool (`tools.exec`)

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `host` | `"sandbox" \| "gateway" \| "node"` | Nơi thực thi lệnh | `"gateway"` |
| `security` | `"deny" \| "allowlist" \| "full"` | Mức bảo mật | `"allowlist"` |
| `ask` | `"off" \| "on-miss" \| "always"` | Hỏi xác nhận | `"on-miss"` |
| `node` | `string` | Path tới Node.js binary | - |
| `pathPrepend` | `string[]` | Thêm vào đầu $PATH | - |
| `safeBins` | `string[]` | Lệnh an toàn (không cần confirm) | - |
| `backgroundMs` | `number` | Thời gian chạy background (ms) | - |
| `timeoutSec` | `number` | Timeout cho lệnh (giây) | - |
| `approvalRunningNoticeMs` | `number` | Delay thông báo chờ approval (ms) | - |
| `applyPatch.enabled` | `boolean` | Bật tính năng apply patch | - |
| `applyPatch.allowModels` | `string[]` | Models được phép dùng apply_patch | - |

### 5.3 Media Tools (`tools.mediaTools`)

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `models` | `array` | Danh sách model phân tích media |
| `concurrency` | `number` | Số task media đồng thời |
| `image` | `object` | Cấu hình phân tích ảnh |
| `audio` | `object` | Cấu hình phân tích âm thanh |
| `video` | `object` | Cấu hình phân tích video |

### 5.4 Link Tools (`tools.linkTools`)

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `enabled` | `boolean` | Bật link tools | `true` |
| `maxLinks` | `number` | Số link tối đa xử lý | - |
| `models` | `array` | Model dùng cho link analysis | - |

```json
{
  "tools": {
    "profile": "coding",
    "alsoAllow": ["web_search", "web_fetch"],
    "deny": ["browser"],
    "exec": {
      "security": "allowlist",
      "safeBins": ["git", "npm", "node", "python3", "ls", "cat"],
      "timeoutSec": 120,
      "backgroundMs": 60000
    }
  }
}
```

---

## 6. Channels

Cấu hình các kênh giao tiếp. Mỗi kênh có cấu hình riêng.

### Các kênh hỗ trợ

#### Kênh tích hợp (Core)

| Kênh | Key | Thư viện nền tảng |
|------|-----|-------------------|
| Discord | `channels.discord` | discord.js |
| Google Chat | `channels.googlechat` | - |
| iMessage | `channels.imessage` | BlueBubbles |
| Signal | `channels.signal` | - |
| Slack | `channels.slack` | Bolt |
| Telegram | `channels.telegram` | grammY |
| WhatsApp | `channels.whatsapp` | Baileys |

#### Kênh mở rộng (Extensions)

| Kênh | Extension | Mô tả |
|------|-----------|--------|
| IRC | `extensions/irc` | Internet Relay Chat |
| Matrix | `extensions/matrix` | Matrix messaging protocol |
| Microsoft Teams | `extensions/msteams` | Microsoft Teams |
| Nostr | `extensions/nostr` | Nostr protocol |
| Tlon/Urbit | `extensions/tlon` | Urbit messaging |
| Twitch | `extensions/twitch` | Twitch streaming platform |
| Voice Call | `extensions/voice-call` | Voice call capability |
| Zalo | `extensions/zalo` | Zalo OA messaging |

### Ví dụ cấu hình WhatsApp

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "accounts": [
        {
          "id": "main",
          "phoneNumber": "+84...",
          "ownerNumbers": ["+84..."]
        }
      ]
    }
  }
}
```

### Cấu hình chi tiết Telegram (`channels.telegram`)

Telegram là một trong những kênh giao tiếp chính của OpenClaw, sử dụng thư viện [grammY](https://grammy.dev/) làm nền tảng. Hỗ trợ đầy đủ: DM, group chat, forum topics, inline buttons, reactions, streaming, webhook, multi-account.

#### 6.1 Cài đặt cơ bản (Account Identity)

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `name` | `string` | - | Tên hiển thị cho account (dùng trong CLI/UI) |
| `enabled` | `boolean` | `true` | Bật/tắt account Telegram này |
| `botToken` | `string` | - | Bot token từ BotFather (nhạy cảm) |
| `tokenFile` | `string` | - | Đường dẫn file chứa bot token (symlinks bị từ chối) |

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "123456:ABC-DEF..."
    }
  }
}
```

#### 6.2 Chính sách DM và Group

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `dmPolicy` | `"pairing" \| "allowlist" \| "open" \| "disabled"` | `"pairing"` | Chính sách xử lý tin nhắn DM |
| `groupPolicy` | `"open" \| "disabled" \| "allowlist"` | `"allowlist"` | Chính sách xử lý tin nhắn group |
| `allowFrom` | `Array<string \| number>` | - | DM allowlist (Telegram user ID hoặc `"*"`) |
| `groupAllowFrom` | `Array<string \| number>` | - | Group sender allowlist (Telegram user ID) |
| `defaultTo` | `string \| number` | - | Target mặc định cho CLI `--deliver` |

**Các giá trị `dmPolicy`:**
- `"pairing"` (mặc định) - Người lạ nhận mã pairing, owner phải approve
- `"allowlist"` - Chỉ cho phép sender trong `allowFrom`
- `"open"` - Cho phép tất cả DM (yêu cầu `allowFrom` chứa `"*"`)
- `"disabled"` - Bỏ qua tất cả DM

**Các giá trị `groupPolicy`:**
- `"open"` - Cho phép tất cả group, chỉ áp dụng mention-gating
- `"allowlist"` - Chỉ cho phép group sender trong allowlist
- `"disabled"` - Chặn tất cả tin nhắn group

```json
{
  "channels": {
    "telegram": {
      "botToken": "123456:ABC-...",
      "dmPolicy": "open",
      "allowFrom": ["*"],
      "groupPolicy": "allowlist",
      "groupAllowFrom": [123456789, 987654321]
    }
  }
}
```

#### 6.3 Webhook

Mặc định Telegram dùng long-polling. Để dùng webhook (hiệu suất cao hơn, phù hợp production):

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `webhookUrl` | `string` | - | URL webhook công khai (HTTPS, phải truy cập được từ internet) |
| `webhookSecret` | `string` | - | Secret token xác thực webhook |
| `webhookPath` | `string` | `/telegram-webhook` | Đường dẫn route webhook trên gateway |
| `webhookHost` | `string` | `127.0.0.1` | Host bind cho webhook listener |
| `webhookPort` | `number` | `8787` | Port bind cho webhook listener (0 = OS tự chọn) |
| `webhookCertPath` | `string` | - | Đường dẫn chứng chỉ self-signed (PEM) để upload lên Telegram |

```json
{
  "channels": {
    "telegram": {
      "botToken": "123456:ABC-...",
      "webhookUrl": "https://example.com/telegram",
      "webhookSecret": "my-webhook-secret",
      "webhookPort": 8443
    }
  }
}
```

#### 6.4 Streaming và hiển thị

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `streaming` | `"off" \| "partial" \| "block" \| "progress"` | `"off"` | Chế độ stream preview |
| `blockStreaming` | `boolean` | - | Tắt block streaming cho account này |
| `blockStreamingCoalesce` | `object` | - | Gộp block reply trước khi gửi |
| `linkPreview` | `boolean` | `true` | Hiển thị link preview trong tin nhắn |
| `markdown` | `object` | - | Tùy chỉnh markdown (tables: `"off"` / `"bullets"` / `"code"`) |
| `textChunkLimit` | `number` | `4000` | Kích thước chunk text tối đa (ký tự) |
| `chunkMode` | `"length" \| "newline"` | `"length"` | Cách chia chunk: theo kích thước hoặc newline |

**Các giá trị `streaming`:**
- `"off"` - Không có preview, chờ phản hồi hoàn chỉnh
- `"partial"` - Edit preview message liên tục (phổ biến nhất)
- `"block"` - Stream theo block lớn
- `"progress"` - Alias, tương đương `"partial"` trên Telegram

```json
{
  "channels": {
    "telegram": {
      "botToken": "...",
      "streaming": "partial",
      "textChunkLimit": 4000,
      "linkPreview": false,
      "markdown": { "tables": "code" }
    }
  }
}
```

#### 6.5 Reactions và thông báo

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `reactionLevel` | `"off" \| "ack" \| "minimal" \| "extensive"` | `"ack"` | Mức độ reaction của agent |
| `reactionNotifications` | `"off" \| "own" \| "all"` | `"off"` | Reaction nào trigger thông báo |
| `ackReaction` | `string` | - | Emoji ack tùy chỉnh (vd: `"👀"`) |
| `silentErrorReplies` | `boolean` | `false` | Gửi error reply im lặng (không notification sound) |
| `responsePrefix` | `string` | - | Prefix cho response (`""` = tắt, `"auto"` = `[{identity.name}]`) |

**Các giá trị `reactionLevel`:**
- `"off"` - Agent không thể react
- `"ack"` - Gửi reaction xác nhận (👀 khi đang xử lý)
- `"minimal"` - React thận trọng (1 per 5-10 exchanges)
- `"extensive"` - React tự do khi phù hợp

#### 6.6 Cấu hình Group (`groups`)

Mỗi group được cấu hình riêng theo group ID (số âm cho supergroup):

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `requireMention` | `boolean` | - | Yêu cầu @mention để trigger bot |
| `groupPolicy` | `"open" \| "disabled" \| "allowlist"` | - | Override policy cho group này |
| `enabled` | `boolean` | `true` | Bật/tắt bot cho group này |
| `allowFrom` | `Array<string \| number>` | - | Allowlist sender cho group |
| `systemPrompt` | `string` | - | System prompt bổ sung cho group |
| `skills` | `string[]` | - | Skills cho group (omit = tất cả, `[]` = không) |
| `tools` | `object` | - | Tool policy override (allow/deny/alsoAllow) |
| `toolsBySender` | `Record<string, ToolPolicy>` | - | Tool policy theo sender |
| `disableAudioPreflight` | `boolean` | - | Bỏ qua transcription voice note cho mention detection |
| `topics` | `Record<string, TopicConfig>` | - | Cấu hình per-topic (key = message_thread_id) |

```json
{
  "channels": {
    "telegram": {
      "botToken": "...",
      "groups": {
        "-1001234567890": {
          "requireMention": true,
          "groupPolicy": "open",
          "systemPrompt": "You are a helpful coding assistant for this group.",
          "skills": ["github", "web-search"],
          "tools": {
            "deny": ["gateway", "cron"]
          }
        }
      }
    }
  }
}
```

#### 6.7 Cấu hình Topic (Forum)

Mỗi topic trong group/DM có thể cấu hình riêng:

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `agentId` | `string` | - | Route topic tới agent cụ thể |
| `requireMention` | `boolean` | - | Yêu cầu @mention |
| `enabled` | `boolean` | `true` | Bật/tắt topic |
| `allowFrom` | `Array<string \| number>` | - | Sender allowlist |
| `systemPrompt` | `string` | - | System prompt cho topic |
| `skills` | `string[]` | - | Skills cho topic |
| `groupPolicy` | `"open" \| "disabled" \| "allowlist"` | - | Override policy |
| `disableAudioPreflight` | `boolean` | - | Bỏ qua voice note transcription |

```json
{
  "channels": {
    "telegram": {
      "botToken": "...",
      "groups": {
        "-1001234567890": {
          "requireMention": true,
          "topics": {
            "1": {
              "agentId": "coder",
              "systemPrompt": "Focus on code review",
              "skills": ["github"],
              "requireMention": false
            },
            "42": {
              "agentId": "writer",
              "systemPrompt": "Help with documentation",
              "enabled": true
            }
          }
        }
      }
    }
  }
}
```

#### 6.8 Cấu hình DM riêng (`direct`)

Override cấu hình cho DM cụ thể:

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `dmPolicy` | `"pairing" \| "allowlist" \| "open" \| "disabled"` | - | Override DM policy |
| `tools` | `object` | - | Tool policy override |
| `toolsBySender` | `Record<string, ToolPolicy>` | - | Per-sender tool policy |
| `skills` | `string[]` | - | Skills cho DM |
| `topics` | `Record<string, TopicConfig>` | - | Per-topic config |
| `enabled` | `boolean` | `true` | Bật/tắt DM |
| `requireTopic` | `boolean` | - | Yêu cầu message phải từ topic |
| `allowFrom` | `Array<string \| number>` | - | Sender allowlist |
| `systemPrompt` | `string` | - | System prompt bổ sung |

#### 6.9 Actions (Tool Gating)

Kiểm soát từng action mà bot có thể thực hiện:

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `actions.reactions` | `boolean` | `true` | Cho phép emoji reactions |
| `actions.sendMessage` | `boolean` | `true` | Cho phép gửi tin nhắn |
| `actions.poll` | `boolean` | `true` | Cho phép tạo poll (cần sendMessage=true) |
| `actions.deleteMessage` | `boolean` | `true` | Cho phép xóa tin nhắn |
| `actions.editMessage` | `boolean` | `true` | Cho phép sửa tin nhắn |
| `actions.sticker` | `boolean` | `true` | Cho phép sticker |
| `actions.createForumTopic` | `boolean` | `true` | Cho phép tạo forum topic |
| `actions.editForumTopic` | `boolean` | `true` | Cho phép sửa forum topic |

#### 6.10 Exec Approvals (Telegram-native)

Cho phép approve lệnh exec qua Telegram:

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `execApprovals.enabled` | `boolean` | `false` | Bật exec approval qua Telegram |
| `execApprovals.approvers` | `Array<string \| number>` | - | User ID được phép approve (bắt buộc nếu enabled) |
| `execApprovals.agentFilter` | `string[]` | - | Chỉ forward approval cho agent IDs này |
| `execApprovals.sessionFilter` | `string[]` | - | Lọc theo session key pattern |
| `execApprovals.target` | `"dm" \| "channel" \| "both"` | `"dm"` | Nơi gửi approval prompt |

#### 6.11 Network và Retry

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `timeoutSeconds` | `number` | - | Timeout cho Telegram API client (giây) |
| `proxy` | `string` | - | HTTP/HTTPS proxy URL |
| `mediaMaxMb` | `number` | - | Kích thước media tối đa (MB) |
| `retry.attempts` | `number` | `3` | Số lần retry tối đa |
| `retry.minDelayMs` | `number` | ~400 | Delay tối thiểu giữa retry (ms) |
| `retry.maxDelayMs` | `number` | `30000` | Delay tối đa (ms) |
| `retry.jitter` | `number` | `0.1` | Jitter factor (0-1) |
| `network.autoSelectFamily` | `boolean` | - | Override Node autoSelectFamily |
| `network.dnsResultOrder` | `"ipv4first" \| "verbatim"` | `"ipv4first"` | Thứ tự DNS resolution |

#### 6.12 Thread Bindings

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `threadBindings.enabled` | `boolean` | - | Bật thread-bound session routing |
| `threadBindings.idleHours` | `number` | - | Giờ idle trước khi binding hết hạn |
| `threadBindings.maxAgeHours` | `number` | - | Tuổi tối đa của binding (giờ) |
| `threadBindings.spawnSubagentSessions` | `boolean` | - | Cho phép spawn subagent trong thread |
| `threadBindings.spawnAcpSessions` | `boolean` | - | Cho phép spawn ACP sessions trong thread |

#### 6.13 Health Monitoring

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `heartbeat.showOk` | `boolean` | `false` | Hiển thị HEARTBEAT_OK trong chat |
| `heartbeat.showAlerts` | `boolean` | `true` | Hiển thị heartbeat alerts |
| `heartbeat.useIndicator` | `boolean` | `true` | Emit indicator events cho UI |
| `healthMonitor.enabled` | `boolean` | - | Bật channel-health-monitor restarts |

#### 6.14 Multi-Account

Hỗ trợ nhiều bot token trên cùng gateway:

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `accounts` | `Record<string, AccountConfig>` | - | Cấu hình per-account |
| `defaultAccount` | `string` | - | Account mặc định khi có nhiều account |

```json
{
  "channels": {
    "telegram": {
      "defaultAccount": "main",
      "accounts": {
        "main": {
          "name": "Production Bot",
          "botToken": "111:AAA...",
          "dmPolicy": "open",
          "allowFrom": ["*"]
        },
        "dev": {
          "name": "Dev Bot",
          "botToken": "222:BBB...",
          "dmPolicy": "allowlist",
          "allowFrom": [123456789]
        }
      }
    }
  }
}
```

#### 6.15 Ví dụ cấu hình Telegram đầy đủ

```json
{
  "channels": {
    "telegram": {
      "botToken": "123456:ABC-DEF...",
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "streaming": "partial",
      "textChunkLimit": 4000,
      "linkPreview": false,
      "reactionLevel": "ack",
      "reactionNotifications": "own",
      "silentErrorReplies": false,
      "configWrites": true,
      "timeoutSeconds": 30,
      "retry": {
        "attempts": 3,
        "maxDelayMs": 30000
      },
      "actions": {
        "reactions": true,
        "sendMessage": true,
        "poll": true,
        "sticker": true,
        "createForumTopic": true
      },
      "groups": {
        "-1001234567890": {
          "requireMention": true,
          "groupPolicy": "open",
          "systemPrompt": "You are a coding assistant.",
          "skills": ["github"],
          "topics": {
            "1": {
              "agentId": "coder",
              "requireMention": false
            }
          }
        }
      },
      "execApprovals": {
        "enabled": true,
        "approvers": [123456789],
        "target": "dm"
      },
      "threadBindings": {
        "enabled": true,
        "idleHours": 24,
        "maxAgeHours": 168
      },
      "heartbeat": {
        "showOk": false,
        "showAlerts": true
      }
    }
  }
}
```

#### 6.16 Thứ tự kế thừa cấu hình (Inheritance)

```
Top-level settings (channels.telegram.*)
    │
    ├── Account settings (accounts.<id>.*)
    │       │
    │       ├── Group settings (groups.<group_id>.*)
    │       │       │
    │       │       └── Topic settings (topics.<thread_id>.*)
    │       │
    │       └── Direct/DM settings (direct.<chat_id>.*)
    │               │
    │               └── DM Topic settings (topics.<thread_id>.*)
    │
    └── Per-DM legacy (dms.<user_id>.*)
```

Cấu hình cấp thấp hơn override cấu hình cấp cao hơn (shallow merge).

---

### Cấu hình chi tiết Zalo (`channels.zalo`)

Zalo là kênh messaging phổ biến tại Việt Nam, tích hợp qua Zalo Bot API. Plugin Zalo hỗ trợ: DM, gửi ảnh, webhook, multi-account, pairing, proxy.

> **Cài đặt:** `openclaw plugins install @openclaw/zalo`

#### Khả năng (Capabilities)

| Tính năng | Hỗ trợ | Ghi chú |
|-----------|--------|---------|
| DM (Direct Message) | Có | Chat type chính |
| Group chat | Không | Chỉ hỗ trợ DM |
| Media (ảnh) | Có | Gửi/nhận ảnh |
| Reactions | Không | Zalo Bot API không hỗ trợ |
| Threads | Không | - |
| Polls | Không | - |
| Block Streaming | Có | Gửi streaming theo block |
| Native Commands | Không | - |

#### Tham số cấu hình Account

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `name` | `string` | - | Tên hiển thị cho account (CLI/UI) |
| `enabled` | `boolean` | `true` | Bật/tắt account |
| `botToken` | `string` | - | Bot token từ Zalo Bot Creator (nhạy cảm) |
| `tokenFile` | `string` | - | Đường dẫn file chứa bot token |
| `dmPolicy` | `"pairing" \| "allowlist" \| "open" \| "disabled"` | `"pairing"` | Chính sách DM |
| `allowFrom` | `Array<string \| number>` | - | Allowlist Zalo user ID |
| `mediaMaxMb` | `number` | `5` | Kích thước media tối đa (MB) |
| `proxy` | `string` | - | HTTP/HTTPS proxy URL cho API requests |
| `responsePrefix` | `string` | - | Prefix cho outbound response |
| `markdown` | `object` | - | Tùy chỉnh markdown (tables: `"off"` / `"bullets"` / `"code"`) |

**Các giá trị `dmPolicy`:**
- `"pairing"` (mặc định) - Người lạ nhận mã pairing code, owner approve qua CLI
- `"allowlist"` - Chỉ cho phép Zalo user ID trong `allowFrom`
- `"open"` - Cho phép tất cả DM (yêu cầu `allowFrom` chứa `"*"`)
- `"disabled"` - Bỏ qua tất cả DM

**Biến môi trường:** `ZALO_BOT_TOKEN` — chỉ dùng được cho account mặc định.

#### Webhook

Mặc định dùng long-polling. Chuyển sang webhook cho production:

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `webhookUrl` | `string` | - | URL webhook công khai (HTTPS bắt buộc) |
| `webhookSecret` | `string` | - | Secret token xác thực (8-256 ký tự) |
| `webhookPath` | `string` | Lấy từ webhookUrl | Đường dẫn route webhook trên gateway |

**Yêu cầu webhook:**
- `webhookUrl` phải bắt đầu bằng `https://`
- `webhookSecret` phải dài 8-256 ký tự
- Nếu `webhookPath` bỏ trống, tự động lấy từ `webhookUrl`

#### Multi-Account

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|--------|
| `accounts` | `Record<string, AccountConfig>` | - | Cấu hình per-account |
| `defaultAccount` | `string` | - | Account mặc định |

#### Giới hạn kỹ thuật

| Giới hạn | Giá trị | Ghi chú |
|----------|---------|---------|
| Text chunk limit | 2000 ký tự | Tin nhắn dài hơn tự động chia chunk |
| Media max | 5 MB (mặc định) | Cấu hình qua `mediaMaxMb` |
| Webhook payload | 1 MB | Body tối đa cho webhook request |

#### Ví dụ cấu hình cơ bản

```json
{
  "channels": {
    "zalo": {
      "enabled": true,
      "botToken": "your-zalo-bot-token",
      "dmPolicy": "pairing"
    }
  }
}
```

#### Ví dụ cấu hình webhook

```json
{
  "channels": {
    "zalo": {
      "botToken": "your-zalo-bot-token",
      "webhookUrl": "https://example.com/zalo-webhook",
      "webhookSecret": "my-secret-8-chars-min",
      "webhookPath": "/zalo-webhook"
    }
  }
}
```

#### Ví dụ multi-account

```json
{
  "channels": {
    "zalo": {
      "defaultAccount": "main",
      "accounts": {
        "main": {
          "name": "Bot chính",
          "botToken": "token-1",
          "dmPolicy": "open",
          "allowFrom": ["*"]
        },
        "support": {
          "name": "Bot hỗ trợ",
          "botToken": "token-2",
          "dmPolicy": "allowlist",
          "allowFrom": ["123456789"]
        }
      }
    }
  }
}
```

#### Ví dụ cấu hình đầy đủ

```json
{
  "channels": {
    "zalo": {
      "enabled": true,
      "botToken": "your-zalo-bot-token",
      "dmPolicy": "pairing",
      "allowFrom": [123456789, 987654321],
      "mediaMaxMb": 10,
      "proxy": "http://proxy.local:8080",
      "responsePrefix": "[Bot]",
      "markdown": { "tables": "code" },
      "webhookUrl": "https://example.com/zalo-webhook",
      "webhookSecret": "my-secret-token-here",
      "webhookPath": "/zalo-webhook"
    }
  }
}
```

#### Sự kiện Zalo được xử lý

| Event Name | Mô tả |
|------------|--------|
| `message.text.received` | Tin nhắn văn bản |
| `message.image.received` | Tin nhắn ảnh (tải về và xử lý) |
| `message.sticker.received` | Sticker (chỉ log) |
| `message.unsupported.received` | Loại tin nhắn chưa hỗ trợ (chỉ log) |

---

## 7. Session

Cấu hình quản lý phiên làm việc (session).

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `session.scope` | `"per-sender" \| "global"` | Phạm vi session | `"per-sender"` |
| `session.dmScope` | `"main" \| "per-peer" \| "per-channel-peer" \| "per-account-channel-peer"` | Phạm vi DM session | `"main"` |
| `session.typingMode` | `"never" \| "instant" \| "thinking" \| "message"` | Chế độ hiển thị typing | `"thinking"` |
| `session.store` | `string` | Đường dẫn lưu trữ session | - |
| `session.mainKey` | `string` | Key session chính | `"main"` |
| `session.reset.mode` | `"daily" \| "idle"` | Chế độ reset session | - |
| `session.reset.atHour` | `number` | Giờ reset (0-23, dùng với `daily`) | `0` |
| `session.reset.idleMinutes` | `number` | Phút idle trước khi reset | - |
| `session.resetByChannel` | `Record<string, ResetConfig>` | Override reset theo channel | - |
| `session.sendPolicy.default` | `"allow" \| "deny"` | Chính sách gửi mặc định | `"allow"` |
| `session.sendPolicy.rules[]` | `array` | Quy tắc gửi tin nhắn | - |

```json
{
  "session": {
    "scope": "per-sender",
    "dmScope": "per-peer",
    "typingMode": "thinking",
    "reset": {
      "mode": "idle",
      "idleMinutes": 120
    },
    "resetByChannel": {
      "whatsapp": { "mode": "daily", "atHour": 6 }
    },
    "sendPolicy": {
      "default": "allow",
      "rules": [
        { "action": "deny", "match": { "channel": "slack", "chatType": "group" } }
      ]
    }
  }
}
```

---

## 8. Plugins

Cấu hình hệ thống plugin.

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `plugins.enabled` | `boolean` | Bật/tắt plugins | `true` |
| `plugins.allow` | `string[]` | Allowlist plugin | - |
| `plugins.deny` | `string[]` | Denylist plugin | - |
| `plugins.load.paths` | `string[]` | Đường dẫn load plugin | - |
| `plugins.slots.memory` | `string` | Plugin memory slot | - |
| `plugins.entries` | `Record<string, PluginEntryConfig>` | Cấu hình từng plugin | - |
| `plugins.installs` | `Record<string, PluginInstallRecord>` | Bản ghi cài đặt | - |

```json
{
  "plugins": {
    "enabled": true,
    "load": {
      "paths": ["./my-plugins"]
    },
    "entries": {
      "my-plugin": {
        "enabled": true,
        "env": { "PLUGIN_KEY": "value" }
      }
    }
  }
}
```

---

## 9. Hooks

Cấu hình webhook và hook nội bộ.

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `hooks.enabled` | `boolean` | Bật/tắt hooks | `false` |
| `hooks.path` | `string` | Đường dẫn webhook | `"/hooks"` |
| `hooks.token` | `string` | Bearer token xác thực | - |
| `hooks.maxBodyBytes` | `number` | Kích thước body tối đa | - |
| `hooks.presets` | `string[]` | Preset hooks | - |
| `hooks.transformsDir` | `string` | Thư mục transform modules | - |

### Hook Mappings (`hooks.mappings[]`)

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `id` | `string` | ID mapping |
| `match.path` | `string` | Path pattern để match |
| `match.source` | `string` | Source filter |
| `action` | `"wake" \| "agent"` | Hành động khi match |
| `wakeMode` | `"now" \| "next-heartbeat"` | Chế độ wake |
| `sessionKey` | `string` | Session key mục tiêu |
| `messageTemplate` | `string` | Template nội dung tin nhắn |
| `textTemplate` | `string` | Template văn bản |
| `deliver` | `boolean` | Có gửi tới channel không |
| `channel` | `string` | Channel mục tiêu |
| `to` | `string` | Người nhận |
| `model` | `string` | Override model |
| `thinking` | `string` | Mức thinking |
| `timeoutSeconds` | `number` | Timeout (giây) |
| `transform.module` | `string` | Transform module path |

### Gmail Hook (`hooks.gmail`)

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `account` | `string` | Gmail account |
| `label` | `string` | Gmail label filter |
| `topic` | `string` | Pub/Sub topic |
| `subscription` | `string` | Pub/Sub subscription |
| `pushToken` | `string` | Push notification token |
| `hookUrl` | `string` | Webhook URL |
| `includeBody` | `boolean` | Bao gồm body email |
| `model` | `string` | Model xử lý |
| `thinking` | `"off" \| "minimal" \| "low" \| "medium" \| "high"` | Mức thinking |

```json
{
  "hooks": {
    "enabled": true,
    "path": "/hooks",
    "token": "my-secret-token",
    "mappings": [
      {
        "id": "github-webhook",
        "match": { "path": "/github" },
        "action": "agent",
        "sessionKey": "github-events",
        "messageTemplate": "New GitHub event: {{body.action}} on {{body.repository.full_name}}"
      }
    ]
  }
}
```

---

## 10. Memory

Cấu hình hệ thống bộ nhớ (memory).

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `memory.backend` | `"builtin" \| "qmd"` | Backend bộ nhớ | `"builtin"` |
| `memory.citations` | `"auto" \| "on" \| "off"` | Hiển thị citations | `"auto"` |

### QMD Backend (`memory.qmd`)

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `command` | `string` | Lệnh QMD |
| `includeDefaultMemory` | `boolean` | Bao gồm memory mặc định |
| `paths[].path` | `string` | Đường dẫn tài liệu |
| `paths[].name` | `string` | Tên hiển thị |
| `paths[].pattern` | `string` | Pattern filter file |
| `sessions.enabled` | `boolean` | Bật session export |
| `sessions.exportDir` | `string` | Thư mục export |
| `sessions.retentionDays` | `number` | Số ngày giữ lại |
| `update.interval` | `string` | Khoảng cách cập nhật |
| `update.debounceMs` | `number` | Debounce (ms) |
| `update.onBoot` | `boolean` | Cập nhật khi khởi động |
| `update.embedInterval` | `string` | Khoảng cách embedding |
| `limits.maxResults` | `number` | Số kết quả tối đa |
| `limits.maxSnippetChars` | `number` | Độ dài snippet tối đa |
| `limits.maxInjectedChars` | `number` | Ký tự inject tối đa |
| `limits.timeoutMs` | `number` | Timeout (ms) |

```json
{
  "memory": {
    "backend": "qmd",
    "citations": "auto",
    "qmd": {
      "paths": [
        { "path": "./docs", "name": "Documentation", "pattern": "**/*.md" },
        { "path": "./notes", "name": "Notes" }
      ],
      "update": {
        "onBoot": true,
        "interval": "1h"
      },
      "limits": {
        "maxResults": 10,
        "maxSnippetChars": 2000
      }
    }
  }
}
```

---

## 11. Browser

Cấu hình trình duyệt tích hợp.

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `browser.enabled` | `boolean` | Bật/tắt browser tool | `false` |
| `browser.snapshotDefaults.mode` | `"efficient"` | Chế độ snapshot | `"efficient"` |
| `browser.cacheDir` | `string` | Thư mục cache | - |

```json
{
  "browser": {
    "enabled": true,
    "snapshotDefaults": { "mode": "efficient" },
    "cacheDir": "/tmp/openclaw-browser-cache"
  }
}
```

---

## 12. Gateway

Cấu hình gateway server (WebSocket + HTTP + TLS).

### 12.1 Cài đặt cơ bản

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `gateway.port` | `number` | Cổng server | `18789` |
| `gateway.mode` | `"local" \| "remote"` | Chế độ gateway | `"local"` |
| `gateway.bind` | `"auto" \| "lan" \| "loopback" \| "custom" \| "tailnet"` | Chế độ bind | `"auto"` |
| `gateway.customBindHost` | `string` | Host bind tùy chỉnh (khi bind=custom) | - |

### 12.2 Authentication

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `gateway.auth.mode` | `"token" \| "password" \| "trusted-proxy"` | Chế độ xác thực |
| `gateway.auth.token` | `string` | Token xác thực |
| `gateway.auth.password` | `string` | Mật khẩu truy cập |
| `gateway.auth.allowTailscale` | `boolean` | Cho phép Tailscale auth |
| `gateway.auth.rateLimit` | `object` | Rate limiting cho auth failures |
| `gateway.auth.trustedProxy` | `object` | Cấu hình trusted proxy auth |

### 12.3 TLS

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `gateway.tls.enabled` | `boolean` | Bật TLS |
| `gateway.tls.autoGenerate` | `boolean` | Tự tạo self-signed cert |
| `gateway.tls.certPath` | `string` | Đường dẫn certificate |
| `gateway.tls.keyPath` | `string` | Đường dẫn private key |
| `gateway.tls.caPath` | `string` | Đường dẫn CA certificate |

### 12.4 HTTP Endpoints

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `gateway.http.chatCompletions.enabled` | `boolean` | Bật `/v1/chat/completions` (OpenAI-compatible) |
| `gateway.http.chatCompletions.images` | `object` | Cấu hình URL fetch cho images |
| `gateway.http.responses.enabled` | `boolean` | Bật `/v1/responses` (OpenResponses API) |
| `gateway.http.responses.files` | `object` | Cấu hình file input |
| `gateway.http.responses.images` | `object` | Cấu hình image input (PDF support) |

### 12.5 Control UI, Remote, Tailscale

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `gateway.controlUi.enabled` | `boolean` | Bật web dashboard |
| `gateway.controlUi.basePath` | `string` | UI path prefix |
| `gateway.remote.url` | `string` | URL remote gateway |
| `gateway.remote.transport` | `string` | Transport mode (ws, ssh) |
| `gateway.remote.tlsFingerprint` | `string` | TLS fingerprint verification |
| `gateway.tailscale.mode` | `"off" \| "serve" \| "funnel"` | Chế độ Tailscale |
| `gateway.reload.mode` | `"hot" \| "restart" \| "hybrid"` | Chế độ reload config |
| `gateway.reload.deferralTimeoutMs` | `number` | Timeout defer in-flight ops |

### 12.6 Nodes

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `gateway.nodes.browser` | `object` | Browser node mode/selection |
| `gateway.nodes.allowCommands` | `string[]` | Lệnh CLI cho phép trên node |
| `gateway.nodes.denyCommands` | `string[]` | Lệnh CLI chặn trên node |

```json
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "auto",
    "auth": {
      "mode": "token",
      "token": "my-gateway-token"
    },
    "tls": {
      "enabled": false
    },
    "http": {
      "chatCompletions": { "enabled": true },
      "responses": { "enabled": true }
    },
    "controlUi": { "enabled": true },
    "tailscale": { "mode": "off" },
    "reload": { "mode": "hot" }
  }
}
```

---

## 13. Sandbox

Cấu hình chạy cách ly (sandbox) cho agent.

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `sandbox.mode` | `"off" \| "non-main" \| "all"` | Chế độ sandbox | `"off"` |
| `sandbox.type` | `"docker"` | Loại sandbox | `"docker"` |
| `sandbox.docker.image` | `string` | Docker image | - |
| `sandbox.docker.bindMounts` | `Record<string, string>` | Volume mounts | - |
| `sandbox.docker.env` | `Record<string, string>` | Biến môi trường trong container | - |
| `sandbox.docker.workspaceRoot` | `string` | Thư mục workspace trong container | - |

```json
{
  "sandbox": {
    "mode": "non-main",
    "type": "docker",
    "docker": {
      "image": "openclaw-sandbox:latest",
      "bindMounts": {
        "/home/user/projects": "/workspace"
      },
      "env": {
        "NODE_ENV": "development"
      },
      "workspaceRoot": "/workspace"
    }
  }
}
```

---

## 14. Logging

Cấu hình hệ thống log.

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `logging.level` | `"silent" \| "fatal" \| "error" \| "warn" \| "info" \| "debug" \| "trace"` | Log level | `"info"` |
| `logging.file` | `string` | File log | - |
| `logging.consoleLevel` | `string` | Log level cho console | - |
| `logging.consoleStyle` | `"pretty" \| "compact" \| "json"` | Kiểu hiển thị console | `"pretty"` |

```json
{
  "logging": {
    "level": "info",
    "file": "/var/log/openclaw/app.log",
    "consoleLevel": "warn",
    "consoleStyle": "pretty"
  }
}
```

---

## Ví dụ file `openclaw.json` đầy đủ

```json
{
  "meta": {
    "lastTouchedVersion": "1.2.0"
  },
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-..."
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-20250514"
    },
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Assistant",
        "model": "anthropic/claude-opus-4-20250514",
        "skills": ["github", "canvas"],
        "subagents": {
          "allowAgents": ["coder", "researcher"]
        }
      },
      {
        "id": "coder",
        "name": "Code Expert",
        "workspace": "/home/user/projects"
      },
      {
        "id": "researcher",
        "name": "Researcher",
        "skills": ["web-research"]
      }
    ]
  },
  "tools": {
    "profile": "coding",
    "exec": {
      "security": "allowlist",
      "safeBins": ["git", "npm", "node"]
    }
  },
  "session": {
    "scope": "per-sender",
    "reset": { "mode": "idle", "idleMinutes": 120 }
  },
  "gateway": {
    "port": 18789,
    "token": "my-token"
  },
  "logging": {
    "level": "info"
  }
}
```

---

## 15. Skills

Cấu hình hệ thống skill (53 built-in skills).

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `skills.allowBundled` | `string[]` | Danh sách bundled skills cho phép | Tất cả |
| `skills.load.extraDirs` | `string[]` | Thư mục skill bổ sung | - |
| `skills.load.watch` | `boolean` | Theo dõi thay đổi skill files | - |
| `skills.load.watchDebounceMs` | `number` | Debounce cho file watch (ms) | - |
| `skills.install.preferBrew` | `boolean` | Ưu tiên Homebrew cho dependencies | - |
| `skills.install.nodeManager` | `"npm" \| "pnpm" \| "yarn" \| "bun"` | Node package manager | - |
| `skills.limits.maxCandidatesPerRoot` | `number` | Max candidates per root | - |
| `skills.limits.maxSkillsLoadedPerSource` | `number` | Max skills per source | - |
| `skills.limits.maxSkillsInPrompt` | `number` | Max skills trong prompt | - |
| `skills.limits.maxSkillsPromptChars` | `number` | Max ký tự skill trong prompt | - |
| `skills.limits.maxSkillFileBytes` | `number` | Max kích thước file SKILL.md | - |
| `skills.entries` | `Record<string, SkillEntry>` | Per-skill config (enabled, apiKey, env, config) | - |

```json
{
  "skills": {
    "allowBundled": ["github", "canvas", "web-research"],
    "install": {
      "nodeManager": "pnpm"
    },
    "entries": {
      "github": {
        "enabled": true,
        "env": { "GITHUB_TOKEN": "ghp_..." }
      }
    }
  }
}
```

---

## 16. Cron

Cấu hình tác vụ lên lịch (scheduled jobs).

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `cron.enabled` | `boolean` | Bật/tắt cron | `false` |
| `cron.store` | `string` | Đường dẫn lưu trữ cron state | - |
| `cron.maxConcurrentRuns` | `number` | Số job chạy đồng thời tối đa | `1` |
| `cron.retry.maxAttempts` | `number` | Số lần retry tối đa | - |
| `cron.retry.backoffMs` | `number[]` | Backoff delays (ms) | - |
| `cron.retry.retryOn` | `string` | Loại lỗi retry | - |
| `cron.webhook` | `string` | Webhook URL thông báo | - |
| `cron.webhookToken` | `string` | Token xác thực webhook | - |
| `cron.sessionRetention` | `string \| false` | Thời gian giữ session (duration string) | - |
| `cron.runLog.maxBytes` | `number` | Max bytes cho run log | - |
| `cron.runLog.keepLines` | `number` | Số dòng giữ lại | - |
| `cron.failureAlert.enabled` | `boolean` | Bật cảnh báo khi thất bại | - |
| `cron.failureAlert.after` | `number` | Số lần thất bại trước khi cảnh báo | - |
| `cron.failureAlert.cooldownMs` | `number` | Cooldown giữa cảnh báo (ms) | - |
| `cron.failureDestination` | `object` | Kênh nhận cảnh báo (channel, to, accountId) | - |

```json
{
  "cron": {
    "enabled": true,
    "maxConcurrentRuns": 2,
    "retry": {
      "maxAttempts": 3,
      "backoffMs": [1000, 5000, 15000]
    },
    "failureAlert": {
      "enabled": true,
      "after": 2
    }
  }
}
```

---

## 17. Talk (TTS)

Cấu hình Text-to-Speech (giọng nói).

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `talk.provider` | `string` | TTS provider |
| `talk.providers` | `Record<string, ProviderConfig>` | Per-provider config |
| `talk.voiceId` | `string` | ID giọng nói |
| `talk.voiceAliases` | `Record<string, string>` | Alias cho voice IDs |
| `talk.modelId` | `string` | Model TTS |
| `talk.outputFormat` | `string` | Định dạng output (mp3, wav, etc.) |
| `talk.apiKey` | `string` | API key (nhạy cảm) |
| `talk.interruptOnSpeech` | `boolean` | Ngắt khi người dùng nói |
| `talk.silenceTimeoutMs` | `number` | Timeout im lặng (ms) |

```json
{
  "talk": {
    "provider": "elevenlabs",
    "voiceId": "voice-123",
    "interruptOnSpeech": true,
    "silenceTimeoutMs": 3000
  }
}
```

---

## 18. Auth

Cấu hình xác thực và API profiles.

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `auth.profiles` | `Record<string, AuthProfile>` | Danh sách profile xác thực |
| `auth.order` | `string[]` | Thứ tự ưu tiên provider |
| `auth.cooldowns.billingBackoffHours` | `number` | Backoff khi billing error |
| `auth.cooldowns.billingMaxHours` | `number` | Max backoff hours |
| `auth.cooldowns.failureWindowHours` | `number` | Cửa sổ theo dõi lỗi |

---

## 19. Discovery

Cấu hình khám phá dịch vụ trên mạng.

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `discovery.wideArea` | `object` | Wide-area DNS-SD discovery |
| `discovery.mdns.mode` | `"off" \| "minimal" \| "full"` | Chế độ mDNS broadcast |

---

## 20. Các section khác

Các section bổ sung trong `openclaw.json`:

| Section | Mô tả |
|---------|--------|
| `ui` | Cài đặt giao diện (seamColor, assistant name/avatar) |
| `audio` | Cấu hình audio |
| `media` | Quản lý media (preserveFilenames) |
| `messages` | Cấu hình tin nhắn (responsePrefix, groupChat, queue, debounce) |
| `commands` | Cấu hình lệnh (useAccessGroups, native commands) |
| `approvals` | Exec approval forwarding config |
| `bindings` | Agent binding rules (route channels → agents) |
| `broadcast` | Broadcast strategy config |
| `web` | Web client (enabled, heartbeatSeconds, reconnect) |
| `canvasHost` | Canvas host server (enabled, root, port, liveReload) |
| `nodeHost` | Node host settings (browserProxy) |
| `diagnostics` | Diagnostics (enabled, flags, otel, cacheTrace) |
| `update` | Update settings (channel: stable/beta/dev, checkOnStart) |
| `wizard` | Wizard run history tracking |
