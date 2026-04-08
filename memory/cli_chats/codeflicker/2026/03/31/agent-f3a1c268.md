---
session_id: agent-f3a1c268
date: 2026-03-31
time_start: 2026-03-31T16:18:41.217Z
time_end: 2026-03-31T16:20:19.581Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-01 00:18:41] [user]: 请分析 /Users/lianzimeng/workspace/xp 项目的整体架构，包括：
1. 目录结构
2. 主要模块及其职责
3. 模块间的依赖关系
4. 技术栈
5. 数据流向

请详细描述每个文件/模块的作用，返回一份清晰的架构概述。
[2026-04-01 00:18:44] [assistant]: 我来帮您分析这个项目的整体架构。首先让我探索项目的目录结构和关键文件。
[2026-04-01 00:18:47] [assistant]: 现在让我详细读取关键文件来理解架构：
[2026-04-01 00:18:51] [assistant]: 现在让我读取关键的模块代码：
[2026-04-01 00:18:55] [assistant]: 让我继续读取更多关键文件：
[2026-04-01 00:18:59] [assistant]: 让我检查一下PostgreSQL后端和其他重要文件：
[2026-04-01 00:19:03] [assistant]: 现在让我查看一些测试文件和设计文档来更好理解系统：
[2026-04-01 00:19:07] [assistant]: 现在让我查看文件监听和其他工具文件：
[2026-04-01 00:20:19] [assistant]: 完美！现在让我生成一份清晰的架构概述文档。

## XP 项目架构分析

基于我对项目的深入分析，以下是 XP 项目的完整架构概述：

---

### 1. 项目定位

**XP** 是一个 **Agent 最佳实践沉淀工具**，用于在 Agent（如 Cursor、Claude、Windsurf）执行任何任务时自动采用或沉淀最佳实践。通过 MCP（Model Context Protocol）协议暴露 tools，实现经验的自动提取、检索和注入，支持团队多人共享。

**适用任务**：编写代码、生成文档、Debug 排查、代码重构、配置环境、代码审查、测试编写等任何通过 Agent 完成的任务。

---

### 2. 目录结构

```
/Users/lianzimeng/workspace/xp/
├── src/                          # 核心源代码
│   ├── __init__.py
│   ├── cli.py                    # CLI 命令行工具
│   ├── server.py                 # MCP Server（提供 tools 给 IDE）
│   ├── knowledge.py              # 知识服务层（核心业务逻辑）
│   ├── storage.py                # 存储层（JSON + SQLite）
│   ├── models.py                 # 数据模型定义
│   ├── embeddings.py             # Embedding 模型管理
│   ├── cloud_providers.py        # 云端存储 Provider 抽象
│   ├── project_config.py         # 项目配置管理
│   ├── file_watcher.py           # 文件监听和 TTL 管理
│   └── backends/                 # 存储后端（可插拔）
│       ├── base.py               # 后端抽象接口
│       ├── local.py              # 本地后端实现
│       └── postgres.py           # PostgreSQL 后端实现
├── tests/                        # 测试
│   ├── test_backends.py
│   ├── test_finalize_task.py
│   ├── test_extraction_refactor.py
│   ├── test_migration.py
│   ├── test_quality_score.py
│   ├── test_review_refactor.py
│   └── test_stats_metrics.py
├── scripts/                      # 工具脚本
│   └── migrate_to_postgres.py    # 数据库迁移脚本
├── docs/                         # 文档和设计方案
│   ├── designs/                  # 设计决策记录
│   ├── plans/                    # 实现计划
│   ├── hybrid-search-design.md   # 混合检索设计
│   └── embedding-refactor-*.md
├── pyproject.toml                # 项目配置和依赖
├── README.md                     # 使用指南
├── ARCHITECTURE_V2.md            # 多人共享改造方案
├── TECH_SPEC.md                  # 技术规范
└── xp-wrapper.sh                 # Shell 包装脚本
```

---

### 3. 主要模块及其职责

#### 3.1 **CLI 模块** (`cli.py`)
- **职责**：提供命令行接口，支持本地经验管理
- **主要命令**：
  - `xp add` - 交互式手动添加经验
  - `xp review` - 逐条确认/拒绝待 review 的经验
  - `xp edit <ID>` - 编辑已有经验
  - `xp archive/delete <ID>` - 归档或删除经验
  - `xp import <文件>` - 批量导入 Markdown 经验
  - `xp stats` - 查看统计数据
  - `xp analyze` - 分析经验质量
  - `xp project list/switch` - 项目管理
  - `xp watch` - 文件变更检查

