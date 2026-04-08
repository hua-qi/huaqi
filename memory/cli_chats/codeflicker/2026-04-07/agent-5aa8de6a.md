---
session_id: agent-5aa8de6a
date: 2026-04-07
time_start: 2026-04-07T09:22:36.850Z
time_end: 2026-04-07T09:25:44.322Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-07 17:22:36] [user]: 探索 /Users/lianzimeng/workspace/xp 项目，了解：
1. 这是什么项目（README、package.json等）
2. 如何在本地运行/使用
3. 是否有本地运行的配置或脚本
4. 项目的主要入口点

返回：项目类型、本地运行方式、依赖要求、以及任何已知问题或限制。
[2026-04-07 17:22:39] [assistant]: 我会帮你探索这个项目。让我从多个角度了解项目的基本信息。
[2026-04-07 17:22:43] [assistant]: 现在让我读取关键的配置文件：
[2026-04-07 17:22:46] [assistant]: 再查看一些其他重要文件：
[2026-04-07 17:22:50] [assistant]: 再查看项目的架构和主要模块：
[2026-04-07 17:22:54] [assistant]: 现在让我检查是否有 CLI 工具或其他重要的入口点：
[2026-04-07 17:22:57] [assistant]: 让我再检查一些基础架构信息：
[2026-04-07 17:23:01] [assistant]: 现在让我检查测试设置和依赖：
[2026-04-07 17:24:22] [assistant]: 现在让我查看一些测试文件来了解如何运行测试：
[2026-04-07 17:24:26] [assistant]: 让我检查Python版本和是否有虚拟环境：
[2026-04-07 17:24:30] [assistant]: 完美！现在让我获取最终的项目信息总结：
[2026-04-07 17:24:37] [assistant]: 很好！现在让我获取更完整的依赖信息：
[2026-04-07 17:24:41] [assistant]: 让我尝试直接查询.venv目录：
[2026-04-07 17:24:44] [assistant]: 让我查看是否使用uv来管理依赖：
[2026-04-07 17:24:48] [assistant]: 完美！现在让我获取更多关于项目的信息：
[2026-04-07 17:24:56] [assistant]: 太好了！现在让我获取关于架构和最近的开发工作的信息：
[2026-04-07 17:25:00] [assistant]: 现在让我获取项目的关键架构文档：
[2026-04-07 17:25:10] [assistant]: 完美！现在我有足够的信息。让我创建一个综合性的项目总结：
[2026-04-07 17:25:44] [assistant]: 完美！现在让我整理完整的项目探索报告。

## 项目探索总结报告

### 1. **项目类型与目标**

**项目名称**: XP (Experience Processor)  
**版本**: 0.3.0  
**语言**: Python 3.11+  
**项目类型**: Agent 最佳实践沉淀与知识管理工具

**核心目标**：
让 Agent（如 Cursor、codeflicker）在任何任务中越用越聪明：
- 自动沉淀每次任务的经验
- 任务开始前自动检索相关历史经验
- 通过人工审核保证质量
- 统计工具使用效果，量化研发效能提升

---

### 2. **项目架构概览**

**最新架构**：CS（客户端-服务器）架构 + MySQL 数据库  
**最新迭代**：完成 "CS 架构 + MySQL 重构" (提交: 7615ff6，Tasks 00-18)

```
Agent (Client) ──HTTP─→ FastAPI MCP Server ──→ MySQL 数据库
                                    ↓
                    向量搜索 (BGE-m3) + 余弦相似度
```

**核心层级结构**：
```
xp/
├── src/
│   ├── domain/              # 纯业务逻辑（DDD 风格）
│   ├── application/         # 编排层（Commands + Handlers）
│   ├── infrastructure/      # 存储后端 + LLM + 基础设施
│   └── server.py           # MCP FastAPI 入口
├── tests/                  # TDD 驱动的测试套件
├── pyproject.toml          # 项目配置 (uv 管理)
└── docs/                   # 架构设计文档
```

---

### 3. **本地运行配置与启动方式**

#### **环境要求**
- Python 3.11+ (当前环境：3.11.13)
- MySQL 数据库 (本地或远程)
- uv 包管理器 (可选，但推荐)

#### **本地启动步骤**

