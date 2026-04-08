---
session_id: agent-37b7c91e
date: 2026-04-01
time_start: 2026-04-01T03:17:55.150Z
time_end: 2026-04-01T03:19:35.249Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-04-01 11:17:55] [user]: 探索 /Users/lianzimeng/workspace/huaqi-growing 项目，重点找：
1. 日期/时间获取相关代码（今天、昨天的判断逻辑）
2. 对话记忆/历史检索相关代码（从数据库或文件读取历史对话）
3. 数据库存储结构（sqlite相关）
4. 系统提示词(system prompt)中关于日期的内容

返回：相关文件路径、关键代码片段、日期处理逻辑、记忆检索逻辑
[2026-04-01 11:17:58] [assistant]: 我来帮你深入探索这个项目，找出日期/时间处理、对话记忆检索、数据库结构等相关代码。

先从项目结构开始了解：
[2026-04-01 11:18:13] [assistant]: 现在让我深入搜索关键区域：
[2026-04-01 11:18:17] [assistant]: 现在我来详细读取关键文件：
[2026-04-01 11:18:22] [assistant]: 继续深入探索更多关键文件：
[2026-04-01 11:18:31] [assistant]: 继续深入读取更多关键文件：
[2026-04-01 11:18:34] [assistant]: 继续查看更多关键文件：
[2026-04-01 11:18:39] [assistant]: 最后查看时间/日期相关的具体处理代码：
[2026-04-01 11:18:44] [assistant]: 现在让我查看更多细节，特别是"今天/昨天"的判断逻辑：
[2026-04-01 11:18:48] [assistant]: 完美！现在让我再查看一些关键的记忆检索文件：
[2026-04-01 11:18:57] [assistant]: 现在让我总结一下查找结果，生成最终报告：
[2026-04-01 11:19:35] [assistant]: 完美！现在我来生成最终的详细报告：

## 项目深度探索报告

我已完成对 **huaqi-growing** 项目的全面深度探索。以下是重点发现：

---

## **1. 日期/时间获取相关代码**

### **今天/昨天判断逻辑** ✓

**关键文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/cli/chat.py`

```python
# 第 798-803 行 - 最近对话时间判断
if conversations:
    last_ts = conversations[0].get("created_at", "")[:10]
    today = datetime.now().strftime("%Y-%m-%d")
    yesterday = (datetime.now() - timedelta(days=1)).strftime("%Y-%m-%d")
    if last_ts == today:
        last_chat = "今天"
    elif last_ts == yesterday:
        last_chat = "昨天"
    elif last_ts:
        last_chat = last_ts
```

**时间处理的其他关键位置:**

| 文件 | 行数 | 功能 |
|------|------|------|
| `/huaqi_src/reports/daily_report.py` | 33 | `datetime.date.today()` 获取当天日期 |
| `/huaqi_src/reports/weekly_report.py` | 35-36 | `today - timedelta(days=6)` 计算周开始 |
| `/huaqi_src/reports/quarterly_report.py` | 32-39 | `today - timedelta(weeks=13)` 计算季度 |
| `/huaqi_src/reports/morning_brief.py` | 33 | 晨间简报获取今日日期 |
| `/huaqi_src/memory/storage/markdown_store.py` | 45-46 | 按日期组织对话目录 |

**核心时间逻辑:**
- 使用 `datetime.date.today()` 获取当前日期
- 使用 `timedelta` 计算相对日期（昨天、上周等）
- 时间戳存储使用 `ISO format` (`"%Y-%m-%d %H:%M:%S"`)

---

## **2. 对话记忆/历史检索相关代码**

### **a) 混合记忆检索系统** ✓

**关键文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/agent/nodes/chat_nodes.py`

```python
# 第 232-308 行 - retrieve_memories 节点
def retrieve_memories(state: AgentState) -> Dict[str, Any]:
    """检索记忆节点

    来源 1: Chroma 向量库（语义相关，覆盖历史）
    来源 2: 当天 Markdown 文件直接扫描（覆盖向量库尚未收录的今日对话）
    合并去重后注入 system prompt。
    """
```

**两层检索架构:**

| 来源 | 实现 | 优势 |
|------|------|------|
| **Chroma 向量库** | `get_hybrid_search()` | 语义匹配，覆盖历史 |
| **今日 Markdown** | 按日期扫描 `{date_str}_*.md` | 覆盖最新未索引内容 |

