# Hướng dẫn thiết lập và sử dụng Sub-Agent

## Mục lục

1. [Tổng quan Sub-Agent](#1-tổng-quan-sub-agent)
2. [Cấu hình Sub-Agent trong openclaw.json](#2-cấu-hình-sub-agent-trong-openclawjson)
3. [Sử dụng sessions_spawn Tool](#3-sử-dụng-sessions_spawn-tool)
4. [Vòng đời Sub-Agent (Lifecycle)](#4-vòng-đời-sub-agent-lifecycle)
5. [Giao tiếp giữa Parent và Sub-Agent](#5-giao-tiếp-giữa-parent-và-sub-agent)
6. [Quản lý Sub-Agent bằng lệnh /subagents](#6-quản-lý-sub-agent-bằng-lệnh-subagents)
7. [Ví dụ thực tế](#7-ví-dụ-thực-tế)
8. [Quy tắc và giới hạn](#8-quy-tắc-và-giới-hạn)
9. [Xử lý sự cố](#9-xử-lý-sự-cố)

---

## 1. Tổng quan Sub-Agent

Sub-Agent là các agent con được spawn (tạo) từ agent chính (parent) để thực hiện các tác vụ nền (background tasks) một cách độc lập. Mỗi sub-agent chạy trong session riêng biệt, có thể sử dụng model và tool khác nhau.

### Kiến trúc Sub-Agent

```
┌──────────────────────────────────────────────────────────────┐
│                    Parent Agent (Main)                        │
│                                                              │
│  sessions_spawn() ──┬──▶ Sub-Agent 1 (task: "Fix bug")      │
│                     ├──▶ Sub-Agent 2 (task: "Write docs")    │
│                     └──▶ Sub-Agent 3 (task: "Run tests")     │
│                                                              │
│  ◀── Announcement ──┤   (kết quả trả về qua announce flow)  │
└──────────────────────────────────────────────────────────────┘
```

### Đặc điểm chính

- **Cách ly hoàn toàn**: Mỗi sub-agent chạy trong session riêng
- **Không đệ quy**: Sub-agent KHÔNG thể spawn sub-agent khác
- **Tự động cleanup**: Session tự xóa sau thời gian archive (mặc định 60 phút)
- **Đồng thời**: Tối đa 8 sub-agent chạy đồng thời (mặc định)
- **Có timeout**: Có thể đặt timeout để tự động dừng

---

## 2. Cấu hình Sub-Agent trong openclaw.json

### 2.1 Cấu hình mặc định toàn cục

```json
{
  "agents": {
    "defaults": {
      "subagents": {
        "maxConcurrent": 8,
        "archiveAfterMinutes": 60,
        "model": "anthropic/claude-haiku-4-5",
        "thinking": "off"
      }
    }
  }
}
```

| Tham số | Kiểu | Mô tả | Mặc định |
|---------|------|--------|----------|
| `maxConcurrent` | `number` | Số sub-agent chạy đồng thời tối đa | `8` |
| `archiveAfterMinutes` | `number` | Tự xóa session sau N phút | `60` |
| `model` | `string \| {primary, fallbacks[]}` | Model AI mặc định cho sub-agents | Kế thừa từ parent |
| `thinking` | `string` | Mức thinking mặc định | `"off"` |

### 2.2 Cấu hình per-Agent

Mỗi agent có thể override cấu hình sub-agent riêng:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "subagents": {
          "allowAgents": ["coder", "researcher", "writer"],
          "model": "anthropic/claude-sonnet-4-20250514",
          "thinking": "medium"
        }
      },
      {
        "id": "coder",
        "name": "Code Expert",
        "workspace": "/home/user/projects",
        "skills": ["github"]
      },
      {
        "id": "researcher",
        "name": "Research Agent",
        "skills": ["web-research"]
      },
      {
        "id": "writer",
        "name": "Content Writer"
      }
    ]
  }
}
```

| Tham số | Kiểu | Mô tả |
|---------|------|--------|
| `allowAgents` | `string[]` | Danh sách agent ID được phép spawn. `"*"` = tất cả, `[]` = chỉ chính nó |
| `model` | `string \| {primary, fallbacks[]}` | Override model cho sub-agents của agent này |
| `thinking` | `string` | Override thinking level |

### 2.3 Tool Policy cho Sub-Agent

Giới hạn tool mà sub-agent được sử dụng:

```json
{
  "tools": {
    "subagents": {
      "tools": {
        "deny": ["gateway", "cron", "message"],
        "allow": ["read", "write", "edit", "exec", "grep", "find", "ls"]
      }
    }
  }
}
```

### 2.4 Thứ tự ưu tiên Model

Khi xác định model cho sub-agent, hệ thống kiểm tra theo thứ tự:

```
1. sessions_spawn({ model: "..." })      ← Ưu tiên cao nhất
2. agents.list[agentId].subagents.model   ← Per-agent override
3. agents.defaults.subagents.model        ← Global default
4. Parent agent's model                   ← Fallback cuối cùng
```

### 2.5 Thứ tự ưu tiên Thinking Level

```
1. sessions_spawn({ thinking: "..." })           ← Ưu tiên cao nhất
2. agents.list[agentId].subagents.thinking        ← Per-agent override
3. agents.defaults.subagents.thinking             ← Global default
```

---

## 3. Sử dụng sessions_spawn Tool

### 3.1 Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|--------|
| `task` | `string` | **Có** | Mô tả tác vụ cho sub-agent |
| `label` | `string` | Không | Nhãn hiển thị (cho logs/UI) |
| `agentId` | `string` | Không | Agent ID mục tiêu (phải nằm trong `allowAgents`) |
| `model` | `string` | Không | Override model (vd: `"claude-opus-4-6"`, `"anthropic/claude-sonnet-4-20250514"`) |
| `thinking` | `string` | Không | Override thinking level: `off`, `low`, `medium`, `high`, `xhigh` |
| `runTimeoutSeconds` | `number` | Không | Tự động dừng sau N giây (≥ 0) |
| `cleanup` | `"delete" \| "keep"` | Không | Chính sách dọn dẹp session. Mặc định: `"keep"` |

### 3.2 Kết quả trả về

```json
{
  "status": "accepted",
  "childSessionKey": "agent:main:subagent:abc-123",
  "runId": "abc-123",
  "modelApplied": true
}
```

| Trường | Mô tả |
|--------|--------|
| `status` | `"accepted"` (thành công), `"error"` (lỗi), `"forbidden"` (không có quyền) |
| `childSessionKey` | Session key của sub-agent |
| `runId` | ID chạy của sub-agent |
| `modelApplied` | Model override có được áp dụng không |
| `warning` | Cảnh báo (nếu có) |
| `error` | Thông báo lỗi (nếu status = error) |

### 3.3 Ví dụ gọi sessions_spawn

**Spawn cơ bản:**
```json
{
  "task": "Phân tích code trong thư mục src/ và liệt kê các vấn đề bảo mật"
}
```

**Spawn với model và thinking:**
```json
{
  "task": "Giải bài toán tối ưu phức tạp",
  "model": "anthropic/claude-opus-4-20250514",
  "thinking": "high",
  "runTimeoutSeconds": 300,
  "label": "Math Solver"
}
```

**Spawn cross-agent:**
```json
{
  "task": "Viết unit test cho module authentication",
  "agentId": "coder",
  "cleanup": "delete"
}
```

---

## 4. Vòng đời Sub-Agent (Lifecycle)

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Register   │───▶│   Execute    │───▶│   Complete   │
│              │    │              │    │              │
│ - Tạo session│    │ - Chạy task  │    │ - Ghi kết quả│
│ - Ghi registry│   │ - Dùng tools │    │ - Set outcome│
│ - Trả runId  │    │ - Stream LLM │    │              │
└──────────────┘    └──────────────┘    └──────┬───────┘
                                               │
                                               ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Archive    │◀───│   Cleanup    │◀───│   Announce   │
│              │    │              │    │              │
│ - Auto-delete│    │ - Delete nếu │    │ - Đọc reply  │
│   sau N phút │    │   cleanup=   │    │ - Build stats│
│              │    │   "delete"   │    │ - Gửi parent │
└──────────────┘    └──────────────┘    └──────────────┘
```

### Chi tiết từng giai đoạn

#### 4.1 Register (Đăng ký)
- Tạo session key mới: `agent:<targetAgentId>:subagent:<uuid>`
- Lưu vào registry in-memory và file `~/.openclaw/subagents/runs.json`
- Ghi nhận: `runId`, `task`, `requesterSessionKey`, `createdAt`

#### 4.2 Execute (Thực thi)
- Gateway xử lý trên lane `subagent` riêng biệt
- Sub-agent nhận system prompt đặc biệt (xem mục 5)
- Chạy LLM loop: prompt → response → tool calls → results → ...
- Lifecycle listener theo dõi trạng thái

#### 4.3 Complete (Hoàn thành)
- Lifecycle listener bắt event "end" hoặc "error"
- Ghi `endedAt` và `outcome` (`ok`, `error`, `timeout`)

#### 4.4 Announce (Thông báo)
- Đợi sub-agent hoàn thành (timeout mặc định 60 giây)
- Đọc reply cuối cùng từ child session
- Nếu reply = `ANNOUNCE_SKIP` → bỏ qua thông báo
- Build thông tin thống kê (runtime, tokens, cost)
- Gửi kết quả về parent qua:
  - **Queue** (nếu parent đang chạy embedded PI)
  - **Direct** (gọi gateway trực tiếp)

#### 4.5 Cleanup (Dọn dẹp)
- `cleanup: "delete"` → xóa session ngay lập tức
- `cleanup: "keep"` → giữ session, đánh dấu `cleanupCompletedAt`

#### 4.6 Archive (Lưu trữ)
- Sweep chạy mỗi 60 giây
- Tự động xóa session cũ hơn `archiveAfterMinutes`

---

## 5. Giao tiếp giữa Parent và Sub-Agent

### 5.1 System Prompt của Sub-Agent

Khi sub-agent được spawn, nó nhận system prompt đặc biệt:

```
# Subagent Context
You are a **subagent** spawned by the main agent for a specific task.

## Your Role
- You were created to handle: [TASK]
- Complete this task. That's your entire purpose.
- You are NOT the main agent. Don't try to be.

## Rules
1. Stay focused - your assigned task only
2. Complete the task - final message reports to main agent
3. Don't initiate - no heartbeats, no proactive actions
4. Be ephemeral - you may be terminated after completion

## What You DON'T Do
- NO user conversations (main agent's job)
- NO external messages (email, tweets) unless explicitly tasked
- NO cron jobs or persistent state
- NO pretending to be the main agent

## Session Context
[Label, requester session, channel, session key]
```

### 5.2 Luồng trả kết quả

```
Sub-Agent hoàn thành
    │
    ▼
Đọc reply cuối cùng từ child session
    │
    ▼
Build announcement message:
    ├── Status: ok / error / timeout
    ├── Result: [nội dung reply]
    └── Notes: [runtime, tokens, cost, session info]
    │
    ▼
Gửi về parent agent
    ├── Cùng channel/thread gốc
    └── Parent tổng hợp và trả lời user
```

### 5.3 Announcement Format

Khi sub-agent hoàn thành, parent nhận thông báo với format:

```
Status: ok
Result: [Nội dung kết quả từ sub-agent]
Notes: Runtime 45s | Tokens: 12,500 in / 3,200 out | Cost: $0.02
       Session: agent:main:subagent:abc-123
```

---

## 6. Quản lý Sub-Agent bằng lệnh /subagents

### Các lệnh có sẵn

| Lệnh | Mô tả |
|-------|--------|
| `/subagents list` | Liệt kê tất cả sub-agents của session hiện tại |
| `/subagents stop <index>` | Dừng sub-agent theo index (bắt đầu từ 1) |
| `/subagents stop <runId>` | Dừng sub-agent theo run ID (có thể dùng prefix) |
| `/subagents stop all` | Dừng tất cả sub-agents |
| `/subagents info <index>` | Xem thông tin chi tiết (status, timestamps, session) |
| `/subagents log <index> [count]` | Xem N tin nhắn cuối (mặc định: tất cả) |
| `/subagents send <index> "message"` | Gửi tin nhắn đến sub-agent |

### Ví dụ sử dụng

```bash
# Liệt kê sub-agents đang chạy
/subagents list

# Output:
# 1. [running] "Fix authentication bug" (coder) - started 2m ago
# 2. [completed] "Write API docs" (writer) - finished 5m ago
# 3. [running] "Run integration tests" (main) - started 30s ago

# Xem chi tiết sub-agent #1
/subagents info 1

# Xem 50 tin nhắn cuối của sub-agent #1
/subagents log 1 50

# Dừng sub-agent #3
/subagents stop 3

# Gửi hướng dẫn bổ sung cho sub-agent #1
/subagents send 1 "Cũng kiểm tra module authorization"
```

---

## 7. Ví dụ thực tế

### 7.1 Setup cơ bản - Một agent chính với sub-agents

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-20250514",
      "subagents": {
        "archiveAfterMinutes": 120,
        "model": "anthropic/claude-haiku-4-5"
      }
    },
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Assistant",
        "model": "anthropic/claude-opus-4-20250514",
        "subagents": {
          "allowAgents": ["*"]
        }
      }
    ]
  }
}
```

### 7.2 Setup nâng cao - Nhiều agent chuyên biệt

```json
{
  "agents": {
    "defaults": {
      "subagents": {
        "maxConcurrent": 4,
        "archiveAfterMinutes": 60
      }
    },
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Project Manager",
        "model": "anthropic/claude-opus-4-20250514",
        "subagents": {
          "allowAgents": ["coder", "tester", "writer"],
          "model": "anthropic/claude-sonnet-4-20250514"
        }
      },
      {
        "id": "coder",
        "name": "Senior Developer",
        "workspace": "/home/user/project",
        "model": "anthropic/claude-sonnet-4-20250514",
        "skills": ["github"],
        "tools": {
          "profile": "coding"
        }
      },
      {
        "id": "tester",
        "name": "QA Engineer",
        "workspace": "/home/user/project",
        "model": "anthropic/claude-haiku-4-5",
        "tools": {
          "profile": "coding",
          "deny": ["write"]
        }
      },
      {
        "id": "writer",
        "name": "Tech Writer",
        "model": "anthropic/claude-sonnet-4-20250514",
        "tools": {
          "profile": "minimal",
          "alsoAllow": ["read", "write", "edit"]
        }
      }
    ]
  }
}
```

### 7.3 Kịch bản sử dụng: Phát triển tính năng song song

Parent agent (main) nhận yêu cầu: "Thêm tính năng authentication cho API"

```
Parent Agent gửi 3 sessions_spawn:

