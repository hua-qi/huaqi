---
session_id: agent-b3017ca4
date: 2026-04-02
time_start: 2026-04-02T11:45:04.640Z
time_end: 2026-04-02T11:46:09.327Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-04-02 19:45:04] [user]: Analyze the codebase to explain in detail how Huaqi integrates with WeChat. Look at files like `huaqi_src/integrations/wechat_webhook.py`, `huaqi_src/layers/data/collectors/wechat_*`, `config.yaml` structures, and any relevant markdown documentation (like PRDs, architecture, or guides).

Explain:
1. What the user needs to configure (e.g. `config.yaml` settings for `modules.wechat`).
2. What underlying tools or frameworks are used (e.g. is it a webhook? a desktop client hook? a specific library?).
3. The specific steps a user needs to take to enable and use the WeChat integration.
[2026-04-02 19:45:12] [assistant]: I'll help you analyze the WeChat integration in Huaqi. Let me start by exploring the codebase to find all relevant files.
[2026-04-02 19:45:18] [assistant]: Now let me read the key files to understand the WeChat integration:
[2026-04-02 19:46:09] [assistant]: Based on my analysis of the codebase, I can now provide a detailed explanation of how Huaqi integrates with WeChat.

---

# WeChat Integration in Huaqi - Detailed Analysis

## CRITICAL: Integration is ARCHIVED/DEPRECATED

**Important**: All WeChat integration code has been **archived and is no longer functional**. Every WeChat-related file contains this header:

```python
# =============================================================================
# 封存声明 / ARCHIVED
#
# 微信采集相关代码已封存，不再提供任何对外入口（CLI 命令、Agent Tool、定时任务）。
# 原因：微信 4.x macOS 版本改用 SQLCipher 加密本地数据库，且 macOS SIP 保护
# 阻止对 /Applications 目录下二进制进行重签名，无法在不破坏系统安全策略的前提下
# 读取本地数据。
#
# 本文件仅作技术参考，非作者本人声明不得重新为 wechat 添加任何系统入口。
# =============================================================================
```

**Why it was archived:**
- WeChat 4.x macOS switched to **SQLCipher** encryption for local databases
- **macOS SIP (System Integrity Protection)** prevents re-signing binaries in `/Applications`
- There is no compliant way to read local WeChat data without breaking system security policies

---

## Technical Architecture (Historical Reference)

### 1. Core Components

| File | Purpose |
|------|---------|
| `wechat_reader.py` | SQLite database reader for WeChat message history |
| `wechat_writer.py` | Markdown archiver for parsed messages |
| `wechat_state.py` | Incremental sync state persistence (JSON) |
| `wechat_watcher.py` | Orchestrator combining reader + state + writer |
| `wechat_webhook.py` | HTTP webhook server for receiving external events |

### 2. How It Worked (Historically)

**Data Flow:**
```
WeChat Local DB (SQLite) 
    → WeChatDBReader (wechat_reader.py)
    → WeChatSyncState (track last rowid)
    → WeChatWriter (wechat_writer.py)
    → Markdown files in memory/wechat/YYYY-MM/<contact>.md
```

**Database Reader (`wechat_reader.py`):**
- Connected to WeChat's local SQLite database (unencrypted in older versions)
- Scanned tables matching `Chat_%` pattern (one table per contact)
- Read messages with `Type = 1` (text messages only)
- Tracked `rowid` for incremental sync

**Message Format:**
```python
@dataclass
class WeChatMessage:
    rowid: int
    contact: str        # Contact identifier
    timestamp: datetime
    content: str        # Message text
    is_self: bool       # des == 0 means sent by user
```

**Writer Output (`wechat_writer.py`):**
- Organized by month: `memory/wechat/YYYY-MM/<contact>.md`
- Format: `**我/对方** [HH:MM]: 消息内容`

### 3. Configuration Required (Historical)

**Directory Structure:**
```
<DATA_DIR>/
├── wechat_db/              # WeChat database files (configured via get_wechat_db_dir())
│   └── *.db                # Message databases
├── memory/
│   └── wechat/             # Output directory (configured via get_wechat_dir())
│       └── YYYY-MM/
│           └── <contact>.md
└── wechat_state.json       # Sync state tracking
```

**Configuration via `config/manager.py`:**
```python
# Module enable/disable (Opt-in)
config.modules["wechat"] = True  # or False to disable

# Paths defined in config/paths.py:
def get_wechat_db_dir() -> Path:
    return require_data_dir() / "wechat_db"

def get_wechat_dir() -> Path:
    return get_memory_dir() / "wechat"
```

### 4. Webhook Server (`wechat_webhook.py`)

An HTTP server for receiving external WeChat events:

```python
# Endpoint: POST /api/webhook/wechat
# Request body:
{
    "actor": "Sender name",
    "content": "Message content",
    "context_id": "Conversation ID"
}

# Response:
{"status": "success", "message": "Event recorded"}
```

- Runs on `127.0.0.1:8080` by default
- Stores events via `LocalDBStorage` in SQLite
- Checks `modules.wechat` config before processing

### 5. Raw Signal Converter (`layers/data/raw_signal/converters/wechat.py`)

For the unified data ingestion pipeline:
```python
class WechatConverter(BaseConverter):
    # Converts WeChat chat logs to RawSignal format
    # Pattern: "YYYY-MM-DD HH:MM:SS Sender\nContent"
```

---

## User Configuration Steps (Historical - No Longer Functional)

### Step 1: Enable the Module
```bash
huaqi config set modules.wechat true
```

### Step 2: Place Database Files
Copy WeChat SQLite databases to:
```
<DATA_DIR>/wechat_db/
```

### Step 3: Run Sync
```bash
huaqi collector sync-wechat  # This command has been REMOVED
```

---

## What Has Been Removed

According to `CHANGELOG.md`:

| Removed Item | Type |
|--------------|------|
| `huaqi collector sync-wechat` | CLI command |
| `search_wechat_tool` | Agent Tool |
| `wechat_sync` | Scheduler job |
| All public entry points | System integration |

**Files preserved but archived (no system entry points):**
- `wechat_reader.py`
- `wechat_state.py`
- `wechat_writer.py`
- `wechat_watcher.py`
- `wechat_webhook.py`

---

## Current State Summary

| Aspect | Status |
|--------|--------|
| **Code files** | Present but archived |
| **CLI commands** | Removed |
| **Agent Tools** | Removed |
| **Scheduler jobs** | Removed |
| **Configuration** | `modules.wechat` key exists but non-functional |
| **Technical barrier** | SQLCipher encryption + macOS SIP |

---

## Alternatives for Users

Since local WeChat data access is blocked, alternatives include:

1. **Manual export**: Export WeChat chat history manually and import as Markdown files
2. **Webhook integration**: If a third-party service provides WeChat data via webhook, the `wechat_webhook.py` could theoretically be reactivated (requires author approval)
3. **Official API**: Wait for Tencent to provide official APIs for personal data access

---

## Documentation References

- `docs/features/listeners.md` - Contains the archive notice
- `CHANGELOG.md` - Documents the removal of all WeChat entry points
- `docs/design/ARCHITECTURE.md` - Shows WeChat components in the data layer architecture