**今日记忆扫描逻辑** (lines 274-301):
```python
today_str = _date.today().strftime("%Y%m%d")
today_dir = conversations_dir / _date.today().strftime("%Y/%m")

# 逐行扫描关键词匹配
for md_file in sorted(today_dir.glob(f"{today_str}_*.md"), reverse=True)[:20]:
    # 提取 snippet，添加"[今天 MM/DD]"标签
    date_label = md_file.stem[:8]  # 提取日期
    memories.append(f"[今天 {date_label[4:6]}/{date_label[6:8]}]\n{snippet[:250]}")
```

### **b) Markdown 存储结构** ✓

**关键文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/memory/storage/markdown_store.py`

**目录组织:**
```
conversations_dir/
├── 2024/
│   ├── 01/
│   │   ├── 20240101_120000_session_id.md
│   │   ├── 20240101_150000_session_id.md
│   ├── 02/
│       └── ...
└── 2025/
    └── ...
```

**Markdown 格式** (frontmatter + 对话轮次):
```markdown
---
session_id: thread_id_xxx
created_at: 2024-01-01T12:00:00
turns: 5
---

# 对话记录 - 2024年01月01日 12:00

## 回合 1

**👤 用户**
用户输入内容

**🤖 Huaqi**
助手回复内容

---

## 回合 2
...
```

### **c) 混合搜索引擎** ✓

**关键文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/memory/vector/hybrid_search.py`

**融合策略** (lines 25-31):
```python
融合公式: final_score = alpha * vector_score + (1-alpha) * bm25_score + recency_boost

参数:
- alpha = 0.7          # 向量权重 70%
- beta = 0.1           # 时间衰减系数
- use_bm25 = True      # 精确匹配
- use_vector = True    # 语义匹配
```

**检索流程** (lines 82-120):
1. BM25 召回 top-20 候选集
2. 向量相似度评分
3. 时间衰减加权
4. 融合排序返回 top-5

---

## **3. 数据库存储结构（SQLite）** ✓

### **a) 事件存储** ✓

**关键文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/core/db_storage.py`

**表结构** (lines 18-25):
```python
CREATE TABLE IF NOT EXISTS events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp INTEGER,              # Unix timestamp
    source TEXT,                    # 来源 (CLI, WeChat, etc)
    actor TEXT,                     # 执行者
    content TEXT,                   # 内容
    context_id TEXT                 # 上下文ID
)
```

**查询方法:**
```python
# 获取最近事件 (按时间倒序)
db.get_recent_events(limit=10)

# 搜索事件 (模糊匹配)
db.search_events(query="keyword", limit=5)
```

**报告中使用** (lines 26-29 in `/huaqi_src/reports/providers/events.py`):
```python
# 查询时间范围内的事件
rows = conn.execute(
    "SELECT source, actor, content, timestamp FROM events "
    "WHERE timestamp >= ? AND timestamp <= ? "
    "ORDER BY timestamp DESC LIMIT 20",
    (start_ts, end_ts),
).fetchall()
```

### **b) 日记存储** ✓

**关键文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/core/diary_simple.py`

**目录结构:**
```
memory/diary/
├── 2024/01/
│   ├── 2024-01-01.md
│   ├── 2024-01-02-suffix.md    # 同一天多篇
│   └── ...
├── 2024/02/
└── ...
```

**YAML 前置元数据:**
```python
@dataclass
class DiaryEntry:
    date: str              # YYYY-MM-DD
    content: str
    mood: Optional[str]    # 情绪标签
    tags: List[str]        # 主题标签
    created_at: str        # ISO format
    updated_at: str        # ISO format
```

### **c) 向量存储（Chroma）** ✓

**关键文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/memory/vector/chroma_client.py`

**集合结构:**
```python
collection.add(
    doc_id="session_id_turn_0",
    content="用户：...\nHuaqi：...",
    metadata={
        "session_id": "thread_id_xxx",
        "turn_index": 0,
        "date": "2024-01-01",
        "type": "conversation"
    },
    embedding=[0.1, 0.2, ...]  # 向量
)
```

**持久化路径:** `~/.huaqi/vector_db/` (PersistentClient)

---

## **4. 系统提示词中关于日期的内容** ✓

### **a) 基础系统提示词** ✓

**关键文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/agent/nodes/chat_nodes.py`

```python
# 第 96-115 行 - build_system_prompt 函数
base_prompt = """你是 Huaqi (花旗)，一个个人 AI 伴侣系统。

你的职责：
1. 作为用户的数字伙伴，提供陪伴和支持
2. 记住用户的重要信息和偏好
3. 帮助用户记录日记、追踪成长、管理目标
4. 在内容创作时提供协助

回复风格：
- 温暖、真诚、有同理心
- 简洁明了，避免冗长
- 适当使用 emoji 增加亲和力
- 记住用户的上下文，保持对话连贯
- 根据用户的情绪状态调整回应方式
- 关注用户的深层需求，不只是表面问题
"""
```

