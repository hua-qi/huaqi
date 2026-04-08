---
session_id: agent-e8a347af
date: 2026-04-07
time_start: 2026-04-07T07:52:05.033Z
time_end: 2026-04-07T07:53:33.598Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-07 15:52:05] [user]: 探索 /Users/lianzimeng/workspace/xp 这个项目的完整结构，包括：
1. 目录结构（顶层和主要子目录）
2. 主要源码文件及其用途
3. 测试框架和测试文件结构
4. package.json / pyproject.toml / requirements.txt 等依赖文件
5. docs/ 目录下的内容
6. 现有的任何计划或规范文档

请返回详细的结构信息，包括文件路径和主要内容摘要。
[2026-04-07 15:52:07] [assistant]: 我来为您探索这个项目的完整结构。让我从多个方面进行深入分析。
[2026-04-07 15:52:10] [assistant]: 现在让我读取关键的配置和文档文件：
[2026-04-07 15:52:13] [assistant]: 现在让我检查源码目录的详细结构和主要模块：
[2026-04-07 15:52:15] [assistant]: 现在让我查看文档中的关键设计文档和规范：
[2026-04-07 15:52:19] [assistant]: 现在让我查看应用层和基础设施的主要模块：
[2026-04-07 15:52:24] [assistant]: 现在让我查看测试框架和最新的设计文档：
[2026-04-07 15:52:28] [assistant]: 让我再检查一些关键的处理器和域模型：
[2026-04-07 15:52:31] [assistant]: 现在让我查看 CLI 指南文档：
[2026-04-07 15:52:34] [assistant]: 现在让我查看应用启动文件和一个重要的处理器：
[2026-04-07 15:52:37] [assistant]: 最后，让我检查现有的测试数量和结构：
[2026-04-07 15:53:33] [assistant]: 完美！现在让我汇总所有信息，创建一个详细的项目结构文档。

## 项目完整结构分析报告

我已经完成了对 `/Users/lianzimeng/workspace/xp` 项目的全面探索。下面是详细的结构信息：

---

### 1. 项目概览

**项目名称**: XP (Experience Processor) - Agent 最佳实践沉淀工具  
**版本**: 0.3.0  
**主要语言**: Python 3.11+  
**核心框架**: FastAPI, asyncpg, MCP (Model Context Protocol)  
**存储**: PostgreSQL + pgvector  
**特点**: TDD + DDD 架构，支持 MCP stdio 接入 Claude Desktop

**核心功能**:
- 自动沉淀任务经验
- 语义向量搜索
- 人工审核机制
- 三层知识层级体系（Team → Business → Project）

---

### 2. 目录结构

