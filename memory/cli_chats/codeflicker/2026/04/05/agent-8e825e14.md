---
session_id: agent-8e825e14
date: 2026-04-05
time_start: 2026-04-05T03:02:53.374Z
time_end: 2026-04-05T03:05:59.305Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-05 11:02:53] [user]: 请探索 /Users/lianzimeng/workspace/xp 目录的完整结构，包括：
1. 顶层目录结构
2. 主要源代码文件（src/ 或类似目录）
3. package.json 或 pyproject.toml 等配置文件的内容
4. README 文件内容
5. 任何 MCP 相关的配置或实现文件
6. docs/ 目录下的内容

请返回尽可能详细的项目概览，包括文件树、关键配置内容、现有 MCP tool 列表等。
[2026-04-05 11:02:56] [assistant]: 我会为您详细探索这个目录结构。让我开始系统地查看项目的各个方面。
[2026-04-05 11:02:59] [assistant]: 现在让我继续探索关键配置文件和更多内容：
[2026-04-05 11:03:05] [assistant]: .codeflicker 目录是空的。让我检查 MCP 相关的配置文件和 main.py：
[2026-04-05 11:03:20] [assistant]: 现在让我探索 docs 目录的内容和源代码结构：
[2026-04-05 11:03:26] [assistant]: 让我继续查看核心源代码文件的结构：
[2026-04-05 11:03:33] [assistant]: 现在让我查看一些关键的配置文件和源代码文件：
[2026-04-05 11:03:37] [assistant]: 现在让我查看一些关键配置文件和创建一个完整的项目结构文档：
[2026-04-05 11:03:41] [assistant]: 现在让我生成一个完整的目录结构树文件和项目概览：
[2026-04-05 11:04:08] [assistant]: 完美！现在让我查看一些额外的关键文件来完成图景：
[2026-04-05 11:04:12] [assistant]: 完美！现在让我创建一个完整的项目概览文档。让我先生成一个详细的文件树：
[2026-04-05 11:05:12] [assistant]: 完美！现在让我创建一个完整的项目概览文档：
[2026-04-05 11:05:15] [assistant]: 让我用另一种方式生成这个文档：
[2026-04-05 11:05:20] [assistant]: 现在让我生成一个完整的总结报告：
[2026-04-05 11:05:59] [assistant]: 现在让我创建一个完整的项目概览总结：

## 完整项目概览

我已经详细探索了 `/Users/lianzimeng/workspace/xp` 目录。以下是完整的项目概览：

---

## 📋 项目基本信息

| 项目 | XP (eXPerience Processor) |
|------|---------------------------|
| **描述** | Agent 最佳实践沉淀工具 - 让 agent 在任何任务中越用越聪明 |
| **版本** | 0.2.0 |
| **语言** | Python 3.11+ |
| **架构** | DDD (Domain-Driven Design) + Command Bus + MCP Server |
| **代码行数** | ~4430 行源代码（src/）|
| **GitHub** | 已配置 CI/CD (GitHub Actions) |
| **许可证** | MIT |

---

## 🗂️ 目录结构总览

```
xp/
├── docs/                    # 完整文档库（21个 MD 文件）
│   ├── design/adr/         # 架构决策记录
│   ├── designs/            # 5个功能设计文档
│   ├── features/           # 4个功能实现文档
│   ├── guides/dev/         # 3个开发规范文档
│   ├── guides/user/        # CLI 和 MCP 工具参考
│   └── plans/              # 8个实施计划文档
│
├── src/                     # 源代码（DDD 架构）
│   ├── domain/             # 业务逻辑层（零 I/O）
│   ├── application/        # 编排层（Command Bus）
│   ├── infrastructure/      # 基础设施层（I/O 实现）
│   ├── interfaces/          # 对外入口（REST API）
│   ├── cli.py              # 命令行界面 (769 行)
│   ├── server.py           # MCP 服务器 (488 行，核心入口)
│   └── 其他配置文件
│
├── tests/                   # 测试套件（四层分离）
│   ├── unit/               # 19个单元测试
│   ├── integration/        # 3个集成测试
│   ├── contract/           # 2个契约测试
│   ├── e2e/                # 5个端到端测试
│   ├── characterization/   # 5个特征测试
│   └── helpers/            # 测试辅助工具（InMemory UoW）
│
├── scripts/                # 2个数据迁移脚本
├── .github/workflows/      # GitHub Actions CI 配置
└── 配置文件 (pyproject.toml, CONTRIBUTING.md 等)
```