1. sessions_spawn({
     task: "Implement JWT authentication middleware in src/middleware/auth.ts",
     agentId: "coder",
     thinking: "medium"
   })

2. sessions_spawn({
     task: "Write unit tests for JWT authentication",
     agentId: "tester",
     thinking: "low"
   })

3. sessions_spawn({
     task: "Write API documentation for authentication endpoints",
     agentId: "writer"
   })

→ 3 sub-agents chạy song song
→ Kết quả trả về parent qua announce flow
→ Parent tổng hợp và báo cáo cho user
```

### 7.4 Kịch bản: Nghiên cứu với timeout

```json
{
  "task": "Research và so sánh 5 database solutions cho real-time chat: PostgreSQL, MongoDB, Redis, Cassandra, CockroachDB. Đánh giá về performance, scalability, cost.",
  "model": "anthropic/claude-opus-4-20250514",
  "thinking": "high",
  "runTimeoutSeconds": 600,
  "label": "DB Research",
  "cleanup": "keep"
}
```

---

## 8. Quy tắc và giới hạn

### 8.1 Những gì Sub-Agent CÓ THỂ làm

- Đọc/ghi/chỉnh sửa file (nếu tool được phép)
- Chạy lệnh shell (nếu exec tool được phép)
- Tìm kiếm web (nếu web_search được phép)
- Sử dụng browser (nếu được phép)
- Phân tích media (ảnh, audio, video)

### 8.2 Những gì Sub-Agent KHÔNG THỂ làm

| Giới hạn | Mô tả |
|----------|--------|
| Không spawn sub-agent | Sub-agent không thể tạo sub-agent khác (cấm đệ quy) |
| Không dùng session tools | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn` bị loại bỏ |
| Không gửi tin nhắn | Không trực tiếp gửi tin nhắn qua channel (trừ khi task yêu cầu rõ ràng) |
| Không tạo cron job | Không thể lên lịch tác vụ |
| Không giả làm main agent | System prompt ngăn chặn rõ ràng |
| Không heartbeat | Không có chế độ heartbeat |

