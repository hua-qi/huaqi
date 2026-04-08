---
session_id: agent-3f5b4c93
date: 2026-04-06
time_start: 2026-04-06T14:26:37.596Z
time_end: 2026-04-06T14:27:32.796Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-06 22:26:37] [user]: 探索 /Users/lianzimeng/workspace/xp 项目的结构，找出：1) README 或文档，2) 入口文件，3) 主要功能，4) 如何安装和使用。返回详细的使用说明。
[2026-04-06 22:26:40] [assistant]: 我来帮您探索这个项目的结构。让我从多个角度来了解这个项目。
[2026-04-06 22:26:47] [assistant]: 现在让我读取关键文件来获取更详细的信息：
[2026-04-06 22:26:50] [assistant]: 让我再读取更多关键文件来完善说明：
[2026-04-06 22:26:54] [assistant]: 最后让我检查一下源代码结构和配置文件：
[2026-04-06 22:26:57] [assistant]: 现在让我生成详细的使用说明文档：
[2026-04-06 22:27:32] [assistant]: 完美！现在我有了充分的信息。让我生成一个详细的项目探索报告：

## 项目探索报告：XP (Experience Processor)

基于对 `/Users/lianzimeng/workspace/xp` 的全面扫描，以下是详细的项目说明：

---

### 1) README 与文档位置

#### 主文档
- **README.md** - `/Users/lianzimeng/workspace/xp/README.md`：项目介绍与快速开始指南
- **QUICK_START.md** - `/Users/lianzimeng/workspace/xp/QUICK_START.md`：5分钟快速上手指南
- **CHANGELOG.md** - `/Users/lianzimeng/workspace/xp/CHANGELOG.md`：版本变更记录
- **CONTRIBUTING.md** - `/Users/lianzimeng/workspace/xp/CONTRIBUTING.md`：开发贡献指南

#### 详细文档库
```
/Users/lianzimeng/workspace/xp/docs/
├── design/adr/                           # 架构决策记录
│   ├── ADR-001-implicit-evolution-architecture.md
│   ├── ADR-002-team-knowledge-hierarchy-and-centralized-deployment.md
│   ├── ADR-003-async-postgresql-migration.md
│   └── ADR-004-remove-local-mode-postgresql-only.md
├── designs/                              # 设计文档
│   ├── 2026-01-04-cloud-refactor-tdd.md
│   ├── 2026-02-04-architecture-refactor.md
│   └── ... (共7个设计方案)
├── features/                             # 功能设计与实现
│   ├── embedding-refactor-design.md
│   ├── embedding-refactor-implementation.md
│   ├── hybrid-search-design.md
│   ├── implicit-evolution-refactor.md
│   ├── mcp-tools-v2.md
│   ├── postgres-async-migration.md
│   ├── team-knowledge-hierarchy.md
│   └── team-knowledge-hierarchy.md
├── guides/
│   ├── user/
│   │   └── cli-guide.md                 # CLI命令与MCP工具完整参考
│   └── dev/
│       ├── DOC_GUIDELINES.md            # 文档规范
│       ├── code-standards.md            # 代码规范
│       └── test-acceptance.md           # 测试验收规范
└── plans/                                # 实现计划
```

---

### 2) 入口文件

#### 主要入口

| 入口文件 | 描述 | 用途 |
|---------|------|------|
| **main.py** | `/Users/lianzimeng/workspace/xp/main.py` | 直接运行入口（调用 server.py） |
| **src/server.py** | MCP Server 入口 | 启动 stdio MCP 服务器，提供 search/save/feedback 工具 |
| **src/cli.py** | CLI 命令行入口 | 管理员命令行工具，支持 review/stats/add/edit 等 |
| **xp-wrapper.sh** | Shell 包装脚本 | 便捷启动脚本 |

#### 项目依赖配置
- **pyproject.toml** - `/Users/lianzimeng/workspace/xp/pyproject.toml`：项目配置、依赖声明、构建信息

#### 环境配置
- **.env.example** - 环境变量模板（需要 DATABASE_URL、XP_AUTH_URL）

---

### 3) 主要功能架构

#### 核心功能

**XP** 是一个 **Agent 最佳实践沉淀工具**，让 AI 助手在每次任务中自动积累经验。

核心功能模块：

```
┌─────────────────────────────────────────────────────────┐
│           XP Agent 最佳实践沉淀系统                      │
├─────────────────────────────────────────────────────────┤
│ 1. 自动沉淀  │ 任务完成后自动提取经验、生成最佳实践     │
│ 2. 智能检索  │ 基于向量搜索，任务开始前自动匹配经验     │
│ 3. 人工审核  │ 通过 CLI 快速 review 候选经验            │
│ 4. 效果追踪  │ 统计工具使用效果，量化研发效能提升       │
│ 5. 层级管理  │ Team → Business → Project 多层知识体系    │
└─────────────────────────────────────────────────────────┘
```