---

## 🔧 核心架构

### MCP 工具（6个）

Agent 通过这 6 个工具与 XP 交互：

| 工具 | 调用时机 | 功能 |
|------|---------|------|
| **search_best_practices** | 任务开始前 | 检索历史经验 |
| **extract_experience** | 任务完成后 | 沉淀新经验 |
| **record_session** | 任务完成后 | 记录会话数据 |
| **record_feedback** | 任务完成后 | 标记经验是否有帮助 |
| **infer_adoption** | 任务完成后 | 自动推断采纳情况 |
| **finalize_task** | 任务完成后 | 一键完成（整合上述三步） |

### CLI 命令（14个）

用户通过命令行管理经验知识库：

```
xp add              # 交互式添加经验
xp review           # 逐条审核待确认的经验
xp edit <id>        # 编辑经验
xp archive <id>     # 归档经验
xp delete <id>      # 永久删除经验
xp import <file>    # 批量导入经验
xp stats [--since]  # 查看效果统计
xp analyze          # 质量分析报告
xp show <id>        # 查看经验详情
xp project          # 项目管理
xp watch            # 文件变更检测
xp sync/migrate     # 数据迁移
```

### Command Bus 架构

```
MCP 请求 / CLI 命令
    ↓
Command 对象构造
    ↓
CommandBus.dispatch()
    ↓
找到注册的 Handler
    ↓
Handler 调用 domain 业务逻辑
    ↓
UnitOfWork 统一提交/回滚三个存储
    ↓
ExperienceStore (JSON) + VectorStore + MetricsStore (SQLite)
```

---

## 📦 三存储架构

| 存储 | 实现方式 | 用途 |
|------|---------|------|
| **ExperienceStore** | JSON 文件 + SQLite | 经验元数据、状态、配置 |
| **VectorStore** | 二进制文件 / PostgreSQL | Embedding 向量（语义搜索） |
| **MetricsStore** | SQLite 数据库 | 搜索事件、反馈、统计数据 |

---

## 📊 核心数据模型

### Experience (经验)
```python
id: UUID
type: bugfix | feature | pattern        # 经验类型
level: L1 | L2 | L3                     # 经验等级
title: str                              # 自动生成标题
tags: list[str]                         # 技术栈标签
problem: str                            # 问题描述
solution: str                           # 解决方案
key_decisions: str                      # 关键决策/踩坑点
confidence: float (0-1)                 # 置信度
status: pending | active | archived     # 状态
project: str                            # 所属项目
created_at: datetime                    # 创建时间
last_hit_at: datetime                   # 最后使用时间（用于 TTL）
metadata: ExperienceMetadata            # 结构化元数据
```

### 经验状态流转
```
PENDING → (人工 review) → ACTIVE → (90天未使用) → ARCHIVED
  ↓
 (拒绝)
  ↓
ARCHIVED
```

### Session (会话)
```python
session_id: str                    # 唯一会话 ID
task_description: str              # 任务描述
experience_ids_injected: list[str] # 注入的经验 ID
iteration_count: int               # 对话轮数
had_error_correction: bool         # 是否出现报错修正
user_accepted: bool                # 用户是否接受结果
ab_test_group: str                 # A/B 测试分组
ab_test_result_shown: bool         # 是否展示了经验
created_at: datetime               # 创建时间
```

---

