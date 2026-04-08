---
session_id: agent-bdee932c
date: 2026-03-25
time_start: 2026-03-25T09:33:47.825Z
time_end: 2026-03-25T09:36:36.639Z
project: users-lianzimeng-working-happy-xp
git_branch: 
---

[2026-03-25 17:33:47] [user]: 请帮我探索 /Users/lianzimeng/working/happy/xp 这个项目的整体架构和代码结构。重点是：
1. 项目的入口点 main.py 和 cli.py 的功能
2. 经验沉淀(mcp__xp__extract_experience)和 review 相关的实现
3. 数据存储机制(storage.py)
4. 各模块之间的关系

请详细阅读关键文件并给出项目架构的总结。
[2026-03-25 17:34:07] [assistant]: 我来帮你探索这个项目的整体架构和代码结构。让我先查看项目的目录结构。
[2026-03-25 17:34:18] [assistant]: 现在让我读取这些关键文件来了解项目的整体架构：
[2026-03-25 17:34:42] [assistant]: 让我再查看一下项目的配置文件和技术规格文档：
[2026-03-25 17:36:36] [assistant]: 现在我已经完整阅读了所有关键文件，下面是项目架构的详细总结：

---

# XP - Agent 最佳实践沉淀工具 项目架构总结

## 1. 项目概述

**XP** 是一个面向业务工程师的基础设施工具，核心目标是让 AI Agent 在项目中"越用越聪明"——自动沉淀每次任务的经验，下次遇到相似问题时直接复用。

**核心设计理念：**
- Agent 提取，人工把关
- 注入是建议不是指令
- 拒绝也是数据（形成优化闭环）
- 指标先行（用数据判断工具价值）

---

## 2. 入口点分析

### 2.1 `main.py` - MCP 服务器入口

```
路径: /Users/lianzimeng/working/happy/xp/main.py
```

**功能：** 
- 加载环境变量（`.env` 文件）
- 启动 MCP Server（调用 `src.server.main()`）
- 使用 asyncio 运行异步服务

**启动方式：**
```bash
python main.py
# 或通过 pyproject.toml 定义的入口
xp-server
```

### 2.2 `src/cli.py` - 命令行工具入口

```
路径: /Users/lianzimeng/working/happy/xp/src/cli.py
```

**功能：** 提供人工 review 和统计查看的 CLI 接口

**命令列表：**

| 命令 | 功能 |
|------|------|
| `xp review` | 逐条确认/拒绝待 review 的候选经验 |
| `xp import <文件>` | 从 Markdown 文件批量导入种子经验 |
| `xp stats [--since Nd]` | 查看效果统计 |

**关键函数：**
- `cmd_review()` - Review 流程，支持确认(y)/拒绝/跳过/退出
- `cmd_import()` - 从 Markdown 导入种子经验
- `cmd_stats()` - 展示知识库状态、过程指标、效果对比

---

## 3. 经验沉淀与 Review 机制

### 3.1 `mcp__xp__extract_experience` - 经验提取

**触发时机：** Agent 完成代码生成任务后

**核心流程（`src/knowledge.py` 中 `extract_experience` 方法）：**

```
输入参数
    ↓
自动推断层级 (_infer_level)
    ├── 包含"架构/模式/pattern/设计" → L1
    ├── 包含"bug/fix/修复/报错" → L3
    └── 默认 → L2
    ↓
自动推断类型 (_infer_type)
    ├── 包含"bug/fix/修复/报错" → BUGFIX
    ├── 包含"模式/pattern/架构/规范" → PATTERN
    └── 默认 → FEATURE
    ↓
自动提取关键词 (_extract_keywords)
    - 分词（中英文混合）
    - 去停用词
    - 去重
    ↓
自动推断技术栈 (_infer_tech_stack)
    - 匹配预定义关键词：react/vue/typescript/python/docker...
    ↓
自动推断场景 (_infer_scene)
    - 匹配预定义场景：表单/列表/异步/状态管理/路由/性能优化...
    ↓
生成 Experience 对象
    - status = PENDING（等待 review）
    - confidence = 0.6
    - source = AGENT
    ↓
存入 ExperienceStore
```

### 3.2 Review 流程

**触发方式：** `xp review` 命令

**流程：**
```
列出所有 pending 状态的经验
    ↓
逐条展示：标题、类型、层级、标签、问题、解决方案、元数据
    ↓
用户操作：
    [y] 确认 → status=ACTIVE, confidence=0.8, 记录 review_events
    [n] 拒绝 → status=ARCHIVED, 记录拒绝原因, 记录 review_events
    [s] 跳过 → 保持 pending
    [q] 退出 → 结束
```