```
/Users/lianzimeng/workspace/xp/
├── src/                           # 主源码目录
│   ├── domain/                    # 领域层 (纯业务逻辑，零 I/O)
│   │   ├── __init__.py
│   │   ├── constants.py           # 业务阈值常量
│   │   ├── events.py              # 领域事件定义
│   │   ├── experience.py          # 经验状态机和激活逻辑
│   │   ├── extraction.py          # 经验提取规则
│   │   ├── feedback.py            # 置信度更新规则 (v1)
│   │   ├── feedback_v2.py         # 置信度更新规则 (v2)
│   │   ├── health.py              # 知识库健康度检测
│   │   ├── manifest_parser.py     # 项目依赖解析
│   │   ├── metadata.py            # tech_stack/scene/level 推断
│   │   ├── promotion.py           # 升层候选判断
│   │   ├── quality.py             # 质量评分算法
│   │   ├── save_domain.py         # 保存经验的业务规则
│   │   ├── search.py              # 搜索排序和 A/B 策略
│   │   ├── search_domain.py       # 搜索的业务规则
│   │   └── search_v2.py           # 搜索 v2 (层级感知)
│   │
│   ├── application/               # 应用层 (编排 domain + infrastructure)
│   │   ├── __init__.py
│   │   ├── command_bus.py         # Command 分发器
│   │   ├── commands.py            # 所有 Command 定义 (用例入口)
│   │   ├── unit_of_work.py        # 抽象 UnitOfWork 接口
│   │   └── handlers/              # 用例 Handler (每个用例一个)
│   │       ├── __init__.py
│   │       ├── analyze_handler.py
│   │       ├── experience_handler.py
│   │       ├── experience_query_handler.py
│   │       ├── extract_handler.py
│   │       ├── feedback_handler.py
│   │       ├── feedback_v2_handler.py
│   │       ├── infer_adoption_handler.py
│   │       ├── promotion_handler.py
│   │       ├── save_handler.py
│   │       ├── search_handler.py
│   │       ├── search_v2_handler.py
│   │       ├── session_handler.py
│   │       └── stats_handler.py
│   │
│   ├── infrastructure/            # 基础设施层 (I/O 实现)
│   │   ├── __init__.py
│   │   ├── unit_of_work.py        # PostgreSQL UoW 实现
│   │   ├── backends/              # 可插拔存储后端
│   │   │   ├── __init__.py
│   │   │   ├── base.py            # StorageBackend 抽象接口
│   │   │   └── postgres.py        # PostgreSQL 后端实现
│   │   └── (遗留) stores/        # 已删除的本地文件存储
│   │
│   ├── interfaces/                # 对外入口层 (参数转换，无业务逻辑)
│   │   ├── __init__.py
│   │   └── rest_api.py            # FastAPI REST API
│   │
│   ├── __init__.py
│   ├── auth.py                    # token 验证
│   ├── cli.py                     # CLI 命令行接口
│   ├── config.py                  # 配置管理
│   ├── container.py               # DI 容器，注册 Handler
│   ├── edge_client.py             # Supabase Edge Function 客户端
│   ├── embeddings.py              # 向量嵌入 (sentence-transformers)
│   ├── file_watcher.py            # 文件监听
│   ├── models.py                  # 核心数据模型 (dataclass)
│   ├── project_config.py          # 项目配置管理
│   └── server.py                  # MCP stdio Server
│
├── tests/                         # 测试目录 (58 个测试文件)
│   ├── unit/                      # 单元测试 (domain 层，零 I/O，零 mock)
│   ├── integration/               # 集成测试 (Handler 流程，InMemoryUoW)
│   ├── contract/                  # 契约测试 (Command/Event schema)
│   ├── characterization/          # 特征测试 (固化现有行为)
│   ├── e2e/                       # E2E 测试 (完整工作流)
│   ├── helpers/                   # 测试辅助
│   │   ├── fake_uow.py            # InMemoryUnitOfWork 实现
│   │   └── test_fake_uow_smoke.py
│   └── __pycache__/
│
├── docs/                          # 文档目录
│   ├── design/                    # ADR (Architecture Decision Record)
│   │   └── adr/
│   │       ├── ADR-001-implicit-evolution-architecture.md      # 异步极速响应架构
│   │       ├── ADR-002-team-knowledge-hierarchy-and-centralized-deployment.md  # 层级体系
│   │       ├── ADR-003-async-postgresql-migration.md           # PostgreSQL 异步迁移
│   │       └── ADR-004-remove-local-mode-postgresql-only.md    # 删除本地模式
│   │
│   ├── designs/                   # 详细设计文档 (12 份)
│   │   ├── 2026-01-04-cloud-refactor-tdd.md
│   │   ├── 2026-02-04-architecture-refactor.md
│   │   ├── 2026-03-31-ab-test-stats-refactor.md
│   │   ├── 2026-03-31-experience-extraction-refactor.md
│   │   ├── 2026-03-31-xp-full-refactor-roadmap.md
│   │   ├── 2026-04-05-storage-layer-postgres-migration.md
│   │   ├── 2026-04-06-hierarchy-aware-knowledge-system.md
│   │   ├── 2026-05-04-pypi-distribution-stdio-mcp.md
│   │   ├── 2026-05-04-xp-team-knowledge-refactor.md
│   │   └── 2026-07-04-cs-arch-mysql-refactor.md (最新)
│   │
│   ├── features/                  # 功能模块设计
│   │   ├── embedding-refactor-design.md
│   │   ├── embedding-refactor-implementation.md
│   │   ├── hybrid-search-design.md
│   │   ├── implicit-evolution-refactor.md
│   │   ├── mcp-tools-v2.md
│   │   ├── postgres-async-migration.md
│   │   └── team-knowledge-hierarchy.md
│   │
│   ├── guides/                    # 用户和开发者指南
│   │   ├── dev/
│   │   │   ├── DOC_GUIDELINES.md          # 文档规范
│   │   │   ├── code-standards.md          # 代码规范 ★ (重要)
│   │   │   └── test-acceptance.md         # 测试验收规范 ★ (重要)
│   │   └── user/
│   │       └── cli-guide.md               # CLI 命令参考
│   │
│   └── plans/                     # 实施计划 (11 份)
│
├── scripts/                       # 工具脚本
│   ├── __init__.py
│   ├── migrate_knowledge_to_server.py     # 迁移脚本
│   └── migrate_to_postgres.py
│
├── supabase/                      # Supabase 配置
│   ├── .temp/                     # 临时文件
│   └── functions/
│       └── verify-token/          # Edge Function
│
├── .codeflicker/                  # IDE 配置
├── .github/
│   └── workflows/
│       └── ci.yml                 # GitHub Actions CI
├── .env.example                   # 环境变量示例
├── .gitignore
├── .ruff_cache/                   # Ruff 缓存
├── pyproject.toml                 # Python 项目配置 ★
├── uv.lock                        # Python 依赖锁文件
├── xp-wrapper.sh                  # Shell 包装脚本
├── main.py                        # 启动入口
├── README.md                      # 项目说明
├── QUICK_START.md                 # 快速开始指南
├── CONTRIBUTING.md                # 贡献指南
├── CHANGELOG.md                   # 版本变更记录
└── LICENSE                        # MIT 许可
```

