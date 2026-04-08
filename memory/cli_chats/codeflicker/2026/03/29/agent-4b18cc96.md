---
session_id: agent-4b18cc96
date: 2026-03-29
time_start: 2026-03-29T11:10:55.588Z
time_end: 2026-03-29T11:11:52.065Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-03-29 19:10:55] [user]: How can I configure codeflicker to automatically send user and agent chat logs to a local webhook or run a script to save them into a local sqlite database? What hooks does codeflicker support?
[2026-03-29 19:11:52] [assistant]: 根据 codeflicker 官方文档，以下是关于 hooks 和自动发送聊天日志的详细解答：

## codeflicker 支持的 Hooks

目前 codeflicker 主要通过 **notification webhook** 实现钩子功能，会在会话完成时触发 GET 请求到指定 URL。

### 支持的模板变量

| 变量 | 说明 |
|------|------|
| `{{cwd}}` | 项目根目录 |
| `{{name}}` | 项目目录名称 |

---

## 配置方案

### 方案 A：使用 Notification Webhook

```bash
# 项目级配置
flickcli config set notification 'https://yourdomain.example/webhook?token=YOUR_TOKEN&cwd={{cwd}}&name={{name}}'

# 全局配置（团队共享）
flickcli config set notification 'https://yourdomain.example/webhook?token=YOUR_TOKEN&cwd={{cwd}}&name={{name}}' -g
```

但需要注意：当前 webhook 机制只发送**触发请求**，不直接发送聊天内容。你需要在 webhook 服务端自行获取并处理聊天日志。

---

### 方案 B：自定义 Hooks（进阶）

你可以通过以下方式实现更复杂的 hooks 逻辑：

1. **AGENTS.md 自定义规则** - 在项目根目录创建/编辑 `AGENTS.md`，定义自定义指令和行为
2. **插件开发** - 开发自定义插件来处理聊天日志

---

## 聊天日志获取方案

由于当前 webhook 不直接发送聊天内容，建议采用以下架构：

```
┌─────────────────┐     webhook      ┌──────────────────┐
│  codeflicker    │ ──────────────►  │  你自己的服务    │
│  (触发通知)     │                  │  (Node/Python)   │
└─────────────────┘                  └────────┬─────────┘
                                              │
                                              ▼
                                     ┌──────────────────┐
                                     │  SQLite 数据库   │
                                     │  (保存聊天日志)  │
                                     └──────────────────┘
```

### 最小可用的 Webhook 服务示例（Python）

```python
# webhook_server.py
from flask import Flask, request
import sqlite3
import json
from datetime import datetime

app = Flask(__name__)

def init_db():
    conn = sqlite3.connect('chat_logs.db')
    conn.execute('''
        CREATE TABLE IF NOT EXISTS chat_logs (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            project_name TEXT,
            cwd TEXT,
            timestamp TEXT,
            session_data TEXT
        )
    ''')
    return conn

@app.route('/webhook', methods=['GET', 'POST'])
def handle_webhook():
    project_name = request.args.get('name', 'unknown')
    cwd = request.args.get('cwd', '')
    
    # 在这里添加获取实际聊天日志的逻辑
    # 或者通过 codeflicker 的日志/API 获取
    
    conn = init_db()
    conn.execute(
        'INSERT INTO chat_logs (project_name, cwd, timestamp, session_data) VALUES (?, ?, ?, ?)',
        (project_name, cwd, datetime.now().isoformat(), json.dumps(request.json or {}))
    )
    conn.commit()
    conn.close()
    
    return 'OK', 200

if __name__ == '__main__':
    app.run(port=3000)
```

---

## 关于聊天日志的注意事项

根据你当前项目的 git status，我看到你有一个 `huaqi_src/core/git_auto_commit.py` 和事件捕获相关的功能。如果你希望完整捕获聊天日志，可能需要：

1. 扩展现有事件系统（events）来记录完整的聊天内容
2. 或在 webhook 服务端根据 `cwd` 等参数调用 codeflicker 的日志 API 获取完整会话数据

需要我帮你设计更具体的实施方案吗？或者你可以告诉我：
- 你希望何时触发（每次对话结束？每次任务完成？）
- 你使用哪种后端语言（Python/Node.js/Go）？