## 🎯 项目核心特性

### 1. 自动沉淀 (Extraction)
- Agent 完成任务后自动提取关键信息
- 生成结构化经验（问题、方案、决策）
- 使用 LLM 生成标题（可选 OpenAI）
- 自动推断元数据（技术栈、场景、等级）

### 2. 智能检索 (Search)
- 基于向量相似度的语义搜索
- 本地 Embedding 模型（BGE-small-zh，90MB）
- 支持标签过滤和项目隔离
- A/B 测试分组（对比展示 vs 不展示）

### 3. 人工审核 (Review)
- 待确认经验逐条审核
- 支持编辑、确认、拒绝、批量操作
- 附加拒绝原因跟踪
- 置信度自动调整（0.8 for 确认，0.65 for 自动）

### 4. 效果追踪 (Analytics)
- 会话数据记录（对话轮数、报错、接受度）
- 经验采纳率统计
- A/B 测试对比分析
- 质量报告（低采纳率、未命中查询、僵尸经验等）

### 5. 文件变更检测
- 跟踪关联文件的内容变化
- 90 天未使用自动归档
- 变更时自动标记经验为过期

---

## 📝 配置文件内容

### pyproject.toml
```toml
[project]
name = "xp"
version = "0.2.0"
requires-python = ">=3.11"

dependencies = [
    "mcp[cli]>=1.0.0",
    "python-dotenv>=1.0.0",
    "sentence-transformers>=2.2.0",  # 本地 Embedding
    "numpy>=1.24.0",
    "asyncpg>=0.29.0",               # PostgreSQL
    "fastapi>=0.110.0",              # REST API
    "uvicorn>=0.29.0",
    "pgvector>=0.2.0",               # PG 向量扩展
]

[project.scripts]
xp = "src.cli:main"
xp-server = "src.server:main"
```

### .env.example
```bash
# 无需任何 API Key
# Embedding 模型在本地运行（首次自动下载 ~90MB）
# 如需更换模型，修改 src/embeddings.py
```

### GitHub Actions CI
```yaml
- 测试：Python 3.9-3.12 (4个版本)
- 排除：server 和 postgres 测试（不依赖外部服务）
- Lint：Ruff 检查致命错误
```

---

## 🎓 代码规范

### DDD 分层规则

| 层 | 禁止规则 |
|----|---------|
| `domain/` | 禁止 import `infrastructure/` 和 `application/` |
| `application/` | 只通过注入接口使用 `infrastructure/`（依赖倒置） |
| `interfaces/` | 只能调用 `CommandBus.dispatch()`（不能直接调用 Handler） |

### 测试四层分离

| 层级 | 特点 | 禁止 |
|------|------|------|
| **Unit** | 零 I/O，纯 domain 逻辑 | I/O、mock |
| **Integration** | 内存存储，完整流程 | 外部 I/O |
| **Contract** | Command/Event 契约验证 | - |
| **E2E** | 真实 REST API | - |

---

## 🔐 业务常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `CONFIDENCE_CONFIRM_VALUE` | 0.8 | 人工确认时置信度 |
| `CONFIDENCE_AUTO_ACTIVATE_INITIAL` | 0.65 | 自动激活初始置信度 |
| `CONFIDENCE_HELPFUL_DELTA` | 0.1 | 反馈有帮助时增加 |
| `CONFIDENCE_UNHELPFUL_DELTA` | -0.05 | 反馈无帮助时减少 |
| `EXPERIENCE_TTL_DAYS` | 90 | 经验 TTL（90天未使用归档） |
| `DUPLICATE_SIMILARITY_THRESHOLD` | 0.85 | 重复经验检测 |
| `LOW_ADOPTION_THRESHOLD` | 0.3 | 低采纳率警告 |
| `INFER_ADOPTION_SIMILARITY_THRESHOLD` | 0.60 | 采纳推断相似度 |
| `QUALITY_SCORE_AUTO_ACTIVATE` | 80 | 质量分数自动激活 |