---

### 3. 核心源码文件及用途

#### 3.1 数据模型 (`src/models.py`)

**核心 dataclass**:
- `ExperienceType` - 经验类型枚举 (bugfix, feature, pattern)
- `ExperienceLevel` - 层级 (L1, L2, L3)
- `ExperienceStatus` - 状态 (pending, active, archived)
- `ExperienceMetadata` - 结构化元数据 (tech_stack, problem_type, scene)
- `Experience` - 核心经验模型
- `Session` - 用户会话
- `Feedback` - 反馈记录
- `Team`, `Business`, `Project` - 三层组织结构
- `PromotionCandidate` - 升层候选

#### 3.2 应用层 (`src/application/`)

**命令** (`commands.py`):
- `SearchCommand` - 搜索经验
- `SaveCommand` - 保存经验
- `FeedbackV2Command` - 提交反馈
- `RecordSessionCommand` - 记录会话
- `GetStatsCommand` - 获取统计数据
- 等 20+ 个 Command 类

**Command Bus** (`command_bus.py`):
- 分发 Command 到对应 Handler
- 统一错误处理

**单位工作** (`unit_of_work.py`):
- 抽象 UnitOfWork 接口
- 事务管理

#### 3.3 Handler 处理器 (`src/application/handlers/`)

| Handler | 用途 |
|---------|------|
| `ExperienceHandler` | 创建、激活、归档经验 |
| `SearchHandler` | 向量搜索经验 |
| `SearchV2Handler` | 层级感知搜索 |
| `SaveHandler` | 保存新经验 |
| `FeedbackHandler` | 更新置信度 v1 |
| `FeedbackV2Handler` | 更新置信度 v2 |
| `SessionHandler` | 记录任务会话 |
| `StatsHandler` | 计算统计指标 |
| `AnalyzeHandler` | 质量分析报告 |
| `ExperienceQueryHandler` | 查询经验 |
| `ExtractHandler` | 提取经验字段 |
| `InferAdoptionHandler` | 推断采纳情况 (遗留) |
| `PromotionScanHandler` | 扫描升层候选 |

#### 3.4 域逻辑 (`src/domain/`)

**核心业务决策**:
- `experience.py` - 经验状态机转换
- `quality.py` - 质量评分算法
- `feedback.py` - 置信度增量计算
- `search.py` - 搜索排序逻辑
- `metadata.py` - 技术栈/场景/层级推断
- `promotion.py` - 升层判断标准
- `health.py` - 知识库健康度 6 项指标

**接口与入口**:
- `cli.py` - CLI 命令行 (xp, xp-server)
- `server.py` - MCP stdio Server
- `rest_api.py` - FastAPI REST API
- `edge_client.py` - Supabase Edge Function 客户端

---

### 4. 测试框架和文件结构

**测试总数**: 58 个测试文件

**四层测试体系** (按 TDD 3 层模型):

```
tests/
├── unit/                 (28 文件) 单元测试：domain 层纯逻辑，零 I/O
│   ├── test_quality.py                  # 质量评分
│   ├── test_feedback*.py                # 反馈逻辑
│   ├── test_experience*.py              # 经验状态机
│   ├── test_search*.py                  # 搜索排序
│   ├── test_promotion*.py               # 升层逻辑
│   ├── test_health_status.py            # 健康度检测
│   ├── test_metadata.py                 # 元数据推断
│   ├── test_manifest_parser.py          # 依赖解析
│   ├── test_hierarchy_models.py         # 层级模型
│   ├── test_backends.py                 # 后端接口
│   ├── test_command_bus.py              # 命令总线
│   ├── test_config.py                   # 配置管理
│   ├── test_auth.py                     # 认证逻辑
│   ├── test_cli_*.py                    # CLI 相关
│   ├── test_postgres_uow.py             # PostgreSQL UoW
│   ├── test_mcp_tools.py                # MCP 工具
│   ├── test_pyproject.py                # pyproject.toml
│   └── test_server_main.py              # 服务启动
│
├── integration/          (3 文件) 集成测试：完整 Handler 流程，InMemoryUoW
│   ├── test_experience_handler.py
│   ├── test_feedback_handler.py
│   └── test_rollback.py
│
├── contract/            (2 文件) 契约测试：Command/Event schema 稳定性
│   ├── test_commands.py
│   └── test_events.py
│
├── characterization/    (5 文件) 特征测试：固化现有行为防回归
│   ├── test_abtest.py
│   ├── test_extraction.py
│   ├── test_feedback.py
│   ├── test_metadata_infer.py
│   └── test_search.py
│
├── e2e/                 (5 文件) E2E 测试：完整工作流
│   ├── test_rest_api.py
│   ├── test_extract_api.py
│   ├── test_search_api.py
│   ├── test_health.py
│   └── test_health.py
│
└── helpers/             (1 文件) 测试辅助
    ├── fake_uow.py                      # InMemoryUnitOfWork 实现
    └── test_fake_uow_smoke.py           # 烟雾测试
```