#### 3.2 **MCP Server 模块** (`server.py`)
- **职责**：提供 MCP 协议接口，支持 IDE 集成
- **暴露的 Tools**（4 个核心）：
  1. `search_best_practices` - 任务开始前检索经验
  2. `extract_experience` - 任务完成后沉淀经验
  3. `record_feedback` - 反馈经验采纳情况
  4. `record_session` - 记录会话效果数据
- **通信方式**：stdio 或 SSE transport（用于云端部署）

#### 3.3 **知识服务层** (`knowledge.py`)
- **职责**：核心业务逻辑，协调各模块协作
- **主要功能**：
  - `extract_experience()` - 提取和结构化经验
  - `search()` - 执行混合检索（BM25 + Embedding）
  - `record_session/feedback()` - 记录指标数据
  - `check_stale_experiences()` - 检查失效经验
  - `run_ttl_check()` - TTL 过期检查（90天未命中自动归档）
  - 项目管理（创建、切换、列表）

#### 3.4 **存储层** (`storage.py`)
- **职责**：抽象存储操作，支持多后端
- **三个主要存储**：
  1. **ExperienceStore** - 经验数据（JSON 格式，本地 `~/.xp/knowledge.json`）
  2. **MetricsStore** - 指标数据（SQLite，本地 `~/.xp/metrics.db`）
  3. **VectorStore** - 向量索引（SQLite，压缩存储）

#### 3.5 **数据模型** (`models.py`)
- **主要数据类**：
  - `Experience` - 经验实体（包含元数据、向量、统计信息）
  - `ExperienceMetadata` - 结构化元数据（技术栈、问题类型、场景标签）
  - `Session` - 会话记录（用于 A/B 测试和效果评估）
  - `Feedback` - 经验反馈（采纳情况）
  - `ExperienceStats` - 经验统计（命中率、采纳率、TTL 追踪）
  - `ProjectConfig` - 项目配置

#### 3.6 **Embedding 模块** (`embeddings.py`)
- **职责**：Embedding 模型提供者抽象
- **支持的提供者**：
  1. **LocalBGEProvider** - 本地 BGE-small-zh 模型（默认）
  2. **CloudAPIProvider** - 云端 OpenAI/兼容 API
- **方法**：
  - `embed_texts()` - 批量编码文本为向量
  - `embed_text()` - 单文本编码
- **向量维度**：512（BGE-small-zh）

#### 3.7 **云端存储** (`cloud_providers.py`)
- **职责**：云端同步 Provider 抽象（Phase 3 特性）
- **支持的 Provider**：
  - `SupabaseProvider` - Supabase PostgreSQL
  - `WeaviateProvider` - Weaviate 向量数据库
  - `ElasticsearchProvider` - Elasticsearch
- **操作**：上传、下载、列表、云端检索、双向同步

#### 3.8 **后端系统** (`backends/`)
- **职责**：可插拔存储后端，支持本地和云端
- **后端实现**：
  1. **LocalBackend** - 纯本地模式（JSON + SQLite）
  2. **PostgresBackend** - PostgreSQL 后端（支持 K8s 云端部署）
  - 两个后端都实现相同接口，支持无缝切换

#### 3.9 **项目管理** (`project_config.py`)
- **职责**：多项目隔离管理（Phase 3 特性）
- **功能**：
  - 创建/列出/切换项目
  - 项目标签和根路径配置
  - 云端同步配置

#### 3.10 **文件监听** (`file_watcher.py`)
- **职责**：追踪经验相关文件变更，自动失效检测（Phase 3 特性）
- **功能**：
  - `calculate_file_hash()` - SHA256 文件 hash
  - `FileWatcher.check_files_changed()` - 检测文件变更
  - `TTLManager.check_experiences()` - 90天 TTL 管理

---

### 4. 模块间的依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                     IDE 客户端 (Cursor/Claude)              │
│                    通过 MCP 协议调用 tools                   │
└──────────────────────────┬──────────────────────────────────┘
                           │ MCP Transport (stdio/SSE)
┌──────────────────────────▼──────────────────────────────────┐
│                   MCP Server (server.py)                     │
│  ├─ search_best_practices      ┐                             │
│  ├─ extract_experience         ├─── 暴露 4 个核心 tools      │
│  ├─ record_feedback            │                             │
│  └─ record_session             ┘                             │
└──────────────────────────┬──────────────────────────────────┘
                           │ 依赖