### 3.3 数据模型（`src/models.py`）

```
ExperienceType (枚举)
├── BUGFIX    - bug 修复
├── FEATURE   - 功能实现
└── PATTERN   - 模式/规范

ExperienceLevel (枚举)
├── L1 - 模式层（架构级）
├── L2 - 场景层（通用场景）
└── L3 - 修复层（具体问题）

ExperienceStatus (枚举)
├── PENDING   - 待 review
├── ACTIVE    - 已确认，可检索
└── ARCHIVED  - 已归档

ExperienceSource (枚举)
├── AGENT  - agent 自动提取
└── MANUAL - 手动导入

ExperienceMetadata (数据类)
├── tech_stack: list[str]     - 技术栈标签
├── problem_type: str         - 问题类型
├── scene: list[str]          - 场景标签
└── keywords: list[str]       - 关键词

Experience (数据类)
├── id, type, level, title, tags
├── problem, solution
├── confidence, status, source
├── created_at, related_files
├── reject_reason (可选)
└── metadata: ExperienceMetadata

Session (数据类) - 用于效果评估
├── session_id, task_description
├── experience_ids_injected
├── iteration_count
├── had_error_correction
├── user_accepted
└── created_at
```

---

## 4. 数据存储机制（`src/storage.py`）

### 4.1 存储位置

```
~/.xp/
├── knowledge.json    # 经验库（JSON 文件）
├── index.npy         # 向量索引（Phase 1 未使用）
├── index_ids.json    # 向量 ID 映射
└── metrics.db        # SQLite 指标库
```

### 4.2 ExperienceStore - 经验存储

**底层存储：** JSON 文件（`~/.xp/knowledge.json`）

**核心方法：**

| 方法 | 功能 |
|------|------|
| `add(exp)` | 新增经验 |
| `get(exp_id)` | 根据 ID 获取经验 |
| `update(exp)` | 更新经验 |
| `list_by_status(status)` | 按状态筛选 |
| `list_active()` | 获取所有 active 经验 |
| `count_by_status()` | 统计各状态数量 |
| `search(query, tech_stack, problem_type, scene, top_k, threshold)` | BM25 检索 |

**检索逻辑：**
```
1. 筛选 active 状态的经验
2. 标签过滤（精确匹配 metadata 字段）
   - tech_stack 过滤
   - problem_type 过滤
   - scene 过滤
3. BM25 排序
   - 语料库：title + problem + solution + keywords
   - 分词：中英文混合，去标点
   - 使用 rank_bm25.BM25Okapi
4. 按 threshold 过滤，返回 top_k
```

### 4.3 MetricsStore - 指标存储

**底层存储：** SQLite 数据库（`~/.xp/metrics.db`）

**表结构：**

```sql
-- 会话记录表
sessions (
    session_id TEXT PRIMARY KEY,
    task_description TEXT,
    experience_ids_injected TEXT,  -- JSON 数组
    iteration_count INTEGER,
    had_error_correction INTEGER,  -- 0/1
    user_accepted INTEGER,         -- 0/1
    created_at TEXT
)

-- 检索事件表
search_events (
    id INTEGER PRIMARY KEY,
    query TEXT,
    result_count INTEGER,
    created_at TEXT
)

-- Review 事件表
review_events (
    id INTEGER PRIMARY KEY,
    experience_id TEXT,
    action TEXT,           -- confirmed/rejected
    reject_reason TEXT,
    created_at TEXT
)
```

**核心方法：**

| 方法 | 功能 |
|------|------|
| `record_search(query, result_count)` | 记录检索事件 |
| `record_review(experience_id, action, reject_reason)` | 记录 review 操作 |
| `record_session(session)` | 记录会话数据 |
| `get_stats(since_days)` | 获取统计指标 |

---

## 5. MCP 服务（`src/server.py`）

### 5.1 架构

```
MCP Server (src/server.py)
    ├── list_tools()   → 注册 3 个工具
    └── call_tool()    → 分发工具调用到 KnowledgeService
```

### 5.2 提供的 MCP Tools

| Tool | 调用时机 | 功能 |
|------|----------|------|
| `extract_experience` | 任务完成后 | 提取经验，生成 pending 候选 |
| `search_best_practices` | 任务开始前 | BM25 检索历史经验 |
| `record_session` | 任务完成后 | 记录会话数据，用于效果评估 |

### 5.3 工具调用流程

```
MCP Client (Cursor/codeflicker)
        ↓
调用 Tool (search_best_practices / extract_experience / record_session)
        ↓
server.py call_tool()
        ↓
KnowledgeService 对应方法
        ↓
ExperienceStore / MetricsStore
        ↓
JSON 文件 / SQLite 数据库
```