**测试验收标准** (见 `docs/guides/dev/test-acceptance.md`):
- 单元测试零 mock，零 I/O
- 集成测试用 `InMemoryUnitOfWork`
- domain 层代码覆盖率 ≥ 90%
- 单个单元测试 < 1ms，集成测试 < 100ms

---

### 5. 依赖配置文件

#### 5.1 `pyproject.toml`

```toml
[project]
name = "xp-agent"
version = "0.3.0"
description = "团队 Agent 最佳实践沉淀工具，支持 stdio MCP 接入 Claude Desktop"
requires-python = ">=3.11"

[project.dependencies]
mcp[cli]>=1.0.0
python-dotenv>=1.0.0
sentence-transformers>=2.2.0          # 向量嵌入
numpy>=1.24.0
asyncpg>=0.29.0                       # PostgreSQL 异步驱动
fastapi>=0.110.0
uvicorn>=0.29.0
pgvector>=0.2.0                       # PostgreSQL 向量扩展
httpx>=0.27.0
pytest-asyncio>=1.3.0

[project.scripts]
xp = "src.cli:run"                    # CLI 入口
xp-server = "src.server:run"          # MCP Server 入口
```

#### 5.2 `.env.example`

```bash
DATABASE_URL=postgresql://user:password@host:5432/dbname
XP_AUTH_URL=https://YOUR_PROJECT.supabase.co/functions/v1/verify-token
```

#### 5.3 `uv.lock`

锁定所有依赖的版本，确保可重现构建。

---

### 6. 文档目录详解

#### 6.1 核心规范文档

| 文档 | 关键内容 |
|------|---------|
| `code-standards.md` ★ | 目录规范、分层原则、import 禁则、命名规范 |
| `test-acceptance.md` ★ | TDD 三层体系、验收标准、示例测试 |
| `DOC_GUIDELINES.md` | 文档格式、Markdown 规范 |

#### 6.2 架构决议 (ADR)

| ADR | 决议内容 |
|-----|---------|
| ADR-001 | 极简入参、异步抽取与显式反馈的架构 (finalize_task 重构) |
| ADR-002 | 三层知识层级体系 + 中心化 Edge Function 部署 |
| ADR-003 | PostgreSQL 异步迁移 (asyncpg) |
| ADR-004 | 删除本地模式，只支持 PostgreSQL |

#### 6.3 最新设计文档

**2026-07-04-cs-arch-mysql-refactor.md** (最新):
- 从 stdio + PostgreSQL 迁移到 CS 架构 + MySQL
- MCP Streamable HTTP 远端模式
- 向量检索方案 (应用层 numpy)
- A/B Test 策略 (LLM rerank vs 语义匹配)
- Embedding 模型升级 (BGE-m3)

#### 6.4 功能设计

- `team-knowledge-hierarchy.md` - 三层升层完整逻辑
- `mcp-tools-v2.md` - MCP 工具 v2 定义
- `hybrid-search-design.md` - 混合搜索 (语义 + 全文)
- `implicit-evolution-refactor.md` - 隐式学习重构

#### 6.5 实施计划

每个设计文档对应一份 `plans/` 中的实施计划，包含：
- 任务分解
- 预计工期
- 依赖关系

---

### 7. CLI 命令体系

#### 7.1 主要命令

