---
session_id: agent-2afb1b6f
date: 2026-04-05
time_start: 2026-04-05T14:42:47.962Z
time_end: 2026-04-05T14:44:30.632Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-05 22:42:47] [user]: 请探索 /Users/lianzimeng/workspace/xp 项目，给出完整的架构概览。需要了解：
1. 项目的整体目录结构（src/ 下各层）
2. 入口点（server.py、cli.py、rest_api.py 的角色）
3. 领域层（domain/）有哪些核心模型
4. 应用层（application/）的 CommandBus、handlers、commands 结构
5. 基础设施层（infrastructure/）的组成
6. 数据流：从用户请求到数据库的完整链路
7. 测试结构

请读取以下文件：
- src/server.py
- src/cli.py
- src/container.py
- src/interfaces/rest_api.py
- src/application/command_bus.py
- src/application/commands.py
- src/application/unit_of_work.py
- src/infrastructure/unit_of_work.py
- src/models.py
- pyproject.toml（了解依赖）

返回完整的架构分析，包括各层职责、关键类、数据流。
[2026-04-05 22:42:50] [assistant]: 我将深入探索这个项目的架构。让我从读取关键文件开始，了解整体结构。
[2026-04-05 22:42:55] [assistant]: 现在继续读取应用层和基础设施层的核心文件：
[2026-04-05 22:42:59] [assistant]: 现在读取模型和接口文件：
[2026-04-05 22:43:03] [assistant]: 现在读取核心的domain模型和postgres后端实现：
[2026-04-05 22:43:07] [assistant]: 现在查看测试结构和其他关键handler：
[2026-04-05 22:43:11] [assistant]: 现在让我查看一些关键的domain模型和处理逻辑：
[2026-04-05 22:43:14] [assistant]: 现在让我查看README和一些架构设计文档，以获得更完整的理解：
[2026-04-05 22:43:18] [assistant]: 现在让我读取一个更详细的handler来了解数据流：
[2026-04-05 22:43:21] [assistant]: 完美！现在让我创建一份完整的架构分析文档。让我读取一个handler来完整理解flow：
[2026-04-05 22:44:30] [assistant]: 现在我有足够的信息了。让我生成完整的架构分析文档：

## 完整的 XP 项目架构分析

基于对整个项目代码的深入分析，以下是完整的架构概览：

---

## 1. 项目整体概况

**项目名称**: XP (Experience Processor) - Agent 最佳实践沉淀工具

**核心目标**: 自动沉淀 Agent 任务经验，建立可检索的知识库，使 Agent 在重复任务中逐次优化

**技术栈**:
- **后端框架**: FastAPI (REST API) + MCP (Model Context Protocol)
- **数据库**: PostgreSQL (+ pgvector 向量存储)
- **异步方案**: asyncio + asyncpg
- **向量化**: Sentence-transformers
- **CLI**: Python asyncio
- **版本**: 0.2.0

---

## 2. 目录结构

```
src/
├── server.py              # MCP 服务器入口，暴露 search/save/feedback 工具
├── cli.py                 # CLI 命令入口，本地经验管理
├── container.py           # 依赖注入容器，构建 CommandBus
├── models.py              # 数据模型（Experience, Session, Feedback 等）
│
├── application/           # 应用层
│   ├── command_bus.py     # CQRS 命令分发总线
│   ├── commands.py        # 所有命令定义（dataclass）
│   ├── unit_of_work.py    # 抽象仓储接口
│   └── handlers/          # 命令处理器
│       ├── search_v2_handler.py        # 【搜索】v2 版本（主要使用）
│       ├── save_handler.py             # 【保存】新经验到库
│       ├── feedback_v2_handler.py      # 【反馈】用户反馈处理
│       ├── extract_handler.py          # 【提取】从任务提取经验
│       ├── session_handler.py          # 【会话】记录 Agent 会话
│       ├── infer_adoption_handler.py   # 【推断】自动推断采纳情况
│       └── ...（其他处理器）
│
├── domain/                # 领域层
│   ├── experience.py      # 向量相似度计算、重复检测
│   ├── extraction.py      # 提取服务核心逻辑
│   ├── quality.py         # 经验质量评分
│   ├── search_v2.py       # 作用域加权、技术栈提升
│   ├── manifest_parser.py # 项目清单解析（package.json/pyproject.toml/go.mod）
│   ├── feedback_v2.py     # 反馈的信心度更新
│   ├── save_domain.py     # 保存时的初始化逻辑
│   └── metadata.py        # 元数据推断（技术栈、场景、类型）
│
├── infrastructure/        # 基础设施层
│   ├── unit_of_work.py    # UnitOfWork 实现
│   ├── backends/
│   │   ├── base.py        # 存储后端接口
│   │   └── postgres.py    # PostgreSQL 具体实现
│   └── ...
│
└── interfaces/
    └── rest_api.py        # FastAPI REST 接口
```