### **b) 动态注入的上下文** ✓

**注入源:**
1. **用户画像上下文** - `profile_manager.get_system_prompt_addition()`
2. **自适应理解洞察** - `adaptive.get_context_for_response()`
3. **历史记忆** - `search.search(query, top_k=3)` 注入到 system prompt

```python
# 第 201-203 行
if memories:
    trimmed = [m[:200] for m in memories]
    system_prompt += f"\n\n相关历史记忆（自动检索）：\n" + ...
```

### **c) 报告中的日期模板** ✓

| 报告类型 | 文件 | 系统提示词 |
|---------|------|-----------|
| **日报** | `/huaqi_src/reports/daily_report.py` | "生成一份简洁的日终复盘报告，包含：1）今日主要收获和亮点..." |
| **周报** | `/huaqi_src/reports/weekly_report.py` | 生成 ISO week format: "2024-W01" |
| **季报** | `/huaqi_src/reports/quarterly_report.py` | 计算当前季度，生成 13 周回顾 |
| **晨间** | `/huaqi_src/reports/morning_brief.py` | "请根据以下背景信息，生成一份简洁的晨间简报..." |

---

## **5. 记忆检索流程图** 

```
用户消息
    ↓
query 文本
    ↓
┌─────────────────────────────────────┐
│  来源 1: Chroma 向量库检索            │
│  (BM25 + Vector + 时间衰减)          │
│  → top_k=3 results                   │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  来源 2: 今日 Markdown 扫描           │
│  - 按 {YYMMdd} 扫描最近 20 个文件    │
│  - 逐行关键词匹配                    │
│  - 提取 snippet + 时间标签          │
│  → 最多 5 条记忆                     │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  去重合并                             │
│  (按前 60 字符粗略去重)               │
│  → unique_memories[:5]              │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  注入到 system prompt                │
│  "相关历史记忆（自动检索）：\n..."    │
└─────────────────────────────────────┘
    ↓
LLM 生成回复
```

---

## **6. 核心文件汇总表**

| 文件路径 | 功能 | 关键函数/类 |
|---------|------|-----------|
| `agent/nodes/chat_nodes.py` | 对话流程节点 | `classify_intent()`, `build_context()`, `retrieve_memories()`, `generate_response()` |
| `agent/tools.py` | 工具定义 | `search_diary_tool()`, `search_events_tool()`, `search_huaqi_chats_tool()` |
| `core/db_storage.py` | SQLite 事件存储 | `LocalDBStorage`, `search_events()` |
| `core/diary_simple.py` | 日记管理 | `DiaryStore`, `DiaryEntry` |
| `memory/storage/markdown_store.py` | Markdown 会话存储 | `MarkdownMemoryStore`, `save_conversation()` |
| `memory/vector/hybrid_search.py` | 混合检索 | `HybridSearch` (BM25 + Vector + 时间衰减) |
| `memory/vector/chroma_client.py` | 向量库封装 | `ChromaClient`, `add()`, `search()` |
| `reports/daily_report.py` | 日报生成 | `DailyReportAgent`, 注入 events + diary |
| `reports/weekly_report.py` | 周报生成 | `WeeklyReportAgent`, `timedelta(days=6)` |
| `reports/providers/events.py` | 事件数据提供 | 查询 `timestamp >= start_ts AND timestamp <= end_ts` |
| `reports/providers/diary.py` | 日记数据提供 | 按日期范围迭代加载日记 |
| `cli/chat.py` | CLI 聊天界面 | "今天" vs "昨天" 判断逻辑 |

---

## **7. 关键查找模式**

✅ **已找到:**
- ✓ 日期判断逻辑（"今天"/"昨天"）
- ✓ 两层混合记忆检索（Chroma + Markdown）
- ✓ SQLite 事件表结构
- ✓ Markdown 对话存储格式
- ✓ 向量库 Chroma 集合结构
- ✓ 系统提示词动态构建逻辑
- ✓ 报告中的日期范围计算

**数据流:**
```
对话 → Markdown 存储 + Chroma 向量索引
    ↓
记忆检索 → 向量库 + 今日 Markdown 扫描
    ↓
上下文注入 → System Prompt
    ↓
LLM 生成回复
```

---

所有相关代码已位置精确定位，可直接跳转查阅！