---

## 6. 模块关系图

```
┌─────────────────────────────────────────────────────────────────┐
│                         Entry Points                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   main.py                    cli.py                             │
│   (MCP Server 入口)           (CLI 入口)                          │
│        │                          │                              │
│        ↓                          ↓                              │
│   ┌─────────┐              ┌─────────────┐                      │
│   │ asyncio │              │ cmd_review  │                      │
│   │         │              │ cmd_import  │                      │
│   │         │              │ cmd_stats   │                      │
│   └────┬────┘              └──────┬──────┘                      │
│        │                          │                              │
└────────┼──────────────────────────┼──────────────────────────────┘
         │                          │
         ↓                          ↓
┌─────────────────────────────────────────────────────────────────┐
│                     server.py (MCP Layer)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   list_tools()  → 注册 MCP 工具定义                              │
│   call_tool()   → 分发工具调用                                   │
│                                                                  │
│   Tools:                                                         │
│   - extract_experience   → KnowledgeService.extract_experience  │
│   - search_best_practices→ KnowledgeService.search              │
│   - record_session       → KnowledgeService.record_session      │
│                                                                  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                 knowledge.py (Business Logic)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   KnowledgeService                                               │
│   ├── extract_experience()  - 提取经验（自动推断元数据）          │
│   ├── search()              - 检索经验                           │
│   ├── record_session()      - 记录会话                           │
│   ├── confirm_experience()  - 确认经验                           │
│   ├── reject_experience()   - 拒绝经验                           │
│   ├── edit_and_confirm()    - 编辑后确认                         │
│   ├── import_from_markdown()- 从 Markdown 导入                   │
│   ├── list_pending()        - 列出待 review 经验                 │
│   ├── get_stats()           - 获取统计                           │
│   └── _infer_*              - 自动推断辅助方法                    │
│                                                                  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    storage.py (Data Layer)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ExperienceStore (JSON)         MetricsStore (SQLite)          │
│   ├── add()                      ├── record_search()            │
│   ├── get()                      ├── record_review()            │
│   ├── update()                   ├── record_session()           │
│   ├── list_by_status()           └── get_stats()                │
│   ├── list_active()                                             │
│   ├── count_by_status()                                         │
│   └── search() (BM25)                                           │
│                                                                  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      models.py (Data Models)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Enums:                                                         │
│   - ExperienceType (BUGFIX/FEATURE/PATTERN)                     │
│   - ExperienceLevel (L1/L2/L3)                                  │
│   - ExperienceStatus (PENDING/ACTIVE/ARCHIVED)                  │
│   - ExperienceSource (AGENT/MANUAL)                             │
│                                                                  │
│   Dataclasses:                                                   │
│   - ExperienceMetadata (tech_stack, problem_type, scene, kw)    │
│   - Experience (核心经验实体)                                    │
│   - Session (会话记录)                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. 技术栈总结

| 模块 | 技术选型 |
|------|----------|
| MCP 框架 | `mcp[cli]>=1.0.0` |
| 检索引擎 | `rank-bm25` (BM25Okapi) |
| 环境变量 | `python-dotenv` |
| 数据存储 | JSON 文件 + SQLite |
| Python 版本 | >= 3.11 |

---

## 8. 核心工作流

### 8.1 经验沉淀流程

```
Agent 完成任务
      ↓
调用 extract_experience MCP tool
      ↓
KnowledgeService.extract_experience()
      ├── 自动推断 type/level
      ├── 自动提取 keywords
      ├── 自动推断 tech_stack/scene
      └── 生成 Experience (status=PENDING)
      ↓
存入 knowledge.json
      ↓
人工运行: xp review
      ├── 确认 → ACTIVE (可检索)
      └── 拒绝 → ARCHIVED (记录原因)
```

### 8.2 经验检索流程

```
Agent 开始新任务
      ↓
调用 search_best_practices MCP tool
      ↓
KnowledgeService.search()
      ↓
ExperienceStore.search()
      ├── 筛选 active 经验
      ├── 标签过滤
      └── BM25 排序
      ↓
返回相关经验 + 记录 search_event
      ↓
Agent 参考经验生成代码
      ↓
任务完成，调用 record_session
      ↓
记录效果数据
```

---

这个项目是一个完整的**经验管理闭环系统**，从经验提取、人工审核、检索注入到效果评估，形成了完整的反馈机制。Phase 1 采用轻量级的本地存储方案，未来 Phase 2/3 规划了云端同步、团队协作、动态权重等高级特性。