```bash
# 1. 克隆项目
git clone <repo> xp
cd xp

# 2. 环境配置
cp .env.example .env
# 编辑 .env，配置 MySQL 连接和 LLM API：
# DATABASE_URL=mysql+aiomysql://root:root@localhost:3306/xp
# XP_LLM_API_BASE=http://your-llm-api-base
# XP_LLM_API_KEY=your-api-key
# XP_LLM_MODEL=gpt-4o-mini
# XP_LLM_TIMEOUT=30

# 3. 安装依赖
uv sync  # 或 pip install -e .

# 4. 启动服务器（选项1：MCP HTTP Server）
xp-server  # 监听 0.0.0.0:8000，或设置 XP_PORT 环境变量

# 5. 接入 MCP 客户端（Cursor/codeflicker）
# 在 ~/.cursor/mcp.json 或 .cursor/mcp.json 中配置：
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

#### **MCP 工具接口（Agent 调用）**

```python
# 任务开始前：检索相关经验
search(task_description: str, project_id: str) → dict
  返回: {session_id, experiences: [...]}

# 任务完成后：保存经验
save(
    task_description: str,
    outcome_description: str,
    outcome: str,  # "success" | "partial" | "failed"
    project_id: str,
    session_id: str = ""
) → dict

# 反馈采纳/拒绝的经验
feedback(
    session_id: str,
    adopted_ids: list[str],
    rejected_ids: list[str],
    comment: str = ""
) → dict
```

---

### 4. **依赖要求与技术栈**

#### **核心依赖**
| 模块 | 用途 | 版本 |
|-----|------|------|
| `mcp[cli]` | Agent 协议 | ≥1.0.0 |
| `fastapi` | HTTP 服务器 | ≥0.110.0 |
| `uvicorn` | ASGI 应用服务器 | ≥0.29.0 |
| `aiomysql` | 异步 MySQL 连接 | ≥0.2.0 |
| `PyMySQL` | MySQL 驱动 | ≥1.1.0 |
| `sentence-transformers` (BGE-m3) | 文本向量化 | ≥3.0.0 |
| `numpy` | 向量计算（余弦相似度） | ≥1.24.0 |
| `openai` | LLM 调用（GPT） | ≥1.0.0 |
| `python-dotenv` | 环境变量管理 | ≥1.0.0 |
| `apscheduler` | 定时任务调度 | ≥3.10.0 |
| `pytest` | 测试框架 | 9.0.2 |
| `pytest-asyncio` | 异步测试支持 | ≥1.3.0 |
| `httpx` | HTTP 客户端 | ≥0.27.0 |

**数据科学栈**：PyTorch、scikit-learn、scipy、transformers、tokenizers

#### **可执行脚本**
```bash
xp-server  # 启动 FastAPI MCP 服务器（src/server.py:run())
```

---

### 5. **主要入口点**

| 入口 | 位置 | 功能 | 执行方式 |
|-----|------|------|---------|
| **MCP 服务器** | `src/server.py:run()` | 启动 HTTP MCP Server | `xp-server` 或 `python src/server.py` |
| **命令总线** | `src/container.py:build_command_bus()` | 注册所有 Command Handler | 由 server 初始化调用 |
| **核心命令** | `src/application/commands.py` | `SearchV2Command`, `SaveCommand`, `FeedbackV2Command` | 通过 MCP 工具触发 |
| **存储后端** | `src/infrastructure/backends/mysql.py` | MySQL 连接与 CRUD 操作 | 由 handlers 自动调用 |
| **向量搜索** | `src/embeddings.py` | BGE-m3 模型 + 余弦相似度 | SearchV2Handler 使用 |
| **LLM 调用** | `src/infrastructure/llm.py` | OpenAI API 包装层 | 后台异步提取经验时调用 |

---

### 6. **数据库配置**

#### **MySQL Schema 位置**
- SQL 文件：`src/infrastructure/backends/mysql_schema.sql`
- 包含表：
  - `experiences` - 存储经验
  - `sessions` - 用户会话
  - `feedback` - 采纳/拒绝反馈
  - `promotion_candidates` - 升层候选
  - `projects`, `teams`, `businesses` - 多层级知识组织
  - `project_business`, `business_team` - 层级关联表

#### **连接字符串格式**
```
mysql+aiomysql://user:password@host:port/database
```

---

### 7. **测试套件状态**

**测试统计**：
- 单元测试：169 passed, 1 skipped (~1.43s)
- 契约测试：15 passed
- 集成测试：2 passed
- **总体**：✅ 所有测试通过

**测试运行命令**：
```bash
# 运行所有测试（跳过需要 server/postgres 的）
pytest tests/ -q -k "not server and not postgres"