---

## 3. 入口点角色分析

### 3.1 server.py - MCP 服务器

**职责**: 作为 MCP (Model Context Protocol) 服务器，为 Agent IDE 提供工具

**核心工具**:
1. **search** - 任务开始前检索相关经验
   - 输入: task_description, project_id, project_manifest
   - 返回: search_event_id + 相关经验列表

2. **save** - 任务完成后保存经验
   - 输入: task_description, solution, key_decisions, tags, outcome
   - 返回: experience_id, quality_score, 状态

3. **feedback** - 用户反馈检索结果质量
   - 输入: search_event_id, helpful_ids, unhelpful_ids
   - 更新: 经验的信心度

**启动**: `xp-server` 命令

```
MCP Protocol (stdio)
  ↓
app.call_tool()
  ↓
CommandBus.dispatch()
  ↓
Handler 处理
```

### 3.2 cli.py - 本地 CLI 管理

**职责**: 提供本地命令行工具，进行知识库管理

**主要命令**:
- `xp add` - 交互式添加新经验
- `xp review` - 审核待确认经验（带编辑功能）
- `xp import <文件>` - 导入 Markdown 种子经验
- `xp search` - 本地搜索
- `xp stats` - 查看效果统计（会话数、采纳率）
- `xp analyze` - 质量分析报告
- `xp show <id>` - 查看经验详情
- `xp edit <id>` - 编辑经验
- `xp init --project <name>` - 初始化项目
- `xp migrate` - 为旧数据生成向量

### 3.3 interfaces/rest_api.py - REST API

**职责**: 提供 RESTful 接口供服务调用

**主要端点**:
- `POST /api/experiences` - 提取经验
- `GET /api/experiences/search` - 搜索
- `PATCH /api/experiences/{id}` - 审核（activate/archive）
- `POST /api/feedback` - 记录反馈
- `POST /api/sessions` - 记录会话

---

## 4. 领域层 (domain/) - 核心模型

### 4.1 Experience 模型

```python
@dataclass
class Experience:
    id: str                          # UUID
    type: ExperienceType             # bugfix/feature/pattern
    level: ExperienceLevel           # L1/L2/L3
    title: str                       # 自动生成或用户编辑
    tags: list[str]                  # 技术栈标签
    problem: str                     # 问题描述
    solution: str                    # 解决方案
    key_decisions: str               # 【关键价值】关键决策/踩坑点
    confidence: float                # [0.0, 1.0] 信心度
    status: ExperienceStatus         # pending/active/archived
    source: ExperienceSource         # agent/manual
    created_at: str                  # ISO 时间戳
    metadata: ExperienceMetadata     # 结构化元数据
    scope_type: ScopeType            # project/business/team
    scope_id: str | None             # 作用域 ID（用于多层级搜索）
```

### 4.2 核心计算逻辑

#### 4.2.1 质量评分 (quality.py)

```python
compute_quality_score():
  - key_decisions ≥50字 → +40分
  - solution ≥50字 → +20分
  - 包含相关文件 → +15分
  - 有技术栈标签 → +15分
  - 无高重复 → +10分
  ────────────────
  总分 100 分
```

**用途**: 决定是否自动激活为 ACTIVE 状态