┌──────────────────────────▼──────────────────────────────────┐
│              知识服务层 (knowledge.py)                       │
│  - 核心业务逻辑（extract/search/record）                     │
│  - 文件监听和 TTL 管理                                       │
│  - 项目管理                                                  │
│  - 反馈流程                                                  │
└──────────────────────────┬──────────────────────────────────┘
         ┌────────────┬────────────┬─────────────┐
         │ 依赖       │ 依赖       │ 依赖        │
┌────────▼──────┐ ┌──▼───────────┐ ┌──▼────────┐ ┌───▼──────────┐
│ 存储层         │ │ Embedding    │ │ 文件监听  │ │ 项目配置     │
│ storage.py    │ │ embeddings.  │ │ file_    │ │ project_    │
│               │ │ py           │ │ watcher. │ │ config.py    │
│ ├─ Experience │ │              │ │ py       │ │              │
│ │   Store    │ │ ├─LocalBGE    │ │          │ │ ├─创建项目   │
│ ├─ Metrics   │ │ └─CloudAPI    │ │ ├─FileW. │ │ ├─切换项目   │
│ └─ Vector    │ │   Provider    │ │ │        │ │ └─配置管理   │
│   Store      │ │                │ │ └─TTLMgr│ │              │
└──────────────┘ └────────────────┘ └────────┘ └──────────────┘
         │                                              │
         │ 实现                                         │
         │                                              │
┌────────▼──────────────────────────────────────────────┴──────┐
│                  后端系统 (backends/)                        │
│  ├─ LocalBackend   → ~/.xp/knowledge.json + metrics.db      │
│  └─ PostgresBackend → PostgreSQL (K8s 云端部署)             │
└──────────────────────────────────────────────────────────────┘
         │                  │
         │ 读写            │ 连接
         │                  │
┌────────▴──┐      ┌────────▴──────────────┐
│ 本地文件   │      │  PostgreSQL Database  │
│ 系统       │      │  (KDB 托管)           │
│            │      │  - experiences       │
│ knowledge. │      │  - sessions          │
│ json       │      │  - feedback          │
│ metrics.db │      │  - stats             │
└────────────┘      └───────────────────────┘
```

**关键依赖**：
1. **Server → Knowledge** - MCP tools 委托给知识服务处理
2. **Knowledge → Storage** - 数据操作通过存储层
3. **Storage → Backend** - 使用可插拔后端（本地/云端）
4. **Knowledge → Embeddings** - 计算文本向量
5. **Knowledge → FileWatcher** - 监听文件变更
6. **Knowledge → ProjectConfig** - 管理项目隔离

---

### 5. 技术栈

```
┌──────────────────────────────────────────┐
│         编程语言 & 核心框架              │
├──────────────────────────────────────────┤
│ • Python 3.11+                           │
│ • MCP (Model Context Protocol)           │
│ • FastAPI (计划用于 K8s 部署)             │
│ • AsyncIO (异步编程)                     │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│         搜索 & 检索                      │
├──────────────────────────────────────────┤
│ • rank-bm25 (BM25 全文搜索)              │
│ • sentence-transformers (Embedding)      │
│   - 本地：BAAI/bge-small-zh-v1.5         │
│   - 云端：OpenAI API                     │
│ • numpy (向量计算)                       │
│ • 混合检索：BM25 70% + Embedding 30%    │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│         数据存储                         │
├──────────────────────────────────────────┤
│ 本地：                                   │
│ • JSON (knowledge.json - 经验库)         │
│ • SQLite (metrics.db - 指标库)           │
│                                          │
│ 云端（可选）：                           │
│ • PostgreSQL (KDB 托管)                  │
│ • pgvector (向量存储扩展)                │
│ • asyncpg (异步 PG 驱动)                 │
│                                          │
│ 云端 Provider（可选）：                  │
│ • Supabase                               │
│ • Weaviate                               │
│ • Elasticsearch                          │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│         环境 & 工具                      │
├──────────────────────────────────────────┤
│ • python-dotenv (.env 配置)              │
│ • uvicorn (ASGI 服务器)                  │
│ • pytest (单元测试)                      │
│ • readline (CLI 中文输入)                │
└──────────────────────────────────────────┘
```

**核心依赖版本**：
```toml
mcp[cli]>=1.0.0
rank-bm25>=0.2.2
python-dotenv>=1.0.0
sentence-transformers>=2.2.0
numpy>=1.24.0
asyncpg>=0.29.0
fastapi>=0.110.0
uvicorn>=0.29.0
pgvector>=0.2.0
```

---

### 6. 数据流向

#### 6.1 **检索流程**（Agent 开始任务前）

```
User Input (IDE)
    │
    ▼
