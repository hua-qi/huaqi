---
session_id: agent-3a84d263
date: 2026-04-07
time_start: 2026-04-07T03:07:12.454Z
time_end: 2026-04-07T03:08:48.626Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-07 11:07:12] [user]: 请探索 /Users/lianzimeng/workspace/xp 这个项目的整体结构，包括：
1. 目录结构（主要目录和文件）
2. 技术栈（package.json、配置文件等）
3. 核心模块和功能
4. 当前的数据存储方式
5. 架构特点（是否是本地工具、CLI、服务等）

请给出详细的总结，不需要写代码。
[2026-04-07 11:07:15] [assistant]: 我来帮你探索这个项目的整体结构。让我从几个方面来深入了解。
[2026-04-07 11:07:21] [assistant]: 现在让我读取 pyproject.toml 和探索更多核心结构：
[2026-04-07 11:07:29] [assistant]: 让我探索核心源代码的结构：
[2026-04-07 11:07:32] [assistant]: 现在让我查看基础设施层和配置信息：
[2026-04-07 11:07:36] [assistant]: 让我检查应用层的核心命令和处理器：
[2026-04-07 11:07:39] [assistant]: 让我检查一些重要的域模型文件：
[2026-04-07 11:07:42] [assistant]: 让我查看一些设计文档来了解架构决策：
[2026-04-07 11:07:45] [assistant]: 现在让我查看一些处理器来理解业务流程：
[2026-04-07 11:07:48] [assistant]: 让我了解更多的架构细节：
[2026-04-07 11:08:48] [assistant]: 完美！现在我有了足够的信息。让我生成一份全面的总结：

## 📋 XP 项目整体结构详细总结

基于对代码库的深入探索，这是一个**团队级 Agent 最佳实践沉淀工具**，我将从多个维度进行详细分析：

---

### **1. 项目定位与核心价值**

**名称**: XP (Experience Processor)  
**定位**: 让 Agent 在任何任务中"越用越聪明"的最佳实践沉淀工具  
**版本**: 0.3.0（alpha 阶段）  
**核心理念**: 自动沉淀→智能检索→人工审核→效果追踪

**四大核心特性**:
- 自动沉淀：任务完成后自动提取总结最佳实践
- 智能检索：基于向量搜索的经验匹配，任务开始前召回相关知识
- 人工审核：CLI 工具快速 review 机器总结内容，保证质量
- 效果追踪：量化工具使用效果，追踪研发效能提升

---

### **2. 目录结构与主要模块**

```
xp/
├── src/                          # 核心源代码
│   ├── application/              # 应用层（DDD模式）
│   │   ├── commands.py           # 命令定义（60+ 个命令）
│   │   ├── command_bus.py        # 命令总线
│   │   ├── handlers/             # 12+ 个处理器（异步处理命令）
│   │   │   ├── search_handler.py
│   │   │   ├── save_handler.py
│   │   │   ├── feedback_v2_handler.py
│   │   │   ├── stats_handler.py
│   │   │   └── promotion_handler.py （升层候选扫描）
│   │   └── unit_of_work.py       # UoW 模式
│   ├── domain/                   # 域模型层（业务逻辑）
│   │   ├── experience.py         # 经验相似度检测
│   │   ├── search.py            # 搜索服务
│   │   ├── search_v2.py         # v2 搜索（支持多层级）
│   │   ├── feedback.py          # 反馈处理
│   │   ├── quality.py           # 质量评分算法
│   │   ├── metadata.py          # 元数据推断
│   │   ├── promotion.py         # 升层决策逻辑
│   │   ├── health.py            # 系统健康状态检测
│   │   ├── constants.py         # 常量定义
│   │   └── events.py            # 领域事件
│   ├── infrastructure/           # 基础设施层
│   │   ├── backends/
│   │   │   ├── base.py          # 存储后端接口
│   │   │   └── postgres.py      # PostgreSQL 后端实现
│   │   └── unit_of_work.py      # UoW 实现
│   ├── interfaces/               # 外层接口
│   │   └── rest_api.py          # REST API 路由
│   ├── cli.py                   # CLI 命令行工具
│   ├── server.py                # MCP Server（主要入口）
│   ├── models.py                # 数据模型定义
│   ├── config.py                # 配置管理
│   ├── embeddings.py            # 向量嵌入抽象
│   ├── edge_client.py           # 远端服务客户端
│   ├── container.py             # IoC 容器（依赖注入）
│   ├── auth.py                  # 认证模块
│   ├── file_watcher.py          # 文件监听与 TTL 管理
│   ├── project_config.py        # 项目配置管理
│   └── __init__.py
├── tests/                        # 完整的测试套件
│   ├── unit/                    # 单元测试 (40+ 个)
│   ├── integration/             # 集成测试
│   ├── e2e/                     # 端到端测试
│   ├── characterization/        # 特征化测试
│   ├── contract/                # 契约测试
│   └── helpers/                 # 测试辅助（fake_uow）
├── docs/                        # 完整的文档体系
│   ├── design/adr/             # 架构决策记录（4 个 ADR）
│   ├── designs/                # 详细设计文档
│   ├── features/               # 功能设计
│   ├── guides/
│   │   ├── dev/               # 开发者指南
│   │   └── user/              # 用户指南
│   └── plans/                 # 实现计划
├── scripts/                    # 辅助脚本
│   ├── migrate_to_postgres.py
│   └── migrate_knowledge_to_server.py
├── supabase/                   # Supabase 配置
├── pyproject.toml             # Python 项目配置
├── main.py                    # 快速启动脚本
└── .env.example               # 环境变量示例
```