#### 4.2.2 向量相似度 & 重复检测 (experience.py)

```python
check_duplicate(query_vec, all_vecs, all_ids):
  → cosine_similarity
  → if score ≥ 0.85: 警告重复
```

**防止知识库重复污染**

#### 4.2.3 信心度更新 (feedback_v2.py)

```python
compute_confidence_after_helpful(confidence):
  → min(confidence + 0.1, 1.0)

compute_confidence_after_unhelpful(confidence):
  → max(confidence - 0.15, 0.0)
  → if confidence ≤ 0.2: 自动归档
```

**自动管理经验生命周期**

#### 4.2.4 清单解析 (manifest_parser.py)

**支持格式**:
- package.json (JavaScript/Node)
- pyproject.toml (Python)
- go.mod (Go)

**提取内容**:
- language: 编程语言
- frameworks: [react, nextjs, fastapi ...]
- top_dependencies: 前 10 个依赖

**用途**: 技术栈匹配、作用域权重调整

---

## 5. 应用层 (application/)

### 5.1 CQRS 架构

所有操作都通过 **Command Bus** 进行分发：

```python
class CommandBus:
    def register(command_type, handler_fn)
    async def dispatch(command) → result
```

**模式**:
1. 用户请求 → 创建 Command 对象
2. CommandBus.dispatch() → 查找对应 Handler
3. Handler 处理 → 返回结果

### 5.2 核心 Command 类型

| Command | Handler | 用途 |
|---------|---------|------|
| SearchV2Command | SearchV2Handler | 搜索相关经验 |
| SaveCommand | SaveHandler | 保存新经验 |
| FeedbackV2Command | FeedbackV2Handler | 记录用户反馈 |
| ExtractExperienceCommand | ExtractHandler | 从任务提取经验 |
| RecordSessionCommand | SessionHandler | 记录 Agent 会话 |
| InferAdoptionCommand | InferAdoptionHandler | 自动推断采纳 |
| GetStatsCommand | StatsHandler | 获取效果统计 |
| AnalyzeQualityCommand | AnalyzeHandler | 质量分析 |

### 5.3 Handler 实现示例

#### SearchV2Handler (主要搜索逻辑)

```
输入: SearchV2Command
  ↓
1. 解析 project_manifest (parse_manifest)
   - 提取 language, frameworks, top_dependencies
  ↓
2. 生成查询向量
   - embed_text(task_description + language + dependencies)
  ↓
3. 多层次检索
   - 按 scope_type 分类: project/business/team
   - 使用 cosine_similarity 计算相似度
  ↓
4. 评分调整
   - apply_scope_weight(): project (1.2x) > business (1.0x) > team (0.9x)
   - apply_tech_stack_boost(): 技术栈匹配 +15%
  ↓
5. 去重与排序
   - merge_and_deduplicate(threshold=0.0)
   - 返回 top_k 结果
  ↓
输出: {
  search_event_id: UUID,
  results: [{
    id, title, problem, solution,
    score, source_level, tags
  }]
}
```

#### SaveHandler (保存经验逻辑)

```
输入: SaveCommand
  ↓
1. 质量评分
   - compute_save_quality_score()
  ↓
2. 初始化信心度
   - outcome='success' → confidence=0.7
   - outcome='partial' → confidence=0.5
   - outcome='failed' → confidence=0.3
  ↓
3. 重复检测
   - embed_text(task + solution + decisions)
   - check_duplicate() 与现有经验对比
   - 相似度≥0.85 → 返回警告
  ↓
4. 保存到数据库
   - create Experience 对象
   - save vector to pgvector
  ↓
5. 状态判定
   - quality_score ≥ 80 → 自动激活 (ACTIVE)
   - 否则 → 待审核 (PENDING)
  ↓
输出: {
  experience_id, status, quality_score,
  duplicate_warning (optional)
}
```

#### InferAdoptionHandler (自动推断采纳)