search_best_practices Tool (MCP Server)
    │ query, tags, top_k
    ▼
KnowledgeService.search()
    │
    ├─ Step 1: 标签过滤
    │   └─ 按 metadata.tech_stack 过滤候选
    │
    ├─ Step 2: BM25 排序 (70% 权重)
    │   ├─ 对 query 和 experience 做分词
    │   ├─ 计算 BM25 分数
    │   └─ 保留 > threshold 的结果
    │
    ├─ Step 3: Embedding 排序 (30% 权重)
    │   ├─ 将 query 编码为向量
    │   ├─ 从 VectorStore 取出所有经验向量
    │   ├─ 计算余弦相似度
    │   └─ 归一化相似度分数
    │
    ├─ Step 4: 混合排序
    │   └─ final_score = 0.7 * bm25_score + 0.3 * embedding_score
    │
    └─ Step 5: 返回 TopK
        └─ 按 final_score 排序，返回前 K 个结果

    ▼
MetricsStore.record_search_event() 记录搜索事件
    │
    ▼
返回给 Agent：
[
  {
    "title": "经验标题",
    "problem": "问题描述",
    "solution": "解决方案",
    "similarity": 0.85,
    "tech_stack": ["react", "hooks"],
    ...
  },
  ...
]
```

**记录的数据**：
- 搜索查询
- 返回结果数
- 时间戳
- 会话 ID（用于关联后续反馈）

#### 6.2 **提取流程**（Agent 完成任务后）

```
extract_experience Tool (MCP Server)
    │ task_description, solution_summary, key_decisions, tags
    ▼
KnowledgeService.extract_experience()
    │
    ├─ Step 1: 生成 Experience ID (UUID)
    │
    ├─ Step 2: LLM 结构化提取（可选，当前使用 rule-based）
    │   ├─ 生成标题（从 task_description 简化）
    │   ├─ 判断经验类型（bugfix/feature/pattern）
    │   ├─ 判断经验级别（L1/L2/L3）
    │   ├─ 提取元数据关键词
    │   └─ 检测技术栈标签
    │
    ├─ Step 3: 去重检测（相似度 > 0.9）
    │   ├─ 调用 search() 检索相似经验
    │   └─ 若相似度 > 0.9，merge 而非新增
    │
    ├─ Step 4: 创建 Experience 对象
    │   ├─ status = "pending"
    │   ├─ source = "agent"
    │   ├─ confidence = 0.6 (初始值)
    │   └─ 计算文件 hash（如有 related_files）
    │
    ├─ Step 5: 计算 Embedding
    │   ├─ 连接 text = title + problem + solution + keywords
    │   ├─ 调用 EmbeddingProvider.embed_texts()
    │   └─ 保存向量到 VectorStore
    │
    └─ Step 6: 保存到存储
        ├─ ExperienceStore.add(experience) → knowledge.json
        ├─ VectorStore.save_vector(id, vec) → metrics.db
        └─ MetricsStore.record_review_event()

    ▼
返回给 Agent：
{
  "id": "uuid-xxx",
  "title": "自动生成的标题",
  "status": "pending",
  "created_at": "2026-03-31T12:00:00",
  ...
}

提示用户：运行 `xp review` 进行审核确认
```

#### 6.3 **反馈和统计流程**（任务完成后）

```
record_session Tool (MCP Server)              record_feedback Tool (MCP Server)
    │ session_id, task, exps, iterations         │ experience_id, adopted, reason
    ▼                                             ▼
MetricsStore.record_session()             MetricsStore.record_feedback()
    │ 保存 Session 对象到 metrics.db          │ 保存 Feedback 对象到 metrics.db
    ▼                                             ▼
    └─────────────┬──────────────┬────────────┘
                  │ 数据流向      │
                  ▼              ▼
        sessions 表         feedback 表
    (session_id, ...)   (experience_id, adopted, ...)
        │                  │
        ▼                  ▼
   experience_stats 表（聚合更新）
    ├─ hit_count ← search 事件
    ├─ adopted_count ← feedback(adopted=true)
    ├─ rejected_count ← feedback(adopted=false)
    ├─ adoption_rate = adopted_count / hit_count
    └─ last_used_at, last_adopted_at

    ▼