---

### **3. 技术栈详解**

#### **核心依赖**:
```python
# pyproject.toml 中的关键依赖

# MCP 框架相关
mcp[cli]>=1.0.0                # Model Context Protocol - 与 Claude、Cursor 等集成
python-dotenv>=1.0.0           # 环境变量管理

# AI/ML 能力
sentence-transformers>=2.2.0   # 本地向量嵌入 (BGE-small-zh)
numpy>=1.24.0                  # 数学计算库

# 数据库
asyncpg>=0.29.0                # 异步 PostgreSQL 驱动
pgvector>=0.2.0                # PostgreSQL 向量存储扩展

# 网络框架
fastapi>=0.110.0               # Web 框架
uvicorn>=0.29.0                # ASGI 服务器
httpx>=0.27.0                  # 异步 HTTP 客户端

# 测试
pytest-asyncio>=1.3.0          # 异步测试支持
```

#### **架构特点**:
- **异步首选**: 100% async/await，基于 asyncio
- **向量优先**: 使用 pgvector 存储嵌入向量，支持相似度搜索
- **PostgreSQL 单一后端**: 不再支持本地文件存储（自 v0.6.0）
- **Sentence Transformers**: 本地运行 BGE-small-zh 模型（512 维向量）
- **MCP 标准**: 完全遵循 Model Context Protocol 规范

---

### **4. 核心数据模型**

#### **Experience（经验）**:
```python
@dataclass
class Experience:
    id: str                          # 唯一标识
    type: ExperienceType             # bugfix/feature/pattern
    level: ExperienceLevel           # L1/L2/L3（质量等级）
    title: str                       # 标题
    tags: list[str]                  # 技术栈标签
    problem: str                     # 问题描述
    solution: str                    # 解决方案 (≥50字)
    key_decisions: str               # 关键决策 (≥30字)
    confidence: float                # 置信度 0-1
    status: ExperienceStatus         # pending/active/archived
    source: ExperienceSource         # agent/manual
    created_at: str                  # 创建时间
    
    # 层级支持（v0.6.0+）
    scope_type: ScopeType            # project/business/team
    scope_id: str                    # 所属项目/业务/团队 ID
    promoted_to: str                 # 升层后的新经验 ID
    demoted_from: str                # 来自哪个更高层的经验
    
    # 元数据
    metadata: ExperienceMetadata     # tech_stack, problem_type, scene
    
    # 效果追踪
    recall_count: int                # 被召回次数
    adoption_rate: float             # 采纳率
    last_hit_at: str                 # 最后使用时间
    
    # 其他
    related_files: list[str]         # 相关文件路径
    file_hashes: dict                # 文件内容 hash（用于过期检测）
```

#### **三层知识层级体系**（v0.6.0 重大特性）:
```
Team（全团队，最通用）
  ↓
Business（业务线，中等通用）
  ↓
Project（单个 git repo，具体场景）
```

- **Project 层**: Agent 自动沉淀，同 repo 所有成员共享
- **Business 层**: 人工从 Project 升层，同业务所有项目共享
- **Team 层**: 人工从 Business 升层，整个团队通用

#### **Session（会话）**:
```python
@dataclass
class Session:
    session_id: str
    task_description: str
    experience_ids_injected: list[str]    # 召回的经验 ID
    iteration_count: int                   # 迭代次数
    had_error_correction: bool             # 是否经过错误修正
    user_accepted: bool                    # 用户是否接受结果
    ab_test_group: str                     # 对照组/实验组
    ab_test_result_shown: bool
    created_at: str
```