```bash
# 经验管理
xp add                              # 交互式添加经验
xp review                           # 审核待 review 经验
xp edit <ID前缀>                    # 编辑经验
xp archive <ID前缀>                 # 归档经验
xp delete <ID前缀>                  # 删除经验
xp import <文件>                    # 批量导入

# 统计分析
xp stats [--since 7d] [--project <id>] [--business <id>] [--team <id>] [--compare]
xp analyze                          # 质量分析报告

# 升层管理
xp promote list                     # 查看升层候选
xp promote <id> --to business <biz-id>
xp promote <id> --to team <team-id>
xp promote ignore <id>

# 项目管理
xp project list
xp project switch <名>

# 其他
xp watch                            # 文件监听
xp login                            # 登录验证
```

#### 7.2 MCP 工具 (3 个)

| 工具 | 调用时机 | 返回值 |
|------|---------|--------|
| `search` | 任务开始前 | 相关经验列表 + `search_event_id` |
| `save` | 任务完成后 | 保存确认 |
| `feedback` | 经验有用时 | 反馈确认 |

---

### 8. 架构关键原则

#### 8.1 分层规则 (DDD)

```
domain/     不能 import application/ 或 infrastructure/
application/ 通过依赖注入使用 infrastructure/ 的接口
interfaces/ 只调用 CommandBus.dispatch()，不直接调用 Handler
```

#### 8.2 测试原则 (TDD)

- **单元测试**: domain 层，零 I/O，零 mock
- **集成测试**: Handler 流程，使用 InMemoryUoW
- **契约测试**: Command/Event schema
- **特征测试**: 固化外部可见行为

#### 8.3 编程约定

- 使用 `async/await` 进行异步编程
- 通过 UnitOfWork 管理事务
- Command → Handler → 事件发布的工作流
- 无需显式 mock，用 FakeProvider/InMemoryStore

---

### 9. 重要的最新变更 (v0.3.0+)

#### 9.1 ADR-002 三层知识层级体系

- **新增**: `Team`, `Business`, `Project` 三层多对多关联
- **新增**: `PromotionCandidate` 升层候选模型
- **改进**: `search` → 层级展开 → 加权合并 → 返回结果
- **改进**: `save` → 质量评分 → 自动激活或进入 pending
- **新增**: `promote` 命令和升层推荐算法

#### 9.2 ADR-001 异步极速响应

- **改进**: `finalize_task` 从同步等待改为 Fire-and-Forget
- **改进**: 后台异步 LLM 拆解，减少端侧负担
- **改进**: 废除向量相似度推断，回归显式反馈
- **新增**: `adopted_experience_ids` / `rejected_experience_ids` 明确声明

#### 9.3 ADR-004 PostgreSQL 唯一路径

- **删除**: 本地 SQLite/JSON 存储层
- **删除**: `src/storage.py` 和 `src/infrastructure/stores/`
- **要求**: 强制配置 `DATABASE_URL`，缺失时抛错

---

### 10. 启动和运行

#### 10.1 开发环境设置

```bash
git clone <repo> xp
cd xp
python -m venv .venv
source .venv/bin/activate
pip install -e .
cp .env.example .env
# 配置 DATABASE_URL 和 XP_AUTH_URL
```

#### 10.2 运行 MCP Server

```bash
xp-server
# 或通过 MCP 配置自动启动
```

#### 10.3 运行测试

```bash
python -m pytest tests/ -v
python -m pytest tests/unit -v  # 单元测试
pytest tests/unit -k "not postgres"  # 排除 postgres 依赖
```

---

### 11. 数据流工作流

```
Agent 任务开始
    ↓
  search() → SearchV2Handler → 层级展开 → 向量检索 → 返回经验 ID 列表
    ↓ (保存 search_event_id)
  [Agent 进行开发工作]
    ↓
Agent 任务完成
    ↓
  save() → SaveHandler → 质量评分 → 是否自动激活
    ↓ (如果质量高)
  自动激活并生成向量嵌入
    ↓
feedback() (可选) → FeedbackV2Handler → 更新置信度
    ↓
定期扫描升层候选
    ↓
promote 命令 → 提升到 Business/Team 层
```

---

### 总结

这是一个**高度结构化的 Python 项目**，严格遵循 DDD + TDD 范式：

✅ **清晰的分层**: domain (业务逻辑) → application (编排) → infrastructure (I/O)  
✅ **完善的测试**: 58 个测试文件覆盖四层测试体系  
✅ **详尽的文档**: 4 份 ADR + 12 份设计文档 + 多份规范指南  
✅ **现代化架构**: asyncio + PostgreSQL + MCP stdio + Edge Function  
✅ **知识层级**: 三层多对多关联，支持升层推荐  
✅ **生产就绪**: 强制 PostgreSQL、严格的测试验收标准、健康度监控  

该项目适合学习 **Python 大型应用架构**、**TDD 实践**、**DDD 领域建模**。