### 8.3 Giới hạn kỹ thuật

| Giới hạn | Giá trị | Ghi chú |
|----------|---------|---------|
| Max concurrent | 8 | Cấu hình qua `maxConcurrent` |
| Announce timeout | 60 giây | Thời gian đợi sub-agent hoàn thành trước khi announce |
| Archive sweep | 60 giây | Tần suất quét sessions cũ |
| Default archive | 60 phút | Thời gian giữ session trước khi tự xóa |

---

## 9. Xử lý sự cố

### 9.1 Sub-agent không được phép spawn

**Lỗi:** `status: "forbidden"`

**Nguyên nhân:** Agent ID không nằm trong `allowAgents`

**Giải pháp:**
```json
{
  "agents": {
    "list": [{
      "id": "main",
      "subagents": {
        "allowAgents": ["target-agent-id"]
      }
    }]
  }
}
```

Hoặc cho phép tất cả: `"allowAgents": ["*"]`

### 9.2 Sub-agent bị timeout

**Lỗi:** Outcome status = `"timeout"`

**Nguyên nhân:** `runTimeoutSeconds` quá ngắn cho task

**Giải pháp:** Tăng `runTimeoutSeconds` hoặc chia nhỏ task

### 9.3 Sub-agent không trả kết quả

**Nguyên nhân có thể:**
- Sub-agent reply = `ANNOUNCE_SKIP` (bỏ qua thông báo)
- Announce timeout (60 giây) hết trước khi sub-agent hoàn thành
- Lỗi trong announce flow

**Giải pháp:**
- Kiểm tra log: `/subagents log <index>`
- Kiểm tra info: `/subagents info <index>`
- Đảm bảo task đủ rõ ràng để sub-agent hiểu

### 9.4 Quá nhiều sub-agent đồng thời

**Lỗi:** Sub-agent bị queue hoặc bỏ qua

**Giải pháp:** Tăng `maxConcurrent` hoặc đợi sub-agent hiện tại hoàn thành

```json
{
  "agents": {
    "defaults": {
      "subagents": {
        "maxConcurrent": 16
      }
    }
  }
}
```

### 9.5 Kiểm tra registry

Registry lưu tại: `~/.openclaw/subagents/runs.json`

```bash
# Xem trạng thái tất cả sub-agent runs
cat ~/.openclaw/subagents/runs.json | jq '.runs'
```

### 9.6 Session không tự xóa

**Nguyên nhân:** `cleanup: "keep"` và `archiveAfterMinutes` chưa hết

**Giải pháp:**
- Đợi hết thời gian archive
- Hoặc set `cleanup: "delete"` khi spawn
- Hoặc giảm `archiveAfterMinutes`