#### **Feedback（反馈）**:
```python
@dataclass
class Feedback:
    id: int
    experience_id: str
    adopted: bool               # 经验是否有用
    reason: str                # 反馈原因
    created_at: str
```

---

### **5. 数据存储方式详解**

#### **PostgreSQL 数据库架构**:

**主要表**:
```sql
-- 核心表
experiences              # 经验知识库
├── id (TEXT PRIMARY KEY)
├── type, level, status, source
├── title, tags (JSONB), problem, solution, key_decisions
├── confidence, metadata (JSONB)
├── scope_type, scope_id      # 新增：层级支持
├── promoted_to, demoted_from # 新增：升层追踪
├── recall_count, adoption_rate # 新增：效果指标
├── embedding (vector(512))   # pgvector：512维BGE向量
└── file_hashes (JSONB)      # 文件哈希

sessions                 # 会话记录
├── session_id (PRIMARY KEY)
├── task_description, experience_ids_injected (JSONB)
├── iteration_count, had_error_correction
├── ab_test_group, ab_test_result_shown
└── created_at

feedback                 # 反馈数据
├── id (SERIAL PRIMARY KEY)
├── experience_id, adopted, reason
└── created_at

experience_stats        # 统计汇总
├── experience_id (PRIMARY KEY)
├── hit_count, adopted_count, rejected_count
├── adoption_rate
└── last_used_at, last_adopted_at

-- 层级支持表（v0.6.0+）
teams                  # 团队
├── id (PRIMARY KEY)
├── name, owner_email
└── created_at

businesses             # 业务线
├── id (PRIMARY KEY)
├── team_id (FOREIGN KEY)
├── name, owner_email
└── created_at

projects               # 项目
├── id (PRIMARY KEY)
├── business_id (FOREIGN KEY)
├── name
└── created_at

project_business       # 多对多关联
├── project_id (FK)
└── business_id (FK)

business_team          # 多对多关联
├── business_id (FK)
└── team_id (FK)

promotion_candidates   # 升层候选
├── id (PRIMARY KEY)
├── source_exp_id, target_scope_type, target_scope_id
├── score, status, ignored_at
└── created_at

user_tokens            # 用户认证令牌
├── user_id (PRIMARY KEY)
├── token (UNIQUE)
└── created_at
```

#### **向量存储**:
- **向量模型**: Sentence Transformers BGE-small-zh-v1.5
- **维度**: 512 维
- **存储方式**: PostgreSQL pgvector 扩展
- **相似度算法**: 余弦相似度
- **相似度阈值**: 
  - 重复检测: 0.85+
  - 搜索返回: 0.5+

#### **向量生成流程**:
```
文本输入 (task + solution + key_decisions)
  ↓
BGE 模型编码 (本地或云端)
  ↓
512 维浮点数向量
  ↓
归一化处理
  ↓
存入 PostgreSQL pgvector
```

#### **缓存策略**:
- **嵌入缓存**: `~/.cache/xp/embeddings/` (Sentence Transformers 自动)
- **配置缓存**: `~/.xp/config.json` (用户配置)
- **Token 管理**: `.env` 文件或数据库 user_tokens 表

---

### **6. 核心业务流程**

#### **A. 经验搜索流程（Search）**:
```
Agent 调用 search(task_description, project_manifest)
  ↓
1. 嵌入 task_description
2. 查询数据库中相同项目的所有 active 经验
3. 过滤过期经验 (TTL 检查 last_hit_at)
4. A/B 测试：10% 概率返回空结果 (control group)
5. 计算余弦相似度
6. 返回 top-3 相似经验
7. 记录 session_id 供后续使用
  ↓
返回结果: [
  {
    id, title, problem, solution, key_decisions,
    tags, confidence, similarity_score
  },
  ...
]
```

#### **B. 经验沉淀流程（Save）**:
```
Agent 调用 save(task_description, solution, key_decisions, tags, outcome)
  ↓
1. 计算质量评分 (solution 长度, key_decisions 质量, outcome)
2. 计算初始置信度 (success: 0.8, partial: 0.6, failed: 0.4)
3. 嵌入 (task + solution + key_decisions)
4. 重复检测：与同项目经验比对相似度
5. 若相似度 > 0.85，警告可能重复
6. 生成唯一 ID (UUID)
7. 存入 PostgreSQL experiences 表
8. 存入向量到 pgvector
9. 质量评分 ≥ 80 → ACTIVE，否则 PENDING
  ↓
返回: {
  experience_id: str,
  status: "active" | "pending",
  quality_score: float,
  duplicate_warning: str?
}
```