```
输入: InferAdoptionCommand
  - session_id, final_response, experience_ids_injected
  ↓
1. 生成响应向量
   - embed_text(final_response)
  ↓
2. 获取注入经验的向量
   - 如果缺失，则计算并保存
  ↓
3. 相似度计算
   - cosine_similarity(response_vec, exp_vecs)
  ↓
4. 推断采纳
   - score ≥ 0.60 → adopted=True
   - 更新 experience.confidence
  ↓
5. 自动归档
   - if confidence ≤ AUTO_ARCHIVE_ADOPTION_THRESHOLD → ARCHIVED
  ↓
输出: [{
  experience_id, adopted, confidence,
  match_rate
}]
```

### 5.4 UnitOfWork 模式

```python
class AbstractUnitOfWork:
    experiences: AbstractExperienceStore   # 经验 CRUD
    vectors: AbstractVectorStore           # 向量存储
    metrics: AbstractMetricsStore          # 指标记录
    analytics: AbstractAnalyticsStore      # 分析统计

async with uow_factory() as uow:
    await uow.experiences.aadd(exp)
    await uow.vectors.asave_vector(exp_id, vec)
    await uow.analytics.record_session(session)
```

**事务管理**:
- 获取 DB 连接
- 执行操作
- 自动提交或回滚

---

## 6. 基础设施层 (infrastructure/)

### 6.1 PostgreSQL 后端 (postgres.py)

**数据库架构**:

```sql
experiences (核心表)
├── id (TEXT PRIMARY KEY)
├── type, level, status, source
├── title, tags (JSONB), problem, solution, key_decisions
├── confidence (FLOAT)
├── metadata (JSONB)
├── embedding (vector(512))  ← pgvector 向量存储
└── scope_type, scope_id     ← 多层级支持

sessions (会话表)
├── session_id (PRIMARY KEY)
├── task_description, experience_ids_injected (JSONB)
├── iteration_count, had_error_correction
├── ab_test_group, ab_test_result_shown
└── created_at

feedback (反馈表)
├── id (SERIAL PRIMARY KEY)
├── experience_id, adopted (BOOLEAN)
├── reason, created_at
└── (foreign key) experience_id

experience_stats (统计表)
├── experience_id (PRIMARY KEY)
├── hit_count, adopted_count, rejected_count
├── adoption_rate (computed)
└── last_used_at, last_adopted_at

search_events, feedback_events (v2 用)
├── 记录每次搜索和反馈
├── 用于数据分析和推荐系统
```

**向量存储**: pgvector 扩展
- 512 维向量（sentence-transformers 默认）
- L2 距离计算

**异步 API**:

```python
async def async_add_experience(exp)
async def async_get_experience(exp_id)
async def async_list_by_status(status)
async def async_save_vector(exp_id, vector)
async def async_get_all_vectors(exp_ids)
async def async_record_session(session)
async def async_get_stats(since_days)
```

### 6.2 连接生命周期

```
create_pool(dsn)
  ↓
acquire() → connection
  ↓
execute SQL
  ↓
release() → 返回池
  ↓
close() → 关闭池
```

---

## 7. 数据流：用户请求到数据库

### 场景 A: Agent 搜索经验