xp stats 命令或 Web Dashboard
    │ 读取并聚合指标
    ▼
显示统计报告：
- 知识库状态（经验数、待审核数）
- 过程指标（检索命中率、确认率等）
- 效果对比（有/无经验注入的差异）
  ├─ 平均对话轮数
  ├─ 报错率
  └─ 用户接受率
```

#### 6.4 **Review 流程**（人工审核）

```
xp review 命令 (CLI)
    │
    ▼
ExperienceStore.list_by_status("pending")
    └─ 读取 knowledge.json 中所有 pending 经验

    ▼
用户交互：逐条显示经验详情
    │
    ├─ 选项 1: y (confirm)
    │   └─ 状态更改：pending → active
    │       confidence = 0.8 (未编辑) or 1.0 (编辑后)
    │
    ├─ 选项 2: e (edit then confirm)
    │   ├─ 交互式编辑 (title, problem, solution, tags)
    │   └─ 状态更改：pending → active，confidence = 1.0
    │
    ├─ 选项 3: n (reject)
    │   ├─ 输入拒绝原因
    │   └─ 状态更改：pending → archived，记录 reject_reason
    │
    └─ 选项 4: s (skip) 或 q (quit)

    ▼
ExperienceStore.update(experience)
    └─ 保存更新后的经验到 knowledge.json
        并在 metrics.db 中记录 review_events

    ▼
active 经验自动加入 BM25 检索索引和向量索引
```

#### 6.5 **失效检测流程**（Phase 3）

```
xp watch 命令或定时任务
    │
    ▼
KnowledgeService.check_stale_experiences()
    │
    ├─ Step 1: 获取当前项目的所有 active 经验
    │
    ├─ Step 2: 对每条经验的 related_files 进行 hash 检查
    │   └─ FileWatcher.check_files_changed()
    │       ├─ 计算当前文件 hash
    │       ├─ 与历史 hash 比较
    │       └─ 返回 (是否变更, [变更文件列表])
    │
    ├─ Step 3: 标记失效经验
    │   └─ exp.stale_reason = "Files changed: ..."
    │       exp.status = "archived" (可选)
    │
    └─ Step 4: TTL 检查
        └─ TTLManager.check_experiences()
            ├─ 检查 last_hit_at 距现在是否超过 90 天
            ├─ 若超期，自动归档
            └─ exp.reject_reason = "TTL expired (90 days no hit)"

    ▼
用户提示：
- "以下经验文件已变更，可能需要更新"
- "以下经验 90 天未使用，已自动归档"

    ▼
xp stats 显示变化趋势
```

---

### 7. 完整工作流示意

```
┌─────────────────────────────────────────────────────────────────┐
│                  Agent 工作流程（端到端）                       │
└─────────────────────────────────────────────────────────────────┘

1️⃣ 任务开始
   └─ Agent 调用 search_best_practices
      ├─ Query: "修复 React useEffect 无限循环"
      ├─ Tags: ["react", "hooks"]
      └─ → 返回相关历史经验

2️⃣ 参考经验后执行任务
   └─ Agent 根据检索结果改进代码
      ├─ 避免了之前踩过的坑
      └─ 对话轮数减少 (假设: 3 轮 vs 无经验的 5 轮)

3️⃣ 任务完成后沉淀经验
   └─ Agent 调用 extract_experience
      ├─ task_description: "修复登录后状态不保持"
      ├─ solution_summary: "Token 过期时自动刷新..."
      ├─ key_decisions: "不要在 useEffect 直接 setState..."
      └─ → 生成 pending 经验

4️⃣ 记录会话效果
   └─ Agent 调用 record_session
      ├─ session_id: "session-20260331-001"
      ├─ experience_ids_injected: ["uuid-search-result"]
      ├─ iteration_count: 3
      ├─ had_error_correction: false
      └─ user_accepted: true

5️⃣ 反馈采纳情况
   └─ Agent 调用 record_feedback
      ├─ experience_id: "uuid-search-result"
      ├─ adopted: true
      └─ → 更新经验统计信息

6️⃣ 人工 Review（异步）
   └─ 工程师运行 xp review
      ├─ 查看新的 pending 经验
      ├─ 确认或编辑后激活
      └─ 更新至 knowledge.json