#### **C. 反馈流程（Feedback）**:
```
Agent 调用 feedback(experience_id, adopted, reason)
  ↓
1. 查询经验原记录
2. 记录反馈 (feedback 表)
3. 更新 adoption_rate
4. 更新 recall_count
5. 更新 last_hit_at (用于 TTL 检测)
  ↓
用于后续:
- 效果评估
- 升层候选扫描
- 经验失效检测
```

#### **D. 升层流程（Promotion，v0.6.0+）**:
```
定时扫描任务:
  ↓
1. 识别候选经验：
   - 同 Business 下 ≥ 2 个不同项目有相似经验
   - adoption_rate > 0.6 且 recall_count > 5
   - created_at > 30 天前
   
2. 计算升层得分:
   score = 跨项目出现次数 * 0.5 
         + adoption_rate * 0.3 
         + min(recall_count / 10, 1) * 0.2
   
3. score > 0.7 → 标记为升层候选
  ↓
人工审核并执行升层:
  ↓
- 原经验保留，新层级创建副本
- 通过 promoted_to / demoted_from 字段关联
- 支持双向流动（升层 + 降级）
```

---

### **7. 架构模式与特点**

#### **采用的设计模式**:

1. **DDD 分层架构** (Domain-Driven Design):
   - Domain Layer: 业务逻辑（search.py, quality.py, promotion.py）
   - Application Layer: 命令处理（handlers/）
   - Infrastructure Layer: 数据访问（backends/postgres.py）
   - Interface Layer: 外部入口（rest_api.py, server.py, cli.py）

2. **CQRS 变体**:
   - Commands: 所有写操作（SaveCommand, FeedbackCommand 等）
   - Queries: 所有读操作（SearchCommand, GetStatsCommand 等）
   - Command Bus: 统一分发处理

3. **Unit of Work 模式**:
   ```python
   async with uow:  # 事务管理
       experiences = uow.experiences
       vectors = uow.vectors
       stats = uow.stats
   ```

4. **Dependency Injection**:
   - Container（container.py）管理所有依赖
   - Handler 通过工厂函数接收 UoW factory

5. **Repository Pattern**:
   - StorageBackend 接口，PostgresBackend 实现
   - ExperienceStore, VectorStore, MetricsStore 抽象

#### **异步并发设计**:
- 所有 I/O 操作都是 async
- 向量批量计算可并行
- 支持多用户并发读写

#### **可扩展性设计**:
- Embedding Provider 抽象（本地 BGE vs 云端 OpenAI）
- Storage Backend 抽象（已支持 PostgreSQL，可扩展其他数据库）
- Handler 插件式注册

---

### **8. 接入方式与用户界面**

#### **三种接入方式**:

1. **MCP 集成**（最推荐）:
   ```json
   {
     "mcpServers": {
       "xp": {
         "command": "/path/to/.venv/bin/python",
         "args": ["/path/to/src/server.py"],
         "env": {
           "OPENAI_API_KEY": "sk-...",
           "PYTHONPATH": "/path/to/xp"
         }
       }
     }
   }
   ```
   - Claude Desktop、Cursor、codeflicker 原生支持
   - 三个 MCP Tools: `search`、`save`、`feedback`

2. **REST API**:
   ```
   POST /experiences/search
   POST /experiences/save
   POST /experiences/feedback
   GET /stats
   ```
   - 独立 REST 端点
   - 支持远端部署

3. **CLI 工具**:
   ```bash
   xp review        # 审核待定经验
   xp stats         # 查看效果统计
   xp add           # 手动添加经验
   xp promote       # 升层管理
   xp admin         # 管理员命令
   ```

---

### **9. 系统架构类型**

**项目类型**: **本地工具 + 云端服务混合**

1. **本地端**:
   - 轻量级 CLI 客户端
   - MCP Server (stdio 协议，无网络开销)
   - 本地 BGE 模型 (自动下载缓存)

2. **远端端**:
   - 中心化 PostgreSQL 数据库
   - 团队共享知识库
   - 可选：Edge Function 认证层 (Supabase)

3. **部署模式**:
   - **开发**: 本地 Python 运行，连接远端 PostgreSQL
   - **生产**: Docker 容器化单进程 FastAPI
   - **认证**: Token 基础，可集成 Supabase JWT 认证

---

### **10. 当前开发状态**

#### **完成的功能**:
- ✅ 三层知识层级体系 (Team/Business/Project)
- ✅ PostgreSQL 后端完整迁移 (v0.6.0)
- ✅ 向量相似度搜索
- ✅ 自动质量评分
- ✅ A/B 测试框架
- ✅ 采纳率追踪
- ✅ 升层候选识别
- ✅ 人工审核 CLI
- ✅ 统计面板 + 健康检查
- ✅ MCP 工具集成
- ✅ REST API

