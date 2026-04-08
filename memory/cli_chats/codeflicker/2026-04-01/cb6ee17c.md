---
session_id: cb6ee17c
date: 2026-04-01
time_start: 2026-04-01T03:46:25.861Z
time_end: 2026-04-01T09:46:18.298Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-01 11:46:25] [user]: 给我一份当前项目的整体架构图、所有模块的详细设计和信息流转流程图
[2026-04-01 11:59:45] [assistant]: 以下是 XP 项目的完整架构文档：

---

## XP 项目整体架构

### 一、系统定位

XP 是一个 **AI Agent 经验沉淀与复用工具**，通过 MCP 协议集成到 IDE（Cursor/Claude），自动沉淀任务经验，检索复用。

---

### 二、整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        外部接入层                                │
│                                                                  │
│   ┌─────────────────┐           ┌─────────────────────────────┐ │
│   │   IDE Agent     │           │      工程师 CLI              │ │
│   │ (Cursor/Claude) │           │   (xp review/add/stats...)  │ │
│   └────────┬────────┘           └──────────────┬──────────────┘ │
└────────────┼───────────────────────────────────┼────────────────┘
             │ MCP Protocol                       │ argparse
             ▼                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                        接口层                                    │
│   ┌──────────────────────────┐   ┌────────────────────────────┐ │
│   │       server.py          │   │          cli.py             │ │
│   │  MCP Server (7 Tools)    │   │   14 个 CLI 命令            │ │
│   │                          │   │                             │ │
│   │  search_best_practices   │   │  review / add / edit        │ │
│   │  extract_experience      │   │  stats / analyze            │ │
│   │  record_session          │   │  project / sync / watch     │ │
│   │  record_feedback         │   │  import / archive / delete  │ │
│   │  finalize_task           │   │                             │ │
│   │  infer_adoption          │   │                             │ │
│   └──────────────┬───────────┘   └───────────────┬────────────┘ │
└──────────────────┼───────────────────────────────┼──────────────┘
                   │                               │
                   ▼                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                        核心业务层                                │