```
┌─────────────────────────────────────────────────────────────┐
│ MCP Client (Claude/其他 Agent)                              │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ search(task_description, project_id, manifest)
┌─────────────────────────────────────────────────────────────┐
│ server.py → call_tool("search")                             │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ SearchV2Command(...)
┌─────────────────────────────────────────────────────────────┐
│ CommandBus.dispatch()                                       │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ SearchV2Handler.handle()
┌─────────────────────────────────────────────────────────────┐
│ 1. parse_manifest()                                         │
│    → language, frameworks, top_dependencies                 │
│ 2. embed_text(task + language + deps)                       │
│    → query_vec (512-dim)                                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ UnitOfWork.experiences.alist_by_status("active")
┌─────────────────────────────────────────────────────────────┐
│ PostgreSQL Query                                            │
│ SELECT * FROM experiences WHERE status='active'             │
│                                                             │
│ 返回: Experience 对象列表                                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ 按 scope_type 分组 (project/business/team)
┌─────────────────────────────────────────────────────────────┐
│ 3. 向量相似度计算 (for each experience)                      │
│    - 获取 embedding 向量                                     │
│    - cosine_similarity(query_vec, exp_vec)                  │
│    - apply_scope_weight() + apply_tech_stack_boost()        │
│    - 得分: project 更高权重                                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ merge_and_deduplicate(top_k=5)
┌─────────────────────────────────────────────────────────────┐
│ 4. 按得分排序，返回 top_k 结果                                │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ 返回 JSON
┌─────────────────────────────────────────────────────────────┐
│ {                                                           │
│   "search_event_id": "uuid-xxx",                            │
│   "results": [                                              │
│     {                                                       │
│       "id": "exp-001",                                      │
│       "title": "Fix NPE in user service",                   │
│       "problem": "...",                                     │
│       "solution": "...",                                    │
│       "key_decisions": "...",                               │
│       "score": 0.82,                                        │
│       "source_level": "project"                             │
│     },                                                      │
│     ...                                                     │
│   ]                                                         │
│ }                                                           │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ 返回给 MCP 客户端
┌─────────────────────────────────────────────────────────────┐
│ Agent 展示给用户或自动使用经验                               │
└─────────────────────────────────────────────────────────────┘
```

### 场景 B: Agent 保存新经验

```
┌─────────────────────────────────────────────────────────────┐
│ save(task_description, solution, key_decisions, tags,       │
│      project_id, outcome, search_event_id)                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ SaveCommand(...)
┌─────────────────────────────────────────────────────────────┐
│ CommandBus → SaveHandler.handle()                           │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ 1. 质量评分
                     │    compute_save_quality_score()
                     │    → score (0-100)
                     │
                     ↓ 2. 初始化信心度
                     │    initial_confidence_for_outcome()
                     │    → success=0.7, partial=0.5, failed=0.3
                     │
                     ↓ 3. 生成向量
                     │    embed_text(task+solution+decisions)
                     │    → new_vec
┌─────────────────────────────────────────────────────────────┐
│ 4. 重复检测                                                  │
│    - 获取项目内所有经验向量                                   │
│    - check_duplicate(new_vec, existing_vecs)                │
│    - if similarity ≥ 0.85 → warning                         │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ 5. 保存到 PostgreSQL
                     │    INSERT INTO experiences (...)
                     │    INSERT INTO search_events (...)  (如有)
                     │
                     ↓ 6. 保存向量
                     │    INSERT INTO embeddings (...)
                     │    UPDATE experiences SET embedding=...
┌─────────────────────────────────────────────────────────────┐
│ 7. 决策状态                                                  │
│    if quality_score ≥ 80 → status='active'                  │
│    else → status='pending'                                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ 返回结果
┌─────────────────────────────────────────────────────────────┐
│ {                                                           │
│   "experience_id": "uuid-yyy",                              │
│   "status": "active" | "pending",                           │
│   "quality_score": 85,                                      │
│   "duplicate_warning": "..."  (optional)                    │
│ }                                                           │
└─────────────────────────────────────────────────────────────┘
```

### 场景 C: 用户反馈与信心度更新

```
┌─────────────────────────────────────────────────────────────┐
│ feedback(search_event_id, helpful_ids, unhelpful_ids)       │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ FeedbackV2Command(...)
┌─────────────────────────────────────────────────────────────┐
│ CommandBus → FeedbackV2Handler.handle()                     │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ 1. 验证反馈 IDs
                     │    (必须来自对应 search_event 的结果)
                     │
                     ↓ 2. 更新有帮助的经验
                     │    for each helpful_id:
                     │      exp.confidence += 0.1  (min 1.0)
                     │
                     ↓ 3. 更新无帮助的经验
                     │    for each unhelpful_id:
                     │      exp.confidence -= 0.15
                     │      if confidence ≤ 0.2 → ARCHIVED
                     │
                     ↓ 4. 保存到数据库
                     │    UPDATE experiences SET confidence=...
                     │    INSERT INTO feedback_events (...)
┌─────────────────────────────────────────────────────────────┐
│ 自动生命周期管理:                                             │
│ - 高采纳 (confidence > 0.9) → 推荐晋升                        │
│ - 低采纳 (confidence < 0.2) → 自动归档                       │
└─────────────────────────────────────────────────────────────┘
```