#### **测试覆盖**:
- 单元测试: 40+ 个
- 集成测试: 正在测试关键路径
- E2E 测试: REST API 和 CLI
- 特征化测试: 关键功能的行为规范

#### **文档完整性**:
- 4 个 ADR (架构决策记录)
- 详细的功能设计文档
- 开发指南 + 代码规范
- 用户 CLI 指南

---

### **11. 关键配置项**

```bash
# .env
DATABASE_URL=postgresql://user:pass@host:5432/dbname
XP_AUTH_URL=https://xxx.supabase.co/functions/v1/verify-token

# 可选
OPENAI_API_KEY=sk-...               # 用于 LLM 生成标题
XP_LLM_API_BASE=...                 # 自定义 LLM 服务
XP_LLM_MODEL=gpt-4o-mini            # 默认模型
XP_EMBEDDING_CACHE=~/.cache/xp      # 本地嵌入缓存
XP_HOME=~/.xp                       # 配置目录
```

---

### **12. 整体架构图概览**

```
┌─────────────────────────────────────────────────────────────┐
│                      Agent (Claude/Cursor)                   │
├─────────────────────────────────────────────────────────────┤
│  MCP Client                                                  │
│  (search, save, feedback tools)                              │
└─────────┬───────────────────────────────────────────────────┘
          │ stdio protocol
          ▼
┌─────────────────────────────────────────────────────────────┐
│                   MCP Server (server.py)                     │
│  - Tool 定义: search, save, feedback                        │
│  - 转发请求到 EdgeFunctionClient 或本地 service layer      │
└─────────┬───────────────────────────────────────────────────┘
          │
    ┌─────▼─────┐
    │ 本地模式   │ 远端模式
    └───┬───────┴──────────────────┐
        │ API calls                │
        ▼                          ▼
    ┌──────────┐          ┌──────────────────┐
    │ Edge     │          │ EdgeFunction     │
    │Function  │          │ (Supabase)       │
    │Client    │          │ - 验证 token     │
    └────┬─────┘          └────┬─────────────┘
         │                     │
         └─────────┬───────────┘
                   ▼
        ┌─────────────────────┐
        │   REST API Layer    │
        │  (FastAPI routes)   │
        └─────────┬───────────┘
                  ▼
        ┌─────────────────────┐
        │  Application Layer  │
        │  - CommandBus       │
        │  - Handlers (12+)   │
        └─────────┬───────────┘
                  ▼
        ┌─────────────────────┐
        │   Domain Layer      │
        │  - SearchService    │
        │  - QualityCompute   │
        │  - PromotionLogic   │
        │  - HealthCheck      │
        └─────────┬───────────┘
                  ▼
        ┌─────────────────────┐
        │Infrastructure Layer │
        │  - PostgresBackend  │
        │  - Unit of Work     │
        └─────────┬───────────┘
                  ▼
        ┌─────────────────────┐
        │ PostgreSQL Database │
        │  + pgvector         │
        │                     │
        │ Tables:             │
        │ - experiences       │
        │ - sessions          │
        │ - feedback          │
        │ - teams/businesses  │
        │ - promotion_cand..  │
        └─────────────────────┘
```

---

### **13. 核心价值与创新点**

1. **三层知识层级** - 支持从项目到业务到团队的知识逐级抽象和共享
2. **Vector-first 设计** - 使用向量相似度而非关键字匹配，提高召回精度
3. **质量保证** - 自动评分 + 人工审核的两层把关
4. **效果可量化** - 采纳率、召回次数、A/B 测试数据完整追踪
5. **MCP 原生集成** - 与现代 AI IDE 无缝协作
6. **异步全覆盖** - 支持高并发多用户场景

---

### **总结**

这是一个**高度工程化的 Python 应用**，采用了成熟的软件架构模式（DDD、CQRS、UoW），展示了：

- **清晰的分层设计**: 从 CLI/MCP → Application → Domain → Infrastructure
- **生产级的异步处理**: 完全基于 async/await
- **可扩展的存储抽象**: 支持多个后端（当前 PostgreSQL）
- **完整的测试体系**: 单元、集成、E2E、特征化测试
- **详尽的文档系统**: ADR、设计文档、用户指南完备

从**0 用户到团队共享知识库**的演进路径已经基本完成，接下来的重点应该是在生产环境中验证三层层级体系的有效性。