│                                                                  │
│   ┌────────────────────────────────────────────────────────┐    │
│   │                    knowledge.py                         │    │
│   │                   (1176 行，核心引擎)                   │    │
│   │                                                         │    │
│   │  ┌─────────────┐  ┌─────────────┐  ┌───────────────┐  │    │
│   │  │  提取模块    │  │  检索模块    │  │  质量评分模块  │  │    │
│   │  │             │  │             │  │               │  │    │
│   │  │ 字段验证     │  │ 标签过滤     │  │ 40分:key_dec  │  │    │
│   │  │ 去重检测     │  │ Embedding   │  │ 20分:solution │  │    │
│   │  │ 自动推断     │  │ 余弦相似度   │  │ 15分:files    │  │    │
│   │  │ 质量评分     │  │ 阈值过滤≥.5 │  │ 15分:stack    │  │    │
│   │  │ 向量生成     │  │ TopK 返回   │  │ 10分:去重距离 │  │    │
│   │  └─────────────┘  └─────────────┘  └───────────────┘  │    │
│   │                                                         │    │
│   │  ┌─────────────┐  ┌─────────────┐  ┌───────────────┐  │    │
│   │  │  反馈模块    │  │  A/B 测试   │  │  自动推断模块  │  │    │
│   │  │             │  │             │  │               │  │    │
│   │  │ confidence  │  │ hash分组    │  │ 10+技术栈     │  │    │
│   │  │ 动态更新     │  │ 10%对照组   │  │ 40+场景标签   │  │    │
│   │  │ 自动归档     │  │ 效果对比    │  │ 问题类型推断   │  │    │
│   │  └─────────────┘  └─────────────┘  └───────────────┘  │    │
│   └────────────────────────────────────────────────────────┘    │
│                                                                  │
│   ┌──────────────────────────┐   ┌────────────────────────────┐ │
│   │       embeddings.py      │   │        models.py            │ │
│   │                          │   │                             │ │
│   │  BGE-small-zh-v1.5       │   │  Experience                 │ │
│   │  (384 维, ~90MB)          │   │  Session                    │ │
│   │  本地推理, 离线可用        │   │  Feedback                   │ │
│   │  归一化余弦相似度          │   │  ExperienceMetadata         │ │
│   └────────────┬─────────────┘   └────────────────────────────┘ │
└────────────────┼────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        存储层（三层）                            │
│                                                                  │
│   Layer 1: 经验数据               Layer 2: 向量索引              │
│   ┌─────────────────────┐        ┌──────────────────────────┐   │
│   │  ~/.xp/knowledge    │        │  ~/.xp/metrics.db        │   │
│   │      .json          │        │  [experience_vectors]    │   │
│   │                     │        │                          │   │
│   │  JSON 结构化存储     │        │  384维 BLOB 压缩          │   │
│   │  ExperienceStore    │        │  1条 = 1.5KB             │   │
│   │  add/get/update/    │        │  VectorStore             │   │
│   │  delete/list        │        │  save/get/delete         │   │
│   └─────────────────────┘        └──────────────────────────┘   │
│                                                                  │
│   Layer 3: 指标与事件                                            │
│   ┌────────────────────────────────────────────────────────┐    │
│   │  ~/.xp/metrics.db                                       │    │
│   │                                                         │    │
│   │  sessions          search_events     review_events      │    │
│   │  feedback          experience_stats  (A/B分组结果)       │    │
│   └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼ (Phase 3 扩展)
┌─────────────────────────────────────────────────────────────────┐
│                      云端层（规划中）                            │
│   Supabase / Weaviate / Elasticsearch / PostgreSQL+pgvector      │
│   cloud_providers.py / backends/postgres.py                      │
└─────────────────────────────────────────────────────────────────┘
```

---

### 三、核心数据模型

```
Experience
├── id: UUID
├── type: ExperienceType        # bugfix / feature / pattern
├── level: ExperienceLevel      # L1 架构 / L2 场景 / L3 修复
├── title, problem, solution
├── key_decisions               # 关键！质量评分核心字段
├── confidence: float           # 动态调整，< 0.1 自动归档
├── status: pending/active/archived
├── metadata
│   ├── tech_stack: [react, ts, python...]   # 自动推断，10+ 种
│   ├── scene: [表单, 异步, 路由...]          # 自动推断，40+ 种
│   └── problem_type: str
├── related_files, file_hashes  # Phase 3：失效检测
└── project                     # Phase 3：多项目隔离
```

---

### 四、信息流转流程图

#### 4.1 任务执行完整生命周期

```
┌─────────────────────────────────────────────────────────────────┐
│ 阶段 1：任务开始 - 检索注入                                      │
│                                                                  │
│  Agent ──search_best_practices(query)──▶ knowledge.py           │
│                                                ↓                 │
│                                   1. 加载所有 active 经验        │
│                                   2. 计算 query Embedding        │
│                                   3. 余弦相似度排序              │
│                                   4. 过滤 score >= 0.5           │
│                                   5. A/B 分组判断               │
│                                          ↓                      │
│           ◀────── 返回 TopK 经验 (含 experience_ids) ───────    │
└─────────────────────────────────────────────────────────────────┘
                               ↓
┌─────────────────────────────────────────────────────────────────┐
│ 阶段 2：任务完成 - 提取沉淀                                      │
│                                                                  │
│  Agent ──extract_experience(...)──▶ knowledge.py                │
│                                          ↓                      │
│                          ┌───────────────────────────────┐      │
│                          │         质量门禁               │      │
│                          │  key_decisions >= 10字 ✓       │      │
│                          │  solution >= 20字 ✓            │      │
│                          │  重复检测 < 0.85 ✓             │      │
│                          └───────────────┬───────────────┘      │
│                                          ↓                      │
│                          ┌───────────────────────────────┐      │
│                          │         自动推断               │      │
│                          │  tech_stack (关键词匹配)        │      │
│                          │  scene (40+ 预定义标签)         │      │
│                          │  problem_type (模式匹配)        │      │
│                          └───────────────┬───────────────┘      │
│                                          ↓                      │
│                          ┌───────────────────────────────┐      │
│                          │         质量评分               │      │
│                          │  key_decisions 内容: 40分      │      │
│                          │  solution 内容:    20分        │      │
│                          │  related_files:   15分         │      │
│                          │  tech_stack:      15分         │      │
│                          │  去重距离:         10分         │      │
│                          └───────────────┬───────────────┘      │
│                                          ↓                      │
│                       score<40 → 拒绝                           │
│                       40-79   → pending (等待人工 review)        │
│                       >=80    → active  (confidence=0.65)        │
│                                          ↓                      │
│              knowledge.json ◀── 写入经验                         │
│              metrics.db     ◀── 写入 Embedding 向量              │
└─────────────────────────────────────────────────────────────────┘
                               ↓