---

## 📚 文档导航

| 文档 | 内容 |
|------|------|
| **README.md** | 项目首页和快速开始 |
| **QUICK_START.md** | 5分钟接入指南 |
| **CONTRIBUTING.md** | 贡献指南 + TDD + DDD 要求 |
| **docs/guides/user/cli-guide.md** | CLI 命令与 MCP 工具完整参考 |
| **docs/guides/dev/code-standards.md** | 代码规范详解（270行） |
| **docs/guides/dev/test-acceptance.md** | 测试验收规范 |
| **docs/design/adr/ADR-001-...** | 架构决策记录 |
| **docs/designs/** | 5个功能设计文档 |
| **docs/features/** | 4个功能实现文档 |
| **docs/plans/** | 8个实施计划文档 |

---

## 🚀 快速启动

```bash
# 1. 安装依赖
pip install -e .

# 2. 初始化环境（Cursor MCP 配置）
cp .env.example .env

# 3. 运行测试
python -m pytest tests/ -v -k "not server and not postgres"

# 4. 使用 CLI
xp add          # 交互式添加经验
xp review       # 审核待确认经验
xp stats        # 查看效果统计
```

---

## 📊 项目统计

| 指标 | 数值 |
|------|------|
| 源代码行数 | ~4,430 |
| CLI 代码行数 | 769 |
| MCP Server 行数 | 488 |
| 测试文件数 | 34 |
| 文档文件数 | 21 |
| 测试覆盖率 | 70%+ |
| GitHub Actions | ✓ 已配置 |

---

## 🎯 最近提交历史

```
c28dd30 docs: add ADR and feature design for implicit evolution
566050e docs: restructure documentation to match huaqi-growing guidelines
7204a8e ci: add GitHub Actions workflow for tests and linting
b1a6b94 docs: add MIT license
d0a9ded docs: add CONTRIBUTING.md guidelines
```

---

## ✅ 关键文件一览

| 文件 | 行数 | 用途 |
|------|------|------|
| `src/server.py` | 488 | MCP 服务器，定义六大工具 |
| `src/cli.py` | 769 | CLI 命令实现 |
| `src/container.py` | 94 | 依赖注入容器 |
| `src/models.py` | 110 | 核心数据模型 |
| `src/domain/constants.py` | 18 | 业务常量 |
| `src/application/commands.py` | 110 | Command 定义 |
| `src/application/command_bus.py` | 21 | Command 分发器 |
| `src/embeddings.py` | 86 | Embedding 提供者 |
| `src/storage.py` | 703 | 存储实现 |
| `pyproject.toml` | 30 | 项目配置 |

---

## 🔗 核心依赖

```
mcp[cli]>=1.0.0              # MCP 协议（Claude for Desktop 等）
sentence-transformers>=2.2.0 # 本地 Embedding 模型
python-dotenv>=1.0.0         # 环境变量管理
numpy>=1.24.0                # 数值计算
asyncpg>=0.29.0              # PostgreSQL 异步驱动
fastapi>=0.110.0             # REST API 框架
uvicorn>=0.29.0              # ASGI 服务器
pgvector>=0.2.0              # PostgreSQL 向量存储扩展
```

---

## 🎓 学习路径

1. **快速了解**：阅读 `README.md` 和 `QUICK_START.md`
2. **理解架构**：阅读 `docs/guides/dev/code-standards.md`（270行详解）
3. **掌握 MCP 工具**：阅读 `docs/guides/user/cli-guide.md`
4. **学习代码**：从 `src/server.py` 开始（488行，高注释密度）
5. **深入 DDD**：查看 `src/domain/` 和 `src/application/handlers/`
6. **运行测试**：`pytest tests/ -v` 看各层测试如何工作

这是一个 **产品级、架构清晰、文档详尽** 的项目，特别适合学习 DDD + MCP + Command Bus 架构！
