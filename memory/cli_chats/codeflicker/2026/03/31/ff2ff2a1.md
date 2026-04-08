---
session_id: ff2ff2a1
date: 2026-03-31
time_start: 2026-03-31T16:18:35.189Z
time_end: 2026-04-01T03:06:37.646Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-01 00:18:35] [user]: 项目整体的架构是怎么样？
[2026-04-01 00:20:34] [assistant]: 以下是 XP 项目的架构概述：

## 项目定位
XP 是一个 **Agent 经验沉淀工具**，通过 MCP 协议与 IDE（Cursor/Claude）集成，自动提取、检索、注入最佳实践。

---

## 目录结构
```
src/
├── cli.py          # 命令行工具（xp add/review/stats 等）
├── server.py       # MCP Server（暴露 4 个核心 tools）
├── knowledge.py    # 知识服务层（核心业务逻辑）
├── storage.py      # 存储层（JSON + SQLite）
├── models.py       # 数据模型
├── embeddings.py   # BGE Embedding 管理
├── cloud_providers.py  # 云端存储 Provider
├── project_config.py   # 多项目隔离
├── file_watcher.py     # 文件变更监听
└── backends/       # 可插拔后端（本地/PostgreSQL）
```

---

## 核心模块

| 模块 | 职责 |
|------|------|
| `server.py` | 通过 MCP 协议暴露 `search_best_practices`、`extract_experience`、`record_feedback`、`record_session` 4 个 tools |
| `knowledge.py` | 核心业务：混合检索、经验提取、去重、TTL 管理 |
| `storage.py` | 三个存储：ExperienceStore（JSON）、MetricsStore（SQLite）、VectorStore（SQLite） |
| `embeddings.py` | 本地 BGE-small-zh 或云端 OpenAI API |
| `backends/` | LocalBackend 或 PostgresBackend，可无缝切换 |

---

## 数据流

```
IDE → MCP Server → KnowledgeService
                      ├── BM25(70%) + Embedding(30%) 混合检索
                      ├── ExperienceStore (knowledge.json)
                      ├── VectorStore (metrics.db)
                      └── MetricsStore (A/B 统计)
```

---

## 技术栈
- **语言**：Python 3.11+
- **检索**：rank-bm25 + sentence-transformers (BAAI/bge-small-zh-v1.5)
- **存储**：JSON + SQLite（本地）/ PostgreSQL + pgvector（云端）
- **协议**：MCP (stdio/SSE transport)

---

## 当前进展

| Phase | 状态 |
|-------|------|
| MVP（BM25 检索、提取、review） | 完成 |
| Embedding 混合检索 | 完成 |
| 多人共享（PostgreSQL 后端） | 进行中 |
| K8s 云端部署 | 计划中 |
[2026-04-01 00:22:29] [user]: 现在还有哪些模块依赖 BM25 ？
[2026-04-01 00:23:02] [assistant]: `knowledge.py` 的 `search` 方法注释说的是"纯向量检索"，已经不用 BM25 了。

**结论：**

目前 `rank-bm25` 库在核心逻辑中**已不再使用**。BM25 只出现在：

| 文件 | 位置 | 说明 |
|------|------|------|
| `src/cloud_providers.py` | `WeaviateProvider`（L180/257/264） | Weaviate 自带的 BM25 接口调用 |
| `src/cloud_providers.py` | `ElasticsearchProvider`（L383） | ES 自带的 BM25 搜索（注释说明） |
| `src/server.py` | L340 | 字段名 `bm25_score`（仅字符串，历史遗留） |

`knowledge.py` 的检索已迁移为**纯 Embedding 向量检索**，`rank-bm25` 依赖实际上可以移除了。
[2026-04-01 00:46:13] [user]: 好的，更新相关代码、注释和文档
[2026-04-01 00:47:47] [assistant]: 现在看 knowledge.py 中过时的注释：
[2026-04-01 00:47:52] [assistant]: knowledge.py 无需修改。现在看文档：
[2026-04-01 00:48:24] [assistant]: 现在批量更新 TECH_SPEC.md：
[2026-04-01 00:49:03] [assistant]: 现在更新 README.md：
[2026-04-01 00:49:09] [assistant]: 现在更新 ARCHITECTURE_V2.md 和 cloud_providers.py：
[2026-04-01 00:49:42] [assistant]: 现在替换第 5 节整个 BM25 检索方案章节：
[2026-04-01 00:50:33] [assistant]: cloud_providers.py 中的 BM25 是 Weaviate/ES 自带的功能，注释本身描述准确，无需修改。标记完成：
[2026-04-01 00:55:01] [assistant]: 所有更新已完成，汇总如下：

| 文件 | 变更内容 |
|------|---------|
| `pyproject.toml` | 移除 `rank-bm25` 依赖，description 改为"Embedding 向量检索" |
| `src/server.py` | `bm25_score` → `similarity` |
| `README.md` | 混合检索描述 → 纯 Embedding 向量检索 |
| `TECH_SPEC.md` | 所有 BM25 相关描述（架构图、检索逻辑、技术选型）全部更新 |
| `ARCHITECTURE_V2.md` | 第5节整个 BM25 检索方案替换为 Embedding 实现，架构图和任务清单同步更新 |

`cloud_providers.py` 无需修改，其中的 BM25 是 Weaviate/ES 第三方服务自带能力，与本地 `rank-bm25` 库无关。
[2026-04-01 11:05:27] [user]: 推送本地更改到远端
[2026-04-01 11:05:58] [assistant]: 暂存内容包含本次 BM25 清理的改动，以及之前未提交的其他更改。
[2026-04-01 11:06:37] [assistant]: 已推送，commit `1b22a7d`，20 个文件变更推送到 `origin/main`。