#### 架构分层

```
src/
├── server.py                  # MCP Server 入口（stdio 接口）
├── cli.py                     # CLI 命令行工具
├── auth.py                    # 认证模块
├── config.py                  # 配置管理
├── edge_client.py             # Supabase Edge Function 客户端
├── embeddings.py              # 向量嵌入模块
├── file_watcher.py            # 文件监测
├── project_config.py          # 项目配置
│
├── application/               # 应用层（业务命令与处理）
│   ├── command_bus.py         # 命令总线
│   ├── commands.py            # 命令定义
│   └── handlers/              # 命令处理器
│       ├── search_handler.py      # 搜索经验
│       ├── save_handler.py        # 保存经验
│       ├── feedback_handler.py    # 反馈经验
│       ├── stats_handler.py       # 统计分析
│       ├── analyze_handler.py     # 质量分析
│       ├── promote_handler.py     # 升层管理
│       └── ... (共13个处理器)
│
├── domain/                    # 领域模型（核心业务逻辑）
│   ├── experience.py          # 经验数据模型
│   ├── search.py/search_v2.py # 搜索算法
│   ├── extraction.py          # 经验提取
│   ├── feedback.py            # 反馈逻辑
│   ├── quality.py             # 质量评分
│   ├── health.py              # 健康状态
│   ├── promotion.py           # 升层策略
│   ├── metadata.py            # 元数据处理
│   └── ... (共17个模型)
│
├── infrastructure/            # 基础设施层
│   ├── unit_of_work.py        # 单元工作模式
│   └── backends/
│       ├── base.py            # 后端抽象
│       └── postgres.py        # PostgreSQL 实现
│
└── interfaces/                # 接口层
    └── rest_api.py            # REST API 路由
```

#### 关键组件

1. **MCP Tools** (src/server.py)
   - `search`: 任务开始前调用，从知识库召回相关经验
   - `save`: 任务完成后调用，沉淀经验到知识库
   - `feedback`: 反馈经验有用/无用，优化知识库质量

2. **CLI 命令** (src/cli.py)
   - 经验管理：add, review, edit, archive, delete, import
   - 统计分析：stats, analyze
   - 升层管理：promote list, promote <id>, promote ignore
   - 项目管理：project list, project switch
   - 管理员命令：create-token, create-team, create-business, link

3. **存储后端** (src/infrastructure/)
   - PostgreSQL 异步支持（asyncpg）
   - 向量存储（pgvector）
   - 单元工作模式（UnitOfWork）

4. **向量检索** (src/embeddings.py)
   - 基于 sentence-transformers
   - 混合搜索（关键词 + 向量）

---

### 4) 安装与使用指南

#### 安装步骤

**第一步：克隆项目**
```bash
git clone https://github.com/your-org/xp.git xp
cd xp
```

**第二步：复制环境配置**
```bash
cp .env.example .env
```

**第三步：创建 Python 虚拟环境**
```bash
python3 -m venv .venv
source .venv/bin/activate  # macOS/Linux
# 或
.venv\Scripts\activate      # Windows
```

**第四步：安装项目依赖**
```bash
pip install -e .
```

**第五步：配置环境变量** (编辑 `.env` 文件)
```bash
# 方案1：本地开发（需要 PostgreSQL）
DATABASE_URL=postgresql://user:password@localhost:5432/xp_db
XP_AUTH_URL=https://your-project.supabase.co/functions/v1/verify-token

# 方案2：Supabase 云端
# 使用 Supabase 提供的连接字符串
```

**第六步：初始化数据库**
```bash
# 自动创建表结构（在首次连接时）
python -m src.cli --help  # 触发初始化
```

#### 快速开始

**方案A：作为 MCP 服务器接入 Cursor/Claude Desktop**

1. 编辑 MCP 配置文件：
   - Cursor: `~/.cursor/mcp.json` 或 `.cursor/mcp.json`
   - Claude Desktop: `~/Library/Application Support/Claude/claude_desktop_config.json`

2. 添加配置：
```json
{
  "mcpServers": {
    "xp": {
      "command": "/path/to/xp/.venv/bin/python",
      "args": ["/path/to/xp/src/server.py"],
      "env": {
        "OPENAI_API_KEY": "your_key_here",
        "PYTHONPATH": "/path/to/xp"
      }
    }
  }
}
```

