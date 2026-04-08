---
session_id: agent-3628de1d
date: 2026-04-01
time_start: 2026-04-01T13:00:56.378Z
time_end: 2026-04-01T13:02:48.600Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-01 21:00:56] [user]: 请全面探索 /Users/lianzimeng/workspace/xp 这个项目的完整架构，包括：
1. 目录结构（所有主要目录和文件）
2. 主要模块和它们的职责
3. 数据存储方式（文件格式、数据库等）
4. 对外暴露的接口（CLI 命令、MCP 工具等）
5. 核心数据流（经验如何被提取、存储、检索）
6. 依赖的主要第三方库
7. 配置方式

请尽量详细，给出每个文件/模块的简要说明。最后用一段话总结整体架构风格。
[2026-04-01 21:00:59] [assistant]: 我将全面探索这个项目的完整架构。让我开始系统地分析各个方面。
[2026-04-01 21:01:02] [assistant]: 现在让我读取主要文档文件来理解架构：
[2026-04-01 21:01:06] [assistant]: 现在让我详细查看核心模块：
[2026-04-01 21:01:10] [assistant]: 现在让我查看 CLI 和 Server 模块：
[2026-04-01 21:01:13] [assistant]: 现在查看其他重要模块：
[2026-04-01 21:01:17] [assistant]: 现在让我查看测试文件和配置文件来了解更多细节：
[2026-04-01 21:01:21] [assistant]: 现在让我查看一些关键的测试文件以了解更多使用细节：
[2026-04-01 21:01:26] [assistant]: 现在让我查看更多的关键部分，比如 storage.py 的完整内容：
[2026-04-01 21:01:29] [assistant]: 现在让我总结一个完整的架构文档。让我再查看一些关键文件的完整摘要：
[2026-04-01 21:02:48] [assistant]: 完美！现在我有足够的信息来创建一个全面的架构探索报告。让我整理成一份详细的文档：

## XP 项目完整架构探索报告

根据对项目的全面分析，我为您提供以下详细的架构文档：

---

### 📋 1. 目录结构与文件分布

```
/Users/lianzimeng/workspace/xp/
├── src/                          # 核心代码目录
│   ├── __init__.py
│   ├── models.py                 # 数据模型定义（110 行）
│   ├── storage.py                # 三层存储抽象（645 行）
│   ├── knowledge.py              # 核心业务引擎（1176 行）
│   ├── embeddings.py             # Embedding 提供者（86 行）
│   ├── config.py                 # 配置管理（5 行）
│   ├── cli.py                    # CLI 命令接口（714 行）
│   ├── server.py                 # MCP 服务器（458 行）
│   ├── file_watcher.py           # 文件监听（Phase 3，140 行）
│   ├── project_config.py         # 项目配置管理（105 行）
│   ├── cloud_providers.py        # 云端存储抽象（505 行）
│   └── backends/                 # 存储后端
│       ├── base.py               # 抽象基类（47 行）
│       ├── local.py              # 本地后端（44 行）
│       └── postgres.py           # PostgreSQL 后端（Phase 3）
├── tests/                        # 测试套件
│   ├── test_extraction_refactor.py
│   ├── test_quality_score.py
│   ├── test_stats_metrics.py
│   ├── test_backends.py
│   ├── test_finalize_task.py
│   ├── test_migration.py
│   └── test_review_refactor.py
├── docs/                         # 设计文档
│   ├── embedding-refactor-design.md
│   ├── hybrid-search-design.md
│   ├── designs/
│   │   ├── 2026-03-31-ab-test-stats-refactor.md
│   │   ├── 2026-03-31-experience-extraction-refactor.md
│   │   └── 2026-03-31-xp-full-refactor-roadmap.md
│   └── plans/
├── scripts/
│   └── migrate_to_postgres.py    # 迁移脚本
├── README.md                     # 用户指南
├── ARCHITECTURE.md               # 系统架构文档
├── ARCHITECTURE_V2.md            # 多人共享架构方案
├── TECH_SPEC.md                  # 技术规范
├── pyproject.toml                # 项目配置
├── main.py                       # 启动脚本
├── .env.example                  # 环境变量示例
├── .gitignore
└── xp-wrapper.sh                 # 包装脚本
```