# 仅单元测试
pytest tests/unit -q

# 仅契约测试
pytest tests/contract -q

# 仅集成测试
pytest tests/integration -q

# 带详细输出
pytest tests/unit -v
```

**测试框架**：
- `pytest 9.0.2` + `pytest-asyncio 1.3.0`（支持异步测试）
- TDD 风格：先写测试后实现
- DDD 风格：domain 层 100% 纯函数可测

---

### 8. **已知问题与限制**

#### **当前限制**

1. **MySQL 强制依赖**
   - 不再支持本地 SQLite/JSON 回退
   - `DATABASE_URL` 为必须环境变量，缺失时抛 `RuntimeError`
   - 原 `src/storage.py` 已删除（CLI 无法独立运行）

2. **异步架构特性**
   - 所有经验提取、推断在**后台异步运行**（fire-and-forget）
   - 排查异步任务失败需要查看 MySQL 中的 `status` 字段或日志

3. **向量模型固定**
   - 仅支持 BGE-m3 (sentence-transformers)
   - 向量维度为模型输出（通常 1024）
   - MySQL BLOB 字段存储二进制向量

4. **LLM 调用限制**
   - 目前只支持 OpenAI 兼容 API（参见 `src/infrastructure/llm.py`）
   - 自动提取经验依赖轻量级 LLM 调用

#### **近期完成的工作**
- ✅ CS 架构完成（远程 HTTP Server）
- ✅ MySQL 完整迁移（Tasks 00-18）
- ✅ 层级知识系统实现（Team → Business → Project）
- ✅ MCP Streamable HTTP 支持
- ✅ 异步提取与显式反馈架构（ADR-001）

#### **正在进行的工作**
- 根据最新 commits，无明显 WIP 分支
- 文档和历史设计文档已恢复（commit 6f39a2a）

---

### 9. **快速验证清单**

```bash
# 验证环境
✓ Python 3.11+          (.venv/bin/python --version)
✓ MySQL 运行中          (mysql -u root -p -e "SELECT 1")
✓ 依赖安装完整          (uv pip list | grep -E "mcp|fastapi")
✓ 测试通过              (pytest tests/ -q --tb=no)
✓ 虚拟环境激活          (which python)
✓ MCP 配置文件就绪      (~/.cursor/mcp.json)
✓ .env 环境变量配置好   (cat .env)

# 启动服务
xp-server
# 或
python -m uvicorn src.server:mcp.streamable_http_app --host 0.0.0.0 --port 8000
```

---

### 10. **文档导航**

| 文档 | 路径 | 内容 |
|-----|------|------|
| **README** | `README.md` | 项目概述 |
| **快速开始** | `QUICK_START.md` | 5分钟接入指南 |
| **贡献指南** | `CONTRIBUTING.md` | 开发规范与 PR 流程 |
| **架构决策记录** | `docs/design/adr/` | ADR-001 ~ ADR-004 |
| **功能设计** | `docs/features/` | 各功能模块深度设计 |
| **开发指南** | `docs/guides/dev/` | 代码规范、测试标准 |
| **CLI 手册** | `docs/guides/user/cli-guide.md` | 命令行工具参考 |
| **实现计划** | `docs/plans/2026-07-04-cs-arch-mysql-refactor/` | 完整的 6 阶段重构计划 |
| **变更日志** | `CHANGELOG.md` | 版本更新记录 |

---

### 总结表格

| 维度 | 描述 |
|-----|------|
| **项目类型** | Python MCP Server 应用（DDD + TDD 驱动） |
| **架构** | 客户端-服务器 (CS)，FastAPI + MySQL + BGE-m3 向量搜索 |
| **本地运行** | `xp-server` (需要 MySQL + LLM API) |
| **依赖数** | ~65 个 Python 包（含数据科学栈） |
| **代码量** | ~2000 行核心源代码 + 详细测试 |
| **测试覆盖** | 186 个测试用例全部通过（unit+contract+integration） |
| **主要限制** | MySQL 强制，异步落库，BGE-m3 向量化 |
| **推荐环境** | macOS/Linux + Python 3.11+ + MySQL 8.0+ + OpenAI API Key |
| **开发周期** | 最近稳定迭代，无未完成分支 |