┌─────────────────────────────────────────────────────────────────┐
│ 阶段 3：人工审核 (xp review)                                     │
│                                                                  │
│  工程师 ──xp review──▶ cli.py ──▶ knowledge.py                  │
│                                          ↓                      │
│              列出所有 pending 经验（含质量评分）                  │
│                                          ↓                      │
│     [y] 确认 ──▶ status=active,  confidence=0.8                 │
│     [e] 编辑 ──▶ status=active,  confidence=1.0                 │
│     [n] 拒绝 ──▶ status=archived                                │
│     [s] 跳过 ──▶ 保持 pending                                   │
└─────────────────────────────────────────────────────────────────┘
                               ↓
┌─────────────────────────────────────────────────────────────────┐
│ 阶段 4：反馈驱动优化                                             │
│                                                                  │
│  Agent ──record_feedback(id, adopted)──▶ knowledge.py           │
│                                          ↓                      │
│       metrics.db.feedback ◀── 记录采纳/拒绝事件                  │
│       metrics.db.experience_stats ◀── 更新统计                  │
│                                          ↓                      │
│       new_confidence = old × 0.9 + adoption_rate × 0.1          │
│                                          ↓                      │
│       confidence < 0.1 ──▶ 自动归档（不再参与检索）              │
└─────────────────────────────────────────────────────────────────┘
```

#### 4.2 A/B 测试流程

```
search_best_practices(session_id)
         ↓
hash(session_id) % 10 == 0?
    ├── YES (10%) → 对照组: ab_test_group="control"
    │              返回空经验，不注入
    └── NO  (90%) → 实验组: ab_test_group="treatment"
                   正常检索注入经验
         ↓
record_session(session_id, user_accepted, iteration_count)
         ↓
xp stats → 对比两组效果
    对照组：平均轮数 5.7，报错率 41%，接受率 67%
    实验组：平均轮数 3.2，报错率 18%，接受率 84%
```

---

### 五、模块依赖关系

```
server.py ──▶ knowledge.py ──▶ storage.py
                │                  ├── ExperienceStore (knowledge.json)
                │                  ├── VectorStore     (metrics.db)
                │                  └── MetricsStore    (metrics.db)
                │
                ├──▶ embeddings.py ──▶ BGE-small-zh / OpenAI API
                └──▶ models.py

cli.py ──▶ knowledge.py (复用同一套业务逻辑)
       └──▶ cloud_providers.py (Phase 3: sync up/down)

