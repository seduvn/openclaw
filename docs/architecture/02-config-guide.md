# Hướng dẫn cấu hình openclaw.json

File `openclaw.json` là file cấu hình chính của OpenClaw. File này thường nằm tại `~/.openclaw/openclaw.json` hoặc trong thư mục workspace của dự án.

## Mục lục

1. [Meta](#1-meta)
2. [Environment Variables](#2-environment-variables-env)
3. [Agents](#3-agents)
4. [Models](#4-models)
5. [Tools](#5-tools)
6. [Channels](#6-channels)
7. [Session](#7-session)
8. [Plugins](#8-plugins)
9. [Hooks](#9-hooks)
10. [Memory](#10-memory)
11. [Browser](#11-browser)
12. [Gateway](#12-gateway)
13. [Sandbox](#13-sandbox)
14. [Logging](#14-logging)

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

Override biến môi trường cho toàn bộ hệ thống.

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `env.[KEY]` | `string` | Giá trị biến môi trường tùy ý |

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-...",
    "OPENAI_API_KEY": "sk-...",
    "BRAVE_API_KEY": "BSA..."
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
| `skills` | `string[]` | Danh sách skills được phép | `[]` |
| `memorySearch` | `object` | Cấu hình tìm kiếm bộ nhớ | - |
| `humanDelay.mode` | `string` | Chế độ delay giả lập người | - |
| `humanDelay.minMs` | `number` | Delay tối thiểu (ms) | - |
| `humanDelay.maxMs` | `number` | Delay tối đa (ms) | - |
| `heartbeat.trigger` | `string` | Trigger cho heartbeat | - |
| `heartbeat.schedule` | `string` | Lịch heartbeat (cron) | - |
| `identity` | `object` | Cấu hình danh tính agent | - |
| `groupChat` | `object` | Cấu hình chat nhóm | - |
| `sandbox` | `object` | Cấu hình sandbox | - |
| `subagents.archiveAfterMinutes` | `number` | Thời gian archive sub-agent | `60` |
| `subagents.model` | `string \| {primary, fallbacks[]}` | Model cho sub-agents | - |
| `tools` | `object` | Cấu hình tool cho agent | - |
| `concurrency.dm` | `number` | Số request DM đồng thời | - |
| `concurrency.group` | `number` | Số request group đồng thời | - |
| `concurrency.thread` | `number` | Số request thread đồng thời | - |

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

Cấu hình gateway server.

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `gateway.port` | `number` | Cổng WebSocket | `18789` |
| `gateway.bind` | `string` | Địa chỉ bind | `"127.0.0.1"` |
| `gateway.token` | `string` | Token xác thực | - |
| `gateway.password` | `string` | Mật khẩu truy cập | - |

### Remote Access (`gateway.remote`)

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `enabled` | `boolean` | Bật truy cập từ xa |
| `url` | `string` | URL truy cập từ xa |
| `wsPort` | `number` | Cổng WebSocket từ xa |

### Tailscale (`gateway.tailscale`)

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `enabled` | `boolean` | Bật Tailscale |
| `serve.mode` | `string` | Chế độ serve |
| `serve.path` | `string` | Path serve |
| `funnel.enabled` | `boolean` | Bật Tailscale Funnel |

```json
{
  "gateway": {
    "port": 18789,
    "bind": "127.0.0.1",
    "token": "my-gateway-token",
    "remote": {
      "enabled": false
    },
    "tailscale": {
      "enabled": false
    }
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