### 场景 D: 自动推断采纳 (InferAdoption)

```
┌─────────────────────────────────────────────────────────────┐
│ Agent 完成任务，生成最终回应                                   │
│ finalize_task(..., final_response, experience_ids_injected)  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ InferAdoptionCommand(...)
┌─────────────────────────────────────────────────────────────┐
│ CommandBus → InferAdoptionHandler.handle()                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ 1. 生成响应向量
                     │    embed_text(final_response)
                     │    → response_vec
                     │
                     ↓ 2. 获取注入经验的向量
                     │    (如缺失，计算并保存)
                     │
                     ↓ 3. 相似度计算
                     │    cosine_similarity(response_vec, exp_vecs)
                     │    for each exp:
                     │      score = dot_product / (norm1 * norm2)
                     │
                     ↓ 4. 推断采纳与信心度
                     │    if score ≥ 0.60:
                     │      adopted = True
                     │      confidence += 0.1
                     │    else:
                     │      adopted = False
                     │      confidence -= 0.15
                     │
                     ↓ 5. 自动归档
                     │    if confidence ≤ AUTO_ARCHIVE_THRESHOLD
                     │      → status = ARCHIVED
                     │
                     ↓ 6. 记录反馈
                     │    INSERT INTO feedback_events
                     │    UPDATE experiences
┌─────────────────────────────────────────────────────────────┐
│ 输出: [{
│   experience_id, adopted, confidence, match_rate
│ }, ...]
└─────────────────────────────────────────────────────────────┘
```

---

## 8. 测试结构

```
tests/
├── unit/                          # 单元测试（最多）
│   ├── test_command_bus.py        # 命令分发
│   ├── test_search_v2_handler.py  # 搜索逻辑
│   ├── test_save_handler.py       # 保存逻辑
│   ├── test_feedback_v2_handler.py # 反馈逻辑
│   ├── test_experience.py         # 向量计算
│   ├── test_quality_score.py      # 质量评分
│   ├── test_manifest_parser.py    # 清单解析
│   ├── test_backends.py           # 存储后端
│   └── ...
│
├── integration/                   # 集成测试
│   ├── test_experience_handler.py
│   ├── test_feedback_handler.py
│   └── test_rollback.py
│
├── e2e/                           # 端到端测试
│   ├── test_rest_api.py
│   ├── test_extract_api.py
│   ├── test_search_api.py
│   └── test_health.py
│
├── characterization/              # 特征化测试
│   ├── test_extraction.py
│   ├── test_search.py
│   └── test_feedback.py
│
├── contract/                      # 契约测试
│   ├── test_commands.py
│   └── test_events.py
│
└── helpers/
    ├── fake_uow.py               # 模拟 UnitOfWork
    └── test_fake_uow_smoke.py
```

**测试框架**: pytest + pytest-asyncio

---

## 9. 核心设计模式

### 9.1 架构模式

| 模式 | 实现 | 用途 |
|------|------|------|
| **CQRS** | CommandBus | 命令与查询分离 |
| **UnitOfWork** | AbstractUnitOfWork | 事务管理与仓储聚合 |
| **Repository** | ExperienceStore, VectorStore | 数据访问抽象 |
| **Adapter** | PostgresBackend | 存储适配 |
| **Factory** | container.build_command_bus() | 依赖注入 |
| **Strategy** | 多个 Handler | 不同命令的处理策略 |

### 9.2 流程控制