---

### 🎯 2. 主要模块及职责

| 模块 | 行数 | 职责描述 |
|------|------|---------|
| **knowledge.py** | 1176 | 核心业务引擎，包含经验提取、检索、反馈、质量评分、自动推断等 |
| **storage.py** | 645 | 三层存储抽象（JSON 经验数据 + SQLite 向量索引 + SQLite 指标） |
| **cli.py** | 714 | CLI 命令行工具（14 个命令：add, review, edit, stats, analyze 等） |
| **server.py** | 458 | MCP 协议服务器（7 个 MCP 工具暴露给 IDE） |
| **cloud_providers.py** | 505 | 云端存储抽象（Supabase, Weaviate, Elasticsearch, Phase 3） |
| **models.py** | 110 | 数据模型定义（Experience, Session, Feedback, ProjectConfig 等） |
| **embeddings.py** | 86 | Embedding 提供者（本地 BGE 或云端 OpenAI） |
| **project_config.py** | 105 | 项目隔离和配置管理（Phase 3） |
| **file_watcher.py** | 140 | 文件变更监听和 TTL 管理（Phase 3） |
| **backends/** | 91 | 存储后端抽象（本地、PostgreSQL） |

**关键数字**：
- 总代码量：~4,500+ 行
- 核心代码（知识库、存储、业务逻辑）：~2,000 行
- 接口层（CLI、Server）：~1,200 行
- Phase 3 扩展代码：~650 行

---

### 💾 3. 数据存储方式

#### 三层存储架构

```
Layer 1: 经验数据（JSON 结构化）
└─ ~/.xp/knowledge.json
   ├── Experience {
   │   ├── id: UUID
   │   ├── type: bugfix/feature/pattern
   │   ├── level: L1/L2/L3
   │   ├── title: 自动生成或人工编辑
   │   ├── problem: 问题描述
   │   ├── solution: 解决方案
   │   ├── key_decisions: 关键决策（质量核心字段）
   │   ├── confidence: float (0-1)
   │   ├── status: pending/active/archived
   │   ├── metadata {
   │   │   ├── tech_stack: [react, typescript, ...]
   │   │   ├── scene: [表单, 异步, 路由, ...]
   │   │   └── problem_type: string
   │   ├── related_files: [src/hooks/useAuth.ts, ...]
   │   ├── file_hashes: {path -> SHA256(content)[:16]}
   │   ├── project: default
   │   └── created_at: ISO8601
   └─ 批量操作：add/get/update/delete/list_by_status
   
Layer 2: 向量索引（SQLite BLOB）
└─ ~/.xp/metrics.db[experience_vectors]
   ├── experience_id: TEXT PRIMARY KEY
   ├── vector: BLOB (384 维 float32 二进制压缩)
   │  • 1 条向量 = 1.5 KB (384 * 4 bytes)
   │  • 1000 条 = 1.5 MB
   ├── updated_at: 更新时间
   └── 操作类：VectorStore
       ├── save_vector(exp_id, vector)
       ├── get_vector(exp_id) -> np.ndarray
       └── get_all_vectors(exp_ids) -> (ids, np.ndarray)

Layer 3: 指标与事件（SQLite 表）
└─ ~/.xp/metrics.db
   ├── sessions (会话记录)
   │   ├── session_id: 唯一标识
   │   ├── experience_ids_injected: JSON 数组
   │   ├── iteration_count: 对话轮数
   │   ├── had_error_correction: boolean
   │   ├── user_accepted: boolean
   │   ├── ab_test_group: control/treatment
   │   └── created_at: 时间戳
   │
   ├── search_events (搜索事件)
   │   ├── query: 搜索查询
   │   ├── result_count: 返回结果数
   │   └── created_at: 时间戳
   │
   ├── review_events (审核事件)
   │   ├── experience_id
   │   ├── action: confirmed/rejected/edited
   │   ├── reject_reason: 可选原因
   │   └── created_at
   │
   ├── feedback (用户反馈)
   │   ├── experience_id
   │   ├── adopted: boolean
   │   ├── reason: 可选原因
   │   └── created_at
   │
   └── experience_stats (经验统计)
       ├── experience_id: PK
       ├── hit_count: 被命中次数
       ├── adopted_count: 被采纳次数
       ├── adoption_rate: 采纳率
       └── last_used_at: 最后使用时间
```

#### 数据流示例

```
提取经验流程：
  task_description + solution_summary + key_decisions
  ↓
  ExperienceStore.add(exp) → knowledge.json（写入）
  ↓
  embeddings.embed_texts([problem, solution, key_decisions]) 
  → BGE-small-zh-v1.5 (384 维向量)
  ↓
  VectorStore.save_vector(exp_id, vector) → metrics.db (压缩 BLOB)

检索流程：
  search_best_practices(query, session_id)
  ↓
  embeddings.embed_texts([query]) → 384 维向量
  ↓
  VectorStore.get_all_vectors() → 所有经验向量 (N × 384)
  ↓
  cosine_similarity(query_vec, doc_vecs) → 相似度数组
  ↓
  过滤 score ≥ 0.5，TopK 返回
  ↓
  MetricsStore.record_search(query, result_count)
```

---

### 🔌 4. 对外暴露的接口

#### MCP 工具（7 个，通过 IDE 调用）

```python
# src/server.py - MCP Server 实现

1. search_best_practices(query, tags?, top_k?, session_id)
   ├─ 功能：检索历史经验（任务开始第一步）
   ├─ 返回：TopK 经验 + 相似度分数 + A/B 测试分组
   ├─ 触发：任务开始前必调
   └─ 处理流程：查询向量化 → 余弦相似度 → 过滤 & 排序 → A/B 分组

2. extract_experience(task_description, solution_summary, key_decisions, 
                      conversation_summary?, tags?, related_files?)
   ├─ 功能：提取并沉淀经验（任务完成后必调）
   ├─ 返回：新创建的 Experience 对象
   ├─ 质量门禁：
   │   ├─ key_decisions ≥ 10 字（建议 ≥ 50 字）
   │   ├─ solution_summary ≥ 20 字（建议 ≥ 50 字）
   │   └─ 质量评分 ≥ 40（触发自动激活）
   ├─ 自动推断：tech_stack, scene, problem_type, level, type
   └─ 质量评分项：key_decisions(40分) + solution(20分) + files(15分) 
                  + tech_stack(15分) + 去重(10分)

3. record_session(session_id, task_description, experience_ids_injected,
                  iteration_count, had_error_correction, user_accepted, 
                  ab_test_group?)
   ├─ 功能：记录会话数据（任务完成后必调）
   ├─ 用途：A/B 测试、效果评估、数据追踪
   ├─ A/B 分组：hash(session_id) % 10 == 0 ? control : treatment
   └─ 存储：MetricsStore.record_session(Session)

4. record_feedback(experience_id, adopted, reason?)
   ├─ 功能：反馈经验采纳情况
   ├─ 影响：更新 confidence、自动归档（< 0.1）
   └─ 公式：new_confidence = old × 0.9 + adoption_rate × 0.1

5. finalize_task(session_id, task_description, experience_ids_injected,
                 iteration_count, had_error_correction, user_accepted)
   ├─ 功能：一键完成（Phase 2 合并工具）
   └─ 等价于：record_session() + infer_adoption()

6. infer_adoption(session_id, final_response, experience_ids_injected)
   ├─ 功能：自动推断经验采纳情况（Phase 2）
   ├─ 方法：检查 final_response 中是否包含经验中的关键代码/决策
   └─ 自动反馈：已采纳 → record_feedback(adopted=true)

7. (计划中) sync(direction=up/down, project?)
   ├─ 功能：云端同步（Phase 3）
   └─ 支持提供商：Supabase, Weaviate, Elasticsearch
```

#### CLI 命令（14 个，通过终端调用）

```bash
# src/cli.py - CLI 实现

# 经验管理
xp add                              # 交互式手动添加
xp review                           # 逐条审核 pending 经验
xp edit <ID前缀>                   # 编辑并重新激活
xp archive <ID前缀>                # 归档经验
xp delete <ID前缀>                 # 永久删除
xp import <文件>                   # 从 Markdown 批量导入
xp show <ID前缀>                   # 显示单条经验详情（Phase 1）

# 统计和分析
xp stats [--since 7d/30d]          # 查看效果统计
xp analyze                         # 质量分析报告

# 项目管理
xp project list                    # 列出所有项目
xp project switch <名>             # 切换项目
xp init                            # 初始化新项目（Phase 2）

# 扩展功能
xp watch                           # 文件变更监听（Phase 3）
xp sync up/down                    # 云端同步（Phase 3）

# 管理员
xp migrate                         # 数据迁移脚本
```

---

### 📊 5. 核心数据流（经验的完整生命周期）

#### 5.1 检索流程（任务开始）

```
IDE Agent 调用 search_best_practices(query="修复 React useEffect 无限循环", session_id="session-001")
                                     ↓
                          KnowledgeService.search()
                                     ↓
                    ┌──────────────────────────────────────┐
                    │     1. 加载所有 active 经验            │
                    │     store.list_active() → [Exp1,...]  │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │   2. 向量化查询 + 所有经验              │
                    │   query_vec = embed(query) (384维)     │
                    │   exp_vecs = get_all_vectors() (N×384) │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │   3. 计算相似度 (余弦相似度)           │
                    │   scores = cosine_similarity(         │
                    │     query_vec, exp_vecs) → [0.72, ...]│
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │   4. 过滤和排序                        │
                    │   filter score >= 0.5                 │
                    │   sort by score DESC                  │
                    │   topk(k=3, score=[0.72, 0.65, 0.58]) │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │   5. A/B 测试分组                      │
                    │   ab_test_group = hash(session_id)%10 │
                    │   == 0 ? "control" : "treatment"      │
                    │   control(10%) → 返回空经验            │
                    │   treatment(90%) → 正常返回            │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │   6. 记录搜索事件                      │
                    │   MetricsStore.record_search(         │
                    │     query, result_count=3)            │
                    │   写入 metrics.db[search_events]      │
                    └──────────────────────────────────────┘
                                     ↓
返回给 IDE: {
  "experiences": [
    {
      "id": "exp-001",
      "title": "useEffect 依赖项遗漏导致无限循环",
      "problem": "...",
      "solution": "...",
      "key_decisions": "...",
      "similarity": 0.72,
      "metadata": {
        "tech_stack": ["react", "hooks"],
        "scene": ["渲染优化"],
        "problem_type": "bugfix"
      }
    },
    ...
  ],
  "metadata": {
    "session_id": "session-001",
    "ab_test_group": "treatment",
    "ab_test_result_shown": true
  }
}
```

#### 5.2 提取流程（任务完成）

```
IDE Agent 调用 extract_experience(
  task_description="修复登录后状态不保持",
  solution_summary="在 token 过期时自动刷新，并存入 localStorage",
  key_decisions="不要在 useEffect 里直接 setState，会导致无限循环",
  tags=["react", "auth"],
  related_files=["src/hooks/useAuth.ts"]
)
                                     ↓
                          KnowledgeService.extract_experience()
                                     ↓
                    ┌──────────────────────────────────────┐
                    │     1. 验证必填字段                    │
                    │     key_decisions.len() >= 10         │
                    │     solution_summary.len() >= 20      │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │     2. 检查重复（向量相似度）          │
                    │     similarity(new_text, all_exp) 检查 │
                    │     < 0.85 才允许保存                 │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │     3. 自动推断元数据                  │
                    │     tech_stack = _infer_tech_stack()  │
                    │       → 关键词匹配 (react, ts, ...)    │
                    │     scene = _infer_scene()            │
                    │       → 40+ 预定义标签 (表单, 异步...)│
                    │     type = _infer_type()              │
                    │       → bugfix / feature / pattern    │
                    │     level = _infer_level()            │
                    │       → L1 / L2 / L3                 │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │     4. 计算质量评分（质量门禁）        │
                    │     score = _compute_quality_score()  │
                    │                                        │
                    │     key_decisions 内容   40 分         │
                    │     solution 内容        20 分         │
                    │     related_files 非空   15 分         │
                    │     tech_stack 推断      15 分         │
                    │     去重距离             10 分         │
                    │     ─────────────────────────         │
                    │     总分: score                        │
                    │     < 40  → 拒绝                       │
                    │     40-79 → pending (等待人工 review)  │
                    │     >= 80 → active (自动激活)          │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │     5. 生成标题                        │
                    │     try:                              │
                    │       title = LLM(task_description)   │
                    │     except:                           │
                    │       title = _make_title(task[:30])  │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │     6. 向量化 + 存储                   │
                    │     all_text = task + solution + key  │
                    │     vector = embed(all_text) (384维)  │
                    │     ExperienceStore.add(exp)          │
                    │       → write knowledge.json          │
                    │     VectorStore.save_vector(id, vec)  │
                    │       → write metrics.db[vectors]     │
                    └──────────────────────────────────────┘
                                     ↓
返回给 IDE: Experience {
  id: "uuid-xxx",
  status: active | pending,  # 取决于质量评分
  confidence: 0.65,          # auto-activate 时为 0.65
  title: "登录状态保持问题修复",
  quality_score: 85,
  metadata: {
    tech_stack: ["react", "auth"],
    scene: ["登录", "状态管理"],
    problem_type: "bugfix"
  }
}

↓ 后续流程:
如果 status == pending:
  engineer 运行 xp review
  ├─ [y] 确认     → status=active, confidence=0.8
  ├─ [e] 编辑后确认 → status=active, confidence=1.0
  ├─ [n] 拒绝    → status=archived, reject_reason="..."
  └─ [s] 跳过    → 保持 pending
```

#### 5.3 反馈流程（任务结束）

```
IDE Agent 或 engineer 调用 record_feedback(
  experience_id="exp-001",
  adopted=true,  # 或 false
  reason="成功修复了登录状态问题"
)
                                     ↓
                          MetricsStore.record_feedback()
                                     ↓
                    ┌──────────────────────────────────────┐
                    │   1. 写入反馈记录                      │
                    │   feedback = Feedback(...)            │
                    │   metrics.db[feedback].insert()       │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │   2. 更新经验统计                      │
                    │   hit_count += 1                      │
                    │   if adopted:                         │
                    │     adopted_count += 1                │
                    │   adoption_rate = adopted_count /     │
                    │                   hit_count           │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │   3. 更新信心度（动态调整）            │
                    │   new_confidence = old × 0.9 +        │
                    │     adoption_rate × 0.1               │
                    │                                        │
                    │   example:                            │
                    │   old_conf = 0.65                     │
                    │   adoption_rate = 0.8 (4/5)           │
                    │   new_conf = 0.65 × 0.9 +             │
                    │              0.8 × 0.1                │
                    │            = 0.585 + 0.08             │
                    │            = 0.665                    │
                    └──────────────────────────────────────┘
                                     ↓
                    ┌──────────────────────────────────────┐
                    │   4. 自动归档（低信心度）              │
                    │   if new_confidence < 0.1:            │
                    │     status = archived                 │
                    │     不再参与检索                       │
                    └──────────────────────────────────────┘
```

---

### 📦 6. 主要第三方库及依赖

#### 直接依赖

```toml
# pyproject.toml

mcp[cli]>=1.0.0                    # MCP 协议实现（IDE 集成）
python-dotenv>=1.0.0               # 环境变量加载
sentence-transformers>=2.2.0       # BGE-small-zh-v1.5 embedding 模型
numpy>=1.24.0                      # 向量运算（余弦相似度）
asyncpg>=0.29.0                    # PostgreSQL 异步驱动（Phase 3）
fastapi>=0.110.0                   # Web 框架（Phase 3 中心化）
uvicorn>=0.29.0                    # ASGI 服务器（Phase 3）
pgvector>=0.2.0                    # PostgreSQL 向量扩展（Phase 3）
```

#### 可选依赖（云端）

```python
# 云端存储 provider 可选导入
supabase>=2.0.0                    # Supabase PostgreSQL
weaviate-client>=4.0.0             # Weaviate 向量数据库
elasticsearch>=8.0.0               # Elasticsearch
```

#### 内置/标准库

```python
asyncio                            # 异步编程
json                               # 经验数据序列化
sqlite3                            # 本地向量索引和指标存储
pathlib                            # 文件路径操作
uuid                               # 经验 ID 生成
struct                             # 二进制编码（向量压缩）
hashlib                            # 文件 hash 计算
argparse                           # CLI 参数解析
datetime                           # 时间戳
```

#### 关键版本信息

- **Python**: ≥ 3.11（使用类型注解、异步特性）
- **Embedding 模型**: BAAI/bge-small-zh-v1.5（384 维，~90MB）
- **向量计算**: NumPy 上的余弦相似度（归一化向量，点积即相似度）

---

### ⚙️ 7. 配置方式

#### 7.1 环境变量

```bash
# .env / .env.example

# 无需任何 API Key，embedding 在本地运行
# （可选）OpenAI API Key - 用于 LLM 标题生成
OPENAI_API_KEY=sk-...

# XP 数据目录
XP_HOME=~/.xp              # 默认：$HOME/.xp

# Embedding 模型缓存
XP_EMBEDDING_CACHE=~/.cache/xp/embeddings

# 云端同步配置（Phase 3）
XP_CLOUD_PROVIDER=supabase
XP_SUPABASE_URL=https://...
XP_SUPABASE_KEY=...

# 权限管理
XP_ADMIN_KEY=...           # 管理员密钥
XP_USER_KEY=...            # 用户密钥（需与 admin key 相同才能 review）
```

#### 7.2 项目配置

```json
// ~/.xp/projects.json

{
  "default": {
    "name": "default",
    "tags": ["python", "typescript"],
    "root_path": "/path/to/project",
    "cloud_sync_enabled": false,
    "cloud_provider": null,
    "cloud_config": {}
  },
  "my-project": {
    "name": "my-project",
    "tags": ["react", "web"],
    "root_path": "/path/to/my-project",
    "cloud_sync_enabled": true,
    "cloud_provider": "supabase",
    "cloud_config": {
      "url": "https://...",
      "key": "..."
    }
  }
}
```

#### 7.3 当前项目

```text
// ~/.xp/current_project
my-project
```

#### 7.4 MCP 集成配置

```json
// ~/.cursor/mcp.json （Cursor 配置示例）

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

---

### 🏗️ 8. 技术栈选择的考量

#### 为什么选择这些技术？

```
┌─ 向量化 (Embedding)
│  ├─ BGE-small-zh-v1.5 (本地)
│  │  ✓ 专为中文优化，384 维，精度高
│  │  ✓ 本地推理，离线可用，无 API 限流
│  │  ✓ 与开源生态兼容（Weaviate/Elasticsearch）
│  └─ OpenAI text-embedding-3 (可选云端)
│     ✓ 文本嵌入能力更强
│
├─ 向量相似度
│  └─ 余弦相似度 (NumPy 实现)
│     ✓ 归一化向量下，简单高效
│     ✓ 与预训练模型对齐
│
├─ 本地存储
│  ├─ JSON (知识库)
│     ✓ 人可读，易于版本控制
│     ✓ 结构化元数据直观
│  ├─ SQLite (向量 + 指标)
│     ✓ 无依赖，零配置
│     ✓ BLOB 存储向量，高效查询
│     ✓ 支持事务，数据一致性强
│  └─ PostgreSQL (Phase 3)
│     ✓ 扩展性好，多用户并发
│     ✓ pgvector 原生向量支持
│
├─ 接口协议
│  ├─ MCP (Model Context Protocol)
│  │  ✓ IDE 原生支持（Cursor/Claude/Windsurf）
│  │  ✓ 无需额外网络配置
│  │  ✓ 零成本集成
│  └─ CLI (argparse)
│     ✓ 简单强大，无额外依赖
│
└─ 云端 (Phase 3 规划)
   ├─ Supabase (PostgreSQL + Auth)
   ├─ Weaviate (专业向量 DB)
   └─ Elasticsearch (搜索 + 向量)
```

---

### 📈 9. 数据规模与性能估算

#### 本地存储规模

```
小团队（10 人，6 个月）
  经验数        100-300 条
  knowledge.json   2-5 MB
  向量索引         200-450 KB
  指标数据库       1-2 MB
  总计             3-7 MB
  典型查询时间     < 100ms

中等团队（50 人，1 年）
  经验数        500-1500 条
  knowledge.json   10-40 MB
  向量索引         750-2.2 MB
  指标数据库       5-20 MB
  总计             15-60 MB
  典型查询时间     100-300ms

大型团队（100+ 人）
  经验数        5000+ 条
  → 推荐迁移到 PostgreSQL + pgvector
  → 建议启用云端同步
```

#### 向量存储效率

```
BGE-small-zh-v1.5:
  维度           384
  数据类型       float32
  单向量大小     384 × 4 = 1.5 KB
  压缩方法       struct.pack (BLOB)
  压缩率         ~100%（无损）
  
1000 条经验向量:
  原始大小       1.5 MB (384 × 4000 × 1000)
  压缩后         1.5 MB (struct BLOB)
  查询性能       全向量 np.array 计算 < 1ms
  
相似度计算:
  算法           余弦相似度 (dot product)
  查询时间       O(N × 384) = O(N)
  1000 条: ~20-50ms (NumPy 优化)
  过滤 + 排序    < 5ms
  总耗时         < 100ms
```

---

### 🚀 10. 整体架构风格总结

**XP 是一个设计简洁、扩展性强、以经验沉淀为中心的 AI Agent 工具链**。

核心特点：

1. **"三层分离"架构**
   - 接口层：MCP（IDE）+ CLI（终端）
   - 业务层：知识库引擎（提取、检索、反馈、质量评分）
   - 存储层：JSON + SQLite（本地） + 云端可选（PostgreSQL/Weaviate/ES）

2. **"渐进式设计"**
   - Phase 1（已完成）：本地 JSON + SQLite，零依赖，零配置，开箱即用
   - Phase 2（进行中）：质量评分、自动激活、分析报告
   - Phase 3（计划中）：文件追踪、多人共享、云端同步

3. **"向量原生"方案**
   - 抛弃关键词搜索，拥抱 Embedding（BGE-small-zh）
   - 纯向量相似度 + 阈值过滤，简洁高效
   - 支持本地推理 + 云端 API 两种模式

4. **"数据流驱动"**
   - 经验流：提取 → 质量评分 → 人工审核 → 反馈优化（自动调整信心度）
   - 会话流：搜索 → 使用 → 反馈，形成完整的 A/B 测试闭环
   - 关键指标：采纳率、命中率、效果对比

5. **"易用优先"**
   - MCP：IDE 中一键调用，零学习成本
   - CLI：简明扼要，交互友好
   - 存储：本地文件（knowledge.json），可版本控制、可迁移
   - 扩展：插件式设计（云端 Provider、存储后端、Embedding Provider）

**适用场景**：

✅ 个人工程师、小团队快速沉淀经验
✅ Code Review、文档编写、Bug 修复提速
✅ 跨项目、跨团队知识共享（Phase 3）
✅ 与 AI Agent（Claude/Cursor）深度集成

**不适用场景**：

❌ 超大规模知识库（100k+ 条）- 需专业向量 DB
❌ 实时协作编辑 - 数据一致性要求高
❌ 复杂权限管理 - 目前仅有基础 Admin/User 隔离

---

这就是 XP 项目的完整架构！它是一个**轻量级、高效、面向 AI Agent 的经验沉淀工具**，通过 Embedding 向量检索和反馈驱动的质量优化，让团队经验得到最大化复用。