backends/postgres.py ──▶ asyncpg (Phase 3: 替换 SQLite)
file_watcher.py      ──▶ watchdog (Phase 3: 文件变更监听)
project_config.py    ──▶ 项目隔离配置
```

---

### 六、各模块详细说明

| 模块 | 行数 | 职责 |
|------|------|------|
| `knowledge.py` | 1176 | 核心引擎：提取/检索/反馈/质量评分/自动推断 |
| `storage.py` | 646 | 三层存储抽象：JSON + SQLite 向量 + SQLite 指标 |
| `cli.py` | 714 | 14 个 CLI 命令交互界面 |
| `server.py` | 458 | MCP Server，7 个工具暴露给 IDE |
| `cloud_providers.py` | 505 | 云同步（Supabase/Weaviate/ES，Phase 3） |
| `backends/postgres.py` | 298 | PostgreSQL 异步后端（Phase 3） |
| `embeddings.py` | 86 | BGE-small-zh-v1.5 向量化，384维 |
| `models.py` | 110 | 5 个数据类定义 |
| `file_watcher.py` | — | 文件变更监听，失效检测（Phase 3） |
| `project_config.py` | — | 多项目配置隔离（Phase 3） |
[2026-04-01 15:04:50] [user]: 把这篇文档使用 md 格式沉淀下来
[2026-04-01 15:17:08] [assistant]: 已保存到 `ARCHITECTURE.md`，包含完整的架构图、模块说明、信息流转流程图、存储层设计、质量评分系统等全部内容。
[2026-04-01 15:51:52] [user]: 现在系统是 CS 架构？
[2026-04-01 15:52:04] [assistant]: 不是。当前（Phase 1/2）是**本地单机架构**：

- MCP Server、知识库、向量存储全部运行在**用户本机**
- 数据存储在 `~/.xp/` 目录，无需网络
- IDE Agent 通过本地 MCP 协议连接，本质是进程间通信

CS 架构是 **Phase 3 规划中**的目标（FastAPI 中心化服务 + PostgreSQL），用于支持多人团队共享知识库。
[2026-04-01 16:27:58] [user]: 仔细查看代码后，再回答这个问题呢
[2026-04-01 16:32:54] [assistant]: 不是 CS 架构。仔细看代码后，结论更明确：

**当前是本地单机架构，MCP 通信基于 stdio（标准输入/输出），不是网络通信。**

关键证据：

```python
# server.py
from mcp.server.stdio import stdio_server

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, ...)
```

- `server.py` 是 MCP stdio Server，**不监听任何端口**
- `pyproject.toml` 虽声明了 `fastapi` / `uvicorn` 依赖，但**源码中完全未使用**
- `backends/postgres.py` 只是存储适配器，agent/CLI 直连 DB，没有服务层
- `cloud_providers.py` 是手动触发的同步工具，不是实时共享

**真实架构**：

```
IDE/Claude ──stdio──▶ MCP Server（本地进程）──▶ ~/.xp/（本地文件）
```

CS 架构（FastAPI + 中心化服务）是 Phase 3 的规划，**尚未实现**。
[2026-04-01 16:37:54] [user]: 现在经验数据都存储在 ~/.xp/ 中吗？不支持用户自定义经验存储目录吗？
[2026-04-01 16:38:12] [assistant]: 支持自定义，通过环境变量 `XP_HOME` 控制存储目录。

但代码有两个问题：

1. **`storage.py` 和 `project_config.py` 的默认值被硬编码成了你本机的绝对路径**，而不是通用的 `~/.xp`：
   ```python
   # storage.py & project_config.py（错误）
   XP_HOME = Path(os.environ.get("XP_HOME", "/Users/lianzimeng/workspace/xp-data"))
   
   # file_watcher.py（正确）
   XP_HOME = Path(os.environ.get("XP_HOME", Path.home() / ".xp"))
   ```

2. **三个文件各自独立读取 `XP_HOME`**，没有统一的配置入口，存在不一致风险。

需要修复 `storage.py` 和 `project_config.py` 的默认值吗？
[2026-04-01 16:40:55] [user]: 1. 修复这个问题
2. 各个文件都统一读取 XP_HOME 配置项
[2026-04-01 16:41:15] [assistant]: 新建一个 `config.py` 作为统一配置入口，再修改各文件引用它。
[2026-04-01 16:42:26] [assistant]: 验证一下没有遗漏：
[2026-04-01 16:42:32] [assistant]: `XP_HOME` 的定义现在只在 `config.py` 中存在一处。修改内容：

- 新增 `src/config.py`，统一定义 `XP_HOME`，默认值修正为 `Path.home() / ".xp"`
- `storage.py`、`project_config.py`、`file_watcher.py` 均改为从 `config.py` 导入，移除各自的 `os.environ` 读取逻辑
[2026-04-01 17:46:02] [user]: 推送本地更改到远端
[2026-04-01 17:46:18] [assistant]: 已推送到远端，commit `27b66a6`，包含 5 个文件变更。