```
1. 【搜索】数据流
   manifest_parser → 清单解析
   embedding → 向量生成
   scope_weight + tech_stack_boost → 评分调整
   merge_and_deduplicate → 结果排序

2. 【保存】数据流
   quality_score → 决策激活/待审
   initial_confidence → 根据 outcome 初始化
   check_duplicate → 防止污染
   save vector → pgvector 存储

3. 【反馈】数据流
   validate_feedback_ids → 验证来源
   compute_confidence_* → 更新信心度
   should_archive → 自动管理生命周期

4. 【推断】数据流
   embed_text → 生成向量
   cosine_similarity → 相似度计算
   confidence_update → 自动学习
   auto_archive → 清理僵尸经验
```

---

## 10. 关键配置与常数

### 10.1 环境变量

```bash
DATABASE_URL=postgresql://user:pass@localhost/xp
OPENAI_API_KEY=sk-...
XP_LLM_API_BASE=https://api.openai.com/v1
XP_LLM_MODEL=gpt-4o-mini
XP_ADMIN_KEY=...
XP_USER_KEY=...
```

### 10.2 阈值常数 (constants.py)

```python
DUPLICATE_SIMILARITY_THRESHOLD = 0.85
AUTO_ARCHIVE_ADOPTION_THRESHOLD = 0.2
QUALITY_SCORE_ACTIVE_THRESHOLD = 80
```

### 10.3 作用域权重 (search_v2.py)

```python
_SCOPE_WEIGHTS = {
    "project": 1.2,    # 当前项目经验 +20%
    "business": 1.0,   # 业务线经验
    "team": 0.9,       # 团队经验 -10%
}
```

---

## 11. 数据生命周期

```
提取阶段 (Extraction)
├─ 质量评分 ≥80 → 自动激活 (ACTIVE)
└─ 质量评分 <80 → 待审核 (PENDING)
    │
    ↓ 人工 review
    │
    ├─ 确认 → ACTIVE (confidence=0.8)
    ├─ 编辑后确认 → ACTIVE (confidence=1.0)
    └─ 拒绝 → ARCHIVED (reject_reason记录)

使用阶段 (Active)
├─ 被搜索命中
├─ 被 Agent 注入到推理上下文
├─ 用户反馈有帮助
│   └─ confidence += 0.1
└─ 用户反馈无帮助 或 自动推断不采纳
    └─ confidence -= 0.15
        └─ if confidence ≤ 0.2 → ARCHIVED

生命周期维护
├─ TTL 检查: 90天未被命中 → ARCHIVED
├─ 文件变更检测: 相关代码改动 → 标记为陈旧
└─ 僵尸清理: 从未被命中或采纳率为0 → 定期清理
```

---

## 12. 系统规模

### 当前版本特性

✅ **已完成**:
- PostgreSQL 存储后端
- 向量检索 (pgvector)
- CQRS 架构
- 多层级经验管理 (project/business/team)
- 自动质量评分
- 信心度动态更新
- MCP 工具暴露
- REST API
- CLI 管理工具

🚧 **规划中**:
- 分布式向量检索
- 高级推荐系统 (协同过滤)
- A/B 测试完整支持
- 知识库权限管理
- 云端同步 (Supabase/Weaviate)

---

## 13. 架构优势

1. **清晰分层**: Domain ← Application ← Infrastructure
2. **解耦设计**: 依赖反转，接口驱动
3. **可测试性**: UnitOfWork 易于 Mock，Handler 独立
4. **可扩展性**: 新 Handler 只需注册，无需修改核心
5. **异步优先**: 全异步流程，支持高并发
6. **多租户**: Scope 系统支持 project/business/team 三层
7. **自适应学习**: 反馈循环自动优化信心度

---

## 14. 快速参考表

| 层级 | 核心文件 | 职责 |
|------|---------|------|
| **Presentation** | server.py, cli.py, rest_api.py | 用户交互入口 |
| **Application** | command_bus.py, handlers/* | 业务流程编排 |
| **Domain** | experience.py, quality.py, search_v2.py | 业务规则与计算 |
| **Infrastructure** | postgres.py, unit_of_work.py | 数据访问与持久化 |
| **Data** | models.py | 数据结构定义 |

---

这就是 XP 项目的完整架构概览！整个系统以 **命令驱动 + 向量检索 + 反馈学习** 为核心，建立了一套自适应的知识管理系统。