7️⃣ 查看效果
   └─ 工程师运行 xp stats
      ├─ 知识库状态
      ├─ 过程指标（检索命中率）
      ├─ 效果对比（有/无注入的差异）
      └─ → 验证工具是否真的有用

┌──────────────────────────────────────────────────────────────┐
│ 反复迭代：每次完成任务都沉淀经验，知识库不断成长             │
│ 最终：团队经验库逐步完善，Agent 越来越聪明                  │
└──────────────────────────────────────────────────────────────┘
```

---

### 8. 关键设计特点

#### 8.1 **混合检索（Hybrid Search）**
- **BM25**（70% 权重）：精确匹配代码片段、函数名、配置项
- **BGE Embedding**（30% 权重）：理解语义相似但字面不同的表达
- **为什么两者都需要**？
  - BM25 擅长：`getUserById`、`vite.config.ts`、`process.env`
  - Embedding 擅长：`hook 依赖问题` ≈ `useEffect 无限循环`

#### 8.2 **多层隔离**
- **项目隔离**：不同项目经验独立管理
- **多人协作**：云端 PostgreSQL 支持团队共享
- **后端切换**：本地/云端无缝切换，无代码改动

#### 8.3 **自动生命周期管理**
- **文件变更监听**：相关文件改动自动标记失效
- **TTL 管理**：90 天未命中自动归档（防止过期经验）
- **统计驱动**：通过采纳率、命中率等指标自动淘汰低质经验

#### 8.4 **质量保证**
- **双审核**：LLM 提取 + 人工确认
- **去重检测**：相似度 > 0.9 自动合并
- **置信度**：pending/pending edited/active 分别对应不同的权重

#### 8.5 **A/B 测试内置**
- 自动分组：10% 对照组，90% 处理组
- 效果对比：对话轮数、报错率、接受率等
- 数据驱动决策

---

### 9. 部署架构（云端 K8s）

```
┌────────────────────────────────────────────┐
│        快手 K8s 集群（未来部署）          │
├────────────────────────────────────────────┤
│  ┌──────────────┐      ┌──────────────┐   │
│  │ xp-server    │◄────►│ PostgreSQL   │   │
│  │ (Deployment) │      │ (KDB 托管)   │   │
│  │              │      │              │   │
│  │ • MCP Server │      │ • experiences│   │
│  │ • BM25 检索  │      │ • sessions   │   │
│  │ • SSE 推送   │      │ • feedback   │   │
│  │              │      │ • vectors    │   │
│  └────┬─────────┘      └──────────────┘   │
│       │ HTTPS/SSE                         │
└───────┼─────────────────────────────────────┘
        │
    ┌───┴────┬──────────┬─────────────┐
    │         │          │             │
  IDE 1   IDE 2      IDE 3         Local CLI
 (Cursor) (Claude)  (Windsurf)   (xp review)
```

**K8s 配置**：
- **Deployment**：2+ 副本，自动扩缩容
- **Service**：负载均衡
- **ConfigMap**：环境变量配置
- **Secret**：API Key 和数据库密钥

---

### 10. 当前状态与进展

根据 git 日志和文件情况：

| Phase | 状态 | 描述 |
|-------|------|------|
| **Phase 1: MVP** | ✅ 完成 | 本地 BM25 检索、经验提取、人工 review、基础统计 |
| **Phase 2: Embedding** | ✅ 完成 | BGE-small-zh 混合检索、向量存储、质量评分 |
| **Phase 3: 多人共享** | 🔄 进行中 | PostgreSQL 后端、云端同步、项目隔离、文件监听 |
| **Phase 4: K8s 部署** | 📋 计划 | Docker 化、K8s 清单、SSE transport |
| **Phase 5: 高级特性** | 📋 计划 | 权限控制、Web Dashboard、监控告警 |

**最新改动（4cc0f6d）**：
- 经验提取质量重构
- 优化工具描述信息
- 改进数据模型

---

### 总结

XP 是一个**智能、自动化、多人协作**的经验沉淀平台：

1. **自动化**：Agent 自动提取，人工异步审核
2. **智能化**：混合检索（BM25 + Embedding）、自动去重、生命周期管理
3. **协作化**：支持团队多人共享，云端 K8s 部署
4. **可测量**：内置 A/B 测试，效果数据可视化
5. **可扩展**：后端可插拔，云端 Provider 支持多种存储

通过这个系统，团队的经验逐步积累，Agent 越用越聪明，最终实现 **"一次成长，终身受益"**。