3. 重启 Cursor/Claude，即可使用 `search`, `save`, `feedback` 工具。

**方案B：使用 CLI 命令**

```bash
# 查看帮助
xp --help

# 登录（保存 token）
xp login

# 手动添加经验
xp add

# 审核待定经验
xp review

# 查看统计数据
xp stats

# 编辑现有经验
xp edit <ID前缀>

# 查看升层候选
xp promote list

# 升层经验
xp promote <id> --to business <biz-id>

# 管理员命令（需要 DATABASE_URL）
xp admin create-team
xp admin create-business
xp admin create-token
```

**方案C：直接启动 Server**

```bash
# 启动 MCP Server（stdio 模式）
python main.py

# 或使用脚本
./xp-wrapper.sh
```

#### Agent 使用工作流

**任务开始前：**
```
Agent 调用 → search (task_description, project_id, project_manifest)
           ↓
           返回 search_event_id + 相关历史经验
           ↓
Agent 参考经验继续工作
```

**任务完成后：**
```
Agent 调用 → save (task_description, solution, key_decisions, tags, project_id, outcome, search_event_id)
           ↓
           保存经验到知识库
           ↓
(可选) feedback (search_event_id, helpful_ids, unhelpful_ids)
           ↓
           优化知识库质量
```

#### CLI 审核与统计

**审核经验：**
```bash
xp review

# 逐条确认/拒绝候选经验
# 按 'y' 确认，'n' 拒绝，'q' 退出
```

**查看统计：**
```bash
# 全部时间
xp stats

# 最近 7 天
xp stats --since 7d

# 按项目过滤
xp stats --project <project-id>

# 比较不同层级
xp stats --compare

# 质量分析报告
xp analyze
```

**质量指标（健康状态面板）**：
- 采纳率 > 60% ✓
- 未命中率 < 20% ✓
- Pending 堆积 < 20 条 ✓
- 反馈覆盖率 > 40% ✓

---

### 5) 核心版本信息

| 项 | 详情 |
|---|------|
| **项目版本** | v0.3.0（含 v0.6.0 计划） |
| **最新提交** | 041e612 - feat: implement hierarchy-aware knowledge system |
| **Python 要求** | >= 3.11 |
| **主要依赖** | mcp, sentence-transformers, asyncpg, fastapi, pgvector |
| **数据库** | PostgreSQL（asyncpg 异步驱动） |
| **向量搜索** | pgvector + sentence-transformers |
| **部署方式** | stdio MCP + Supabase Edge Function |

---

### 6) 重要架构特性

#### 三层知识体系
```
Project 层（项目级）← 团队本地经验
    ↓
Business 层（业务线级）← 跨项目通用经验
    ↓
Team 层（团队级）← 团队级最佳实践
```

#### 异步架构
- 全量异步操作（async/await）
- PostgreSQL 异步连接（asyncpg）
- pytest-asyncio 支持异步测试

#### 无状态服务
- MCP Server 本地无数据库依赖
- 所有逻辑运行在 Supabase Edge Function
- Token 认证 + Edge Function 转发

#### 多重搜索
- 向量相似度搜索
- 关键词全文搜索
- 混合排序

---

### 7) 测试覆盖

```
tests/
├── unit/              # 单元测试（40+ 测试文件）
├── integration/       # 集成测试（数据库、业务流）
├── e2e/              # 端到端测试（REST API、完整流程）
├── characterization/ # 行为特征化测试
├── contract/         # 约定测试（命令、事件）
└── helpers/          # 测试辅助（FakeUoW、InMemoryStore）
```

运行测试：
```bash
# 全部测试
pytest tests/ -v

# 排除需要外部依赖的测试
pytest tests/ -k "not server and not postgres" -v

# 特定测试
pytest tests/unit/test_experience.py -v
```

---

### 总结

**XP** 是一个企业级的 AI 知识沉淀系统，具有：
- ✓ **完善的 MCP 集成**，无缝接入 Cursor、Claude Desktop
- ✓ **生产级的 PostgreSQL 存储**，支持异步操作
- ✓ **三层知识体系**，支持团队、业务线、项目级别管理
- ✓ **智能向量检索**，基于语义的经验匹配
- ✓ **完整的 CLI 工具**，支持审核、统计、升层
- ✓ **严谨的测试体系**，TDD + DDD 开发模式
- ✓ **详尽的文档**，包含设计文档、API 文档、开发规范

所有代码位于 `/Users/lianzimeng/workspace/xp` 目录下，项目已完成最新的"层级感知知识系统"实现（Task 1-12）。
