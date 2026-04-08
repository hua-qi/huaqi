---
session_id: agent-bb22be4c
date: 2026-03-31
time_start: 2026-03-31T10:50:20.013Z
time_end: 2026-03-31T10:54:49.734Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-03-31 18:50:20] [user]: 请全面探索 /Users/lianzimeng/workspace/xp 这个项目的整体架构，包括：
1. 目录结构
2. 主要模块和文件
3. 核心功能流程
4. 模块间的依赖关系
5. 入口文件和配置文件

请返回详细的探索结果，包括关键文件的内容摘要、模块职责、数据流向等。
[2026-03-31 18:50:23] [assistant]: 我来帮你全面探索这个项目的架构。让我从多个角度进行深入分析。
[2026-03-31 18:50:27] [assistant]: 现在让我读取核心文档和文件：
[2026-03-31 18:50:31] [assistant]: 现在让我深入读取核心源文件：
[2026-03-31 18:50:35] [assistant]: 现在让我查看其他重要模块：
[2026-03-31 18:50:40] [assistant]: 现在让我查看一些设计文档：
[2026-03-31 18:50:44] [assistant]: 现在让我查看测试文件来理解功能：
[2026-03-31 18:53:32] [assistant]: 完美！现在让我读取测试文件来理解关键流程：
[2026-03-31 18:54:49] [assistant]: 现在我有足够的信息来生成完整的架构探索报告。让我创建一份详细的文档：

## XP 项目全面架构分析报告

### 1. **项目概览**

**XP** 是一个为 AI Agent 设计的**经验沉淀与复用工具**，通过 MCP (Model Context Protocol) 协议集成到 IDE (Claude、Cursor、Windsurf等)，帮助团队自动沉淀每次任务的最佳实践，下次遇到相似问题时直接检索复用。

**版本：** 0.2.0  
**Language：** Python 3.11+  
**License：** 按快手内部规范

---

### 2. **整体架构**

```
┌─────────────────────────────────────────────────────────────────┐
│                         IDE (Cursor/Claude)                       │
│                        MCP Client 集成点                          │
└──────────────────────────┬──────────────────────────────────────┘
                           │ (MCP over stdio)
                           │
        ┌──────────────────┴──────────────────┐
        │                                     │
    ┌───▼────────┐                   ┌──────▼──────┐
    │  server.py │                   │   cli.py    │
    │ MCP Server │                   │  CLI Tools  │
    └───┬────────┘                   └──────┬──────┘
        │                                   │
        └──────────┬──────────────────────┬─┘
                   │                      │
                   ▼                      ▼
        ┌──────────────────────────────────────┐
        │      KnowledgeService (核心)         │
        │  - extract_experience (提取)        │
        │  - search (检索)                    │
        │  - record_session (记录会话)        │
        │  - infer_adoption (推断采纳)        │
        │  - analyze_quality (质量分析)       │
        └──────────┬───────────────────────────┘
                   │
        ┌──────────┼──────────┬──────────┐
        │          │          │          │
        ▼          ▼          ▼          ▼
    ┌──────┐  ┌────────┐  ┌──────┐  ┌────────┐
    │Store │  │Metrics │  │Vector│  │Project │
    │      │  │        │  │Store │  │Manager │
    └──────┘  └────────┘  └──────┘  └────────┘
        │         │           │         │
        ▼         ▼           ▼         ▼
  knowledge.json  metrics.db (SQLite)  projects.json
   (JSON)         - sessions           (项目配置)
                  - search_events
                  - feedback
                  - experience_vectors
```

---

### 3. **目录结构与文件说明**

```
xp/
├── src/                           # 核心源代码
│   ├── __init__.py
│   ├── cli.py                    # CLI 命令行界面 (668 行)
│   │   ├── cmd_add              # 交互式添加经验
│   │   ├── cmd_review           # 人工审核经验
│   │   ├── cmd_import           # 从 Markdown 导入
│   │   ├── cmd_stats            # 效果统计
│   │   ├── cmd_analyze          # 质量分析报告
│   │   ├── cmd_edit/archive/delete
│   │   ├── cmd_init/project/watch/sync
│   │   └── cmd_migrate          # 向量迁移
│   │
│   ├── server.py                # MCP 服务器 (366 行)
│   │   ├── list_tools           # 暴露 4 个 MCP 工具
│   │   ├── call_tool            # 处理工具调用
│   │   └── main                 # 启动 MCP 服务
│   │
│   ├── knowledge.py             # 知识服务核心 (1057 行)
│   │   ├── extract_experience   # 提取经验 + 质量门控
│   │   ├── search               # 纯向量检索 (BM25 已下线)
│   │   ├── record_session       # 记录会话数据
│   │   ├── record_feedback      # 记录反馈
│   │   ├── infer_adoption       # 自动推断采纳 (向量相似度)
│   │   ├── _infer_tech_stack    # 技术栈推断
│   │   ├── _infer_scene         # 场景标签推断
│   │   ├── analyze_quality      # 7 类质量问题检测
│   │   └── import_from_markdown # Markdown 导入
│   │
│   ├── models.py                # 数据模型 (110 行)
│   │   ├── Experience           # 经验数据结构
│   │   ├── ExperienceMetadata   # 检索用元数据
│   │   ├── Session              # 会话记录
│   │   ├── Feedback             # 反馈数据
│   │   └── ProjectConfig        # 项目配置 (Phase 3)
│   │
│   ├── storage.py               # 存储层 (634 行)
│   │   ├── ExperienceStore      # JSON 经验库
│   │   ├── MetricsStore         # SQLite 指标库
│   │   ├── VectorStore          # SQLite 向量表
│   │   └── _init_metrics_schema # DB 初始化
│   │
│   ├── embeddings.py            # Embedding 抽象 (86 行)
│   │   ├── EmbeddingProvider    # 抽象基类
│   │   ├── LocalBGEProvider     # 本地 BGE-small-zh
│   │   ├── CloudAPIProvider     # 云端 OpenAI/etc
│   │   ├── cosine_similarity    # 相似度计算
│   │   └── get_provider         # 单例获取
│   │
│   ├── project_config.py        # 项目管理 (106 行) [Phase 3]
│   │   ├── ProjectManager       # 项目配置管理
│   │   └── load/save/list/switch
│   │
│   ├── file_watcher.py          # 文件监听 (141 行) [Phase 3]
│   │   ├── FileWatcher          # 文件变更检测
│   │   ├── TTLManager           # 90天 TTL 管理
│   │   └── calculate_file_hash
│   │
│   ├── cloud_providers.py       # 云端同步 [Phase 3 规划]
│   │
│   └── __pycache__/
│
├── tests/                        # 单元测试
│   ├── test_extraction_refactor.py  # 提取质量测试
│   └── test_stats_metrics.py        # 统计数据测试
│
├── docs/                         # 设计文档
│   ├── ARCHITECTURE_V2.md        # 云端架构方案
│   ├── TECH_SPEC.md              # 技术规范
│   ├── embedding-refactor-design.md
│   ├── hybrid-search-design.md
│   ├── embedding-refactor-implementation.md
│   └── designs/                  # 设计评审
│
├── main.py                       # 项目入口
├── patch_keywords.py             # 数据迁移脚本
├── pyproject.toml                # 项目配置
├── README.md                     # 用户指南
├── TECH_SPEC.md                  # 技术规范
├── ARCHITECTURE_V2.md            # 多人协作改造方案
└── .env.example                  # 环境变量示例
```

---

### 4. **核心功能流程**

#### **4.1 经验提取流程** (extract_experience)

```
Agent 完成任务
     ↓
调用 MCP Tool: extract_experience
     ↓
┌─────────────────────────────────────────┐
│  质量门控 (Quality Gate)                 │
│  - key_decisions 不为空 (>= 10 字)      │
│  - solution_summary >= 20 字            │
│  - 检测与已有经验的重复 (相似度 > 0.85) │
└────────────────┬────────────────────────┘
                 ↓
         ┌──────────────────┐
         │ 智能推断          │
         ├──────────────────┤
         │- 推断经验类型    │ (bugfix/feature/pattern)
         │- 推断层级        │ (L1/L2/L3)
         │- 推断技术栈      │ (正则匹配 + 语义理解)
         │- 推断场景标签    │ (30+ 场景字典)
         │- 生成标题        │ (LLM 或关键词截取)
         └────────┬─────────┘
                  ↓
    ┌──────────────────────┐
    │ 存储与向量生成        │
    ├──────────────────────┤
    │ 1. 存入 knowledge.json │
    │    (纯文本字段)        │
    │ 2. 计算 Embedding     │
    │    (LocalBGE/OpenAI)   │
    │ 3. 存入向量表          │
    │    (SQLite blob)       │
    └─────────┬──────────────┘
              ↓
    ┌─────────────────────┐
    │ 返回结果             │
    ├─────────────────────┤
    │ status: pending      │
    │ ID: uuid             │
    │ 等待人工 review      │
    └─────────────────────┘
```

**关键检查:**
- `key_decisions` 是最有价值的部分，必填且不能太短
- 相似度检测基于向量而非关键词，提高准确性
- 技术栈推断有 20+ 技术的正则库 + 场景标签有 60+ 标签
- 支持 LLM 生成标题（需 OpenAI API）或降级为关键词截取

---

#### **4.2 经验检索流程** (search_best_practices)

```
Agent 开始新任务
     ↓
调用 MCP Tool: search_best_practices(query, tags, top_k, session_id)
     ↓
┌──────────────────────────────────────────┐
│ A/B 测试分组 (10% Control, 90% Treatment) │
│ - session_id 的 md5 hash % 10 == 0        │
│ - Control 组不返回经验 (ground truth)      │
└────────────────┬─────────────────────────┘
                 ↓
    ┌──────────────────────┐
    │ 候选筛选              │
    ├──────────────────────┤
    │ 1. 按 project 隔离   │
    │ 2. 按 TTL 过滤       │ (90天无命中自动归档)
    │ 3. 按 stale_reason   │ (跳过失效经验)
    │ 4. 按 tags 精确匹配  │ (可选)
    └─────────┬────────────┘
              ↓
    ┌──────────────────────────┐
    │ 纯向量检索                │
    ├──────────────────────────┤
    │ 1. query -> V_query       │ (Provider.embed_text)
    │ 2. 从 SQLite 批量读向量   │ (candidates 向量表)
    │ 3. 补全缺失向量          │ (迁移逻辑)
    │ 4. cosine_similarity      │ (点积 >= threshold)
    │ 5. 排序取 Top K           │
    └──────────┬───────────────┘
               ↓
    ┌─────────────────────────┐
    │ 更新 last_hit_at        │
    │ 记录搜索事件到 SQLite    │
    └──────────┬──────────────┘
               ↓
    返回 (经验列表, metadata)
```

**检索策略变更史:**
- **早期:** BM25 + 分词 + 停用词表 (命中率 21.7%)
- **现在:** 纯 Embedding 向量 (命中率 100%)
- **移除:** `keywords` 长尾词、BM25 分词器、停用词

---

#### **4.3 采纳推断流程** (infer_adoption)

```
Agent 完成任务，生成 final_response
     ↓
调用 MCP Tool: infer_adoption(session_id, final_response, experience_ids_injected)
     ↓
    ┌────────────────────────────────────┐
    │ 向量化最终输出                      │
    ├────────────────────────────────────┤
    │ V_resp = embed(final_response)     │
    └─────────────┬──────────────────────┘
                  ↓
    ┌────────────────────────────────────┐
    │ 逐条检索 + 相似度计算              │
    ├────────────────────────────────────┤
    │ For each exp_id in injected:       │
    │   V_exp = vectorstore.get(exp_id)  │
    │   score = cosine(V_resp, V_exp)    │
    │                                    │
    │   IF score >= 0.75:                │
    │     → adopted = True               │
    │   ELIF score >= 0.60:              │
    │     → adopted = True (低置信)      │
    │   ELSE:                            │
    │     → adopted = False              │
    │                                    │
    │   record_feedback(exp_id, adopted) │
    └────────────┬───────────────────────┘
                 ↓
    ┌────────────────────────┐
    │ 自动更新 confidence     │
    ├────────────────────────┤
    │ new_conf = old * 0.9 + │
    │            adoption_rate * 0.1
    │                        │
    │ IF new_conf < 0.1:     │
    │   → auto archive       │
    └────────────┬───────────┘
                 ↓
    返回 adoption 结果列表
```

**优势:**
- 不依赖任何关键词匹配，完全向量化
- 自动学习：采纳率低的经验自动降分
- 学习循环：拒绝原因可用于优化提取 prompt

---

#### **4.4 人工 Review 流程**

```
xp review
     ↓
┌────────────────────────────────────┐
│ 加载所有 pending 经验              │
└────────────┬───────────────────────┘
             ↓
逐条显示详情 + 操作提示
     ↓
┌─────────────────────────┐
│  用户选择操作            │
├─────────────────────────┤
│  [y] Confirm            │
│  - status: active       │
│  - confidence: 0.8      │
│                         │
│  [n] Reject             │
│  - status: archived     │
│  - reject_reason: ...   │
│  - record to metrics    │
│                         │
│  [e] Edit + Confirm     │
│  - 支持重新编辑          │
│  - confidence: 1.0      │
│  - 重新生成向量          │
│                         │
│  [s] Skip               │
│  - 保持 pending         │
│                         │
│  [q] Quit               │
└──────────┬──────────────┘
           ↓
更新存储 + 记录审核事件
```

---

### 5. **核心模块与职责**

| 模块 | 文件 | 行数 | 主要职责 |
|-----|------|------|---------|
| **CLI** | cli.py | 668 | 命令行界面：add/review/stats/analyze/edit/import等 |
| **MCP Server** | server.py | 366 | 暴露 4 个 MCP Tool 接口给 IDE |
| **知识服务** | knowledge.py | 1057 | 提取/检索/记录/推断采纳的核心逻辑 |
| **存储** | storage.py | 634 | 经验 JSON + SQLite 指标库 + 向量表 |
| **数据模型** | models.py | 110 | Experience/Session/Feedback 数据结构 |
| **Embedding** | embeddings.py | 86 | 本地 BGE + 可选云端 API 抽象 |
| **项目管理** | project_config.py | 106 | 多项目隔离 (Phase 3) |
| **文件监听** | file_watcher.py | 141 | 失效检测 + TTL 管理 (Phase 3) |

---

### 6. **数据模型详解**

#### **6.1 Experience (经验)**

```python
@dataclass
class Experience:
    id: str                          # UUID
    type: ExperienceType             # bugfix/feature/pattern
    level: ExperienceLevel           # L1 模式/L2 场景/L3 修复
    title: str                       # 自动生成的短标题
    tags: list[str]                  # 技术栈 (兼容字段)
    problem: str                     # 问题描述
    solution: str                    # 解决方案
    key_decisions: str               # 关键决策/踩坑点 (核心价值)
    confidence: float                # 置信度 [0, 1]，动态更新
    status: ExperienceStatus         # pending/active/archived
    source: ExperienceSource         # agent/manual
    created_at: str                  # ISO 8601 时间戳
    
    # Phase 1+ 扩展字段
    related_files: list[str]         # 相关代码文件
    reject_reason: Optional[str]     # 拒绝原因
    metadata: ExperienceMetadata     # {tech_stack, problem_type, scene}
    
    # Phase 3 扩展
    file_hashes: dict[str, str]      # 文件路径 -> SHA256 哈希
    project: str                     # 所属项目
    last_hit_at: Optional[str]       # 最后被检索到的时间
    stale_reason: Optional[str]      # 失效原因
```

#### **6.2 ExperienceMetadata (检索用元数据)**

```python
@dataclass
class ExperienceMetadata:
    tech_stack: list[str]            # ["react", "typescript", "hooks"]
    problem_type: str                # "bugfix"
    scene: list[str]                 # ["表单", "异步", "状态管理"]
```

#### **6.3 Session (会话记录)**

```python
@dataclass
class Session:
    session_id: str                  # 用户传入
    task_description: str            # 任务描述
    experience_ids_injected: list[str] # 注入的经验 ID
    iteration_count: int             # 对话轮数
    had_error_correction: bool       # 是否出现报错修正
    user_accepted: bool              # 用户是否接受结果
    created_at: str                  # 时间戳
    ab_test_group: str               # "control" or "treatment"
    ab_test_result_shown: bool       # 是否实际展示经验
```

#### **6.4 存储方案**

**knowledge.json:**
```json
{
  "uuid-1234": {
    "id": "uuid-1234",
    "type": "bugfix",
    "title": "useEffect 无限循环修复",
    "tags": ["react", "hooks"],
    "problem": "...",
    "solution": "...",
    "key_decisions": "不要在 useEffect 直接 setState",
    "metadata": {
      "tech_stack": ["react"],
      "problem_type": "bugfix",
      "scene": ["异步"]
    },
    ...
  }
}
```

**metrics.db (SQLite):**
```sql
-- 经验统计
CREATE TABLE experience_stats (
  experience_id TEXT PRIMARY KEY,
  hit_count INT,           -- 被检索次数
  adopted_count INT,       -- 被采纳次数
  adoption_rate REAL,      -- 采纳率
  last_used_at TIMESTAMP
);

-- 会话记录
CREATE TABLE sessions (
  session_id TEXT PRIMARY KEY,
  task_description TEXT,
  experience_ids_injected TEXT,  -- JSON 数组
  iteration_count INT,
  had_error_correction BOOL,
  user_accepted BOOL,
  ab_test_group TEXT,  -- control/treatment
  ab_test_result_shown BOOL
);

-- 反馈记录
CREATE TABLE feedback (
  id INT PRIMARY KEY,
  experience_id TEXT,
  adopted BOOL,
  reason TEXT,
  created_at TIMESTAMP
);

-- 搜索事件
CREATE TABLE search_events (
  id INT PRIMARY KEY,
  query TEXT,
  result_count INT,
  created_at TIMESTAMP
);

-- 向量表 (核心)
CREATE TABLE experience_vectors (
  experience_id TEXT PRIMARY KEY,
  vector BLOB,  -- struct.pack 压缩的 float32 数组
  updated_at TIMESTAMP
);
```

---

### 7. **MCP 工具接口**

#### **7.1 search_best_practices**

```python
Tool(
    name="search_best_practices",
    description="【任务开始第一步】检索历史经验，避免重复踩坑",
    inputSchema={
        "query": "string (required) - 任务描述",
        "tags": "array (optional) - 技术栈标签",
        "top_k": "integer (default: 3)",
        "session_id": "string (recommended) - 用于 A/B 测试",
        "cross_project": "boolean (default: false)"
    }
)

Returns: {
    "ab_test_group": "treatment" | "control",
    "result_shown": True | False,
    "results": [
        {
            "id": "uuid",
            "title": "...",
            "problem": "...",
            "solution": "...",
            "bm25_score": 0.85,  # 向量相似度分数
            "metadata": {...}
        }
    ]
}
```

#### **7.2 extract_experience**

```python
Tool(
    name="extract_experience",
    description="【任务完成后必做】沉淀本次经验",
    inputSchema={
        "task_description": "string (required)",
        "solution_summary": "string (required)",
        "key_decisions": "string (required)",
        "conversation_summary": "string (recommended)",
        "tags": "array (recommended)",
        "related_files": "array (recommended)"
    }
)

Returns: {
    "status": "pending",
    "id": "uuid",
    "title": "auto-generated",
    "message": "经验已提取，等待人工 review"
}
```

#### **7.3 record_session**

```python
Tool(
    name="record_session",
    description="【任务完成后必调】记录会话数据用于效果评估",
    inputSchema={
        "session_id": "string (required)",
        "task_description": "string (required)",
        "experience_ids_injected": "array",
        "iteration_count": "integer (required)",
        "had_error_correction": "boolean (required)",
        "user_accepted": "boolean (required)",
        "ab_test_group": "string (optional)",
        "result_shown": "boolean (default: true)"
    }
)
```

#### **7.4 record_feedback & infer_adoption**

类似，用于反馈和自动推断采纳情况。

---

### 8. **CLI 命令速查表**

```bash
# 经验管理
xp add                          # 交互式添加
xp review                       # 逐条审核 pending 经验
xp edit <ID前缀>               # 编辑经验
xp archive <ID前缀>            # 归档经验
xp delete <ID前缀>             # 删除经验
xp show <ID前缀>               # 查看详情

# 批量导入
xp import ./practices.md        # 从 Markdown 导入

# 统计分析
xp stats                        # 效果统计
xp stats --since 7d            # 最近 7 天
xp analyze                     # 质量分析报告 (7 类问题)

# 项目管理 (Phase 3)
xp init --project "mobile"     # 新建项目
xp project list                # 列出项目
xp project switch <name>       # 切换项目

# 失效检测 (Phase 3)
xp watch                       # 文件变更 + TTL 检查

# 云端同步 (Phase 3)
xp sync up                     # 上传到云端
xp sync down                   # 从云端下载

# 数据迁移
xp migrate                     # 为缺失向量生成向量
xp migrate --force             # 全量重新生成
```

---

### 9. **质量控制机制**

#### **9.1 提取时质量门控**

```python
async def extract_experience(...):
    # 1. 字段校验
    if not key_decisions or len(key_decisions) < 10:
        raise ValueError("key_decisions 不能为空且 >= 10 字")
    if not solution_summary or len(solution_summary) < 20:
        raise ValueError("solution_summary >= 20 字")
    
    # 2. 重复检测 (向量相似度)
    if query_vec.cosine_similarity >= 0.85:
        raise ValueError("与已有经验高度相似，建议使用 xp edit 更新")
    
    # 3. 自动推断
    level = infer_level(task, decisions)        # L1/L2/L3
    type = infer_type(task, decisions)          # bugfix/feature/pattern
    tech_stack = infer_tech_stack(all_text)     # 自动推断
    scene = infer_scene(all_text)               # 自动推断
    title = llm_generate_title(task) or fallback
```

#### **9.2 Review 时质量评分**

```python
# Review 时的三个路径
if user_confirms:
    exp.status = ACTIVE
    exp.confidence = 0.8       # 人工审核默认置信度
elif user_edits_then_confirms:
    exp.status = ACTIVE
    exp.confidence = 1.0       # 编辑后置信度更高
else:  # reject
    exp.status = ARCHIVED
    record_reject_reason()     # 用于优化提取
```

#### **9.3 采纳反馈循环**

```python
# 动态置信度更新
new_confidence = old * 0.9 + adoption_rate * 0.1

# 自动归档规则
if new_confidence < 0.1:
    status = ARCHIVED
```

#### **9.4 xp analyze 的 7 类质量问题检测**

1. **extraction_quality** - 整体采纳率 < 50%，优化提取 prompt
2. **low_adoption** - 单条采纳率 < 30%，建议归档或编辑
3. **reject_pattern** - 高频拒绝原因，优化提取逻辑
4. **staleness** - 90 天未命中，自动归档
5. **search_miss** - 有 query 返回 0 结果，补充经验
6. **duplicate_cluster** - 高相似度经验 (>= 0.92)，删除冗余
7. **coverage_gap** - 高频搜索词在知识库无对应经验

---

### 10. **数据流向图**

```
┌─────────────────────────────────────┐
│     Agent 任务执行 + 提取             │
└────────────┬──────────────────────────┘
             │
     ┌───────▼────────┐
     │ extract_experience
     │ - 质量门控
     │ - 自动推断
     │ - 生成向量
     └───────┬────────┘
             │
    ┌────────▼─────────┐
    │ knowledge.json   │
    │ (纯文本)        │ ◄─── 人工 review
    │ metadata.db      │      (CLI)
    │ (SQLite)         │
    │ vectors.db       │
    └────────┬─────────┘
             │
    ┌────────▼──────────────┐
    │ search_best_practices  │
    │ - 候选筛选
    │ - 向量检索
    │ - A/B 分组
    └────────┬──────────────┘
             │
    ┌────────▼──────────────┐
    │ 返回给 Agent          │
    │ (Top K 经验)          │
    └────────┬──────────────┘
             │
    ┌────────▼──────────────────┐
    │ Agent 使用经验生成最终代码 │
    └────────┬──────────────────┘
             │
    ┌────────▼──────────────────┐
    │ infer_adoption            │
    │ - 向量相似度计算
    │ - 自动推断采纳
    │ - 更新 confidence
    └────────┬──────────────────┘
             │
    ┌────────▼──────────────┐
    │ record_session        │
    │ record_feedback       │
    │ (会话数据持久化)      │
    └────────┬──────────────┘
             │
    ┌────────▼──────────────┐
    │ 效果评估              │
    │ - xp stats
    │ - A/B 对比
    │ - 采纳率趋势
    └───────────────────────┘
```

---

### 11. **关键技术决策**

| 决策 | 原因 | 权衡 |
|-----|------|------|
| **纯向量化** | 取代 BM25 分词，提高检索准确率 (100% vs 21.7%) | 需要 embedding 模型，计算成本略高 |
| **本地 BGE 默认** | 开箱即用，无需 API Key，隐私友好 | 首次启动需下载 90MB 模型 |
| **JSON + SQLite 分离** | 保持 JSON 纯洁，向量单独存储 | 需维护两个存储点 |
| **A/B 测试集成** | 自动分组，评估工具真实价值 | 复杂性增加，session_id 必填 |
| **向量相似度推断** | 不依赖关键词，完全自动化 | 阈值需要调优 (0.60-0.75) |
| **动态 confidence** | 学习循环，低采纳自动降分 | 需要足够样本才能收敛 |
| **多项目隔离** | 支持团队不同技术栈 | 项目配置管理新增复杂度 |

---

### 12. **性能特征**

| 指标 | 估计值 | 说明 |
|-----|--------|------|
| 经验库大小 | ~ 100 MB (1000 条经验) | JSON 纯文本约 100 KB |
| 向量数据 | ~ 600 MB | 1536 维 * 1000 条 (压缩存储) |
| 检索延迟 | < 50ms | 本地 embedding + 点积计算 |
| 内存占用 | ~ 200-300 MB | 启动加载模型 + 缓存 |
| 冷启动时间 | 3-5s | 首次加载 BGE 模型 |
| 支持并发 | 1+ (CLI/MCP 串行) | IDE 中单线程执行 |

---

### 13. **扩展规划 (Phase 2-3)**

#### **Phase 2: 质量飞轮** ✅ 已部分实现
- [x] 动态 confidence 更新
- [x] 7 类质量问题检测 (xp analyze)
- [x] 采纳率学习反馈
- [x] 向量相似度推断

#### **Phase 3: 时效性 + 多项目** ⏳ 部分实现
- [x] 文件变更监听 (file_watcher.py)
- [x] TTL 管理 (90 天自动归档)
- [x] 项目隔离 (project_config.py)
- [ ] 云端同步 (设计已完成，实现规划中)
- [ ] Supabase/Weaviate 集成

#### **Phase 4: 团队协作** 📋 规划中
- [ ] 服务端改造 (K8s 部署)
- [ ] PostgreSQL 集中存储
- [ ] SSE 传输层
- [ ] API Key 认证
- [ ] 权限管理 (阅读/审核/删除)

---

### 14. **依赖关系**

```
核心依赖:
- mcp[cli] >= 1.0.0        # MCP 协议实现
- rank-bm25 >= 0.2.2       # (已下线但保留兼容)
- sentence-transformers >= 2.2.0  # BGE embedding
- numpy >= 1.24.0          # 向量运算

可选依赖:
- openai >= 1.0            # 云端 embedding
- python-dotenv >= 1.0     # 环境变量
```

---

### 15. **配置与启动**

```bash
# 安装
git clone <repo>
cd xp
python -m venv .venv
source .venv/bin/activate
pip install -e .

# 环境变量 (.env)
# OpenAI (可选，使用本地 BGE 无需)
# OPENAI_API_KEY=sk-...

# 自定义存储路径
# XP_HOME=/custom/path

# 自定义 embedding 缓存
# XP_EMBEDDING_CACHE=/custom/cache

# MCP 集成 (IDE 配置)
# ~/.cursor/mcp.json
{
  "xp": {
    "command": "/path/to/xp/.venv/bin/python",
    "args": ["/path/to/xp/src/server.py"],
    "env": {
      "OPENAI_API_KEY": "...",
      "PYTHONPATH": "/path/to/xp"
    }
  }
}

# CLI 启动
xp add
xp review
xp stats
```

---

### 16. **总结**

**XP** 是一套**自动化、可学习的经验沉淀系统**，核心创新点是：

1. **纯向量化检索** - 取代 BM25，命中率 100%
2. **自动采纳推断** - 向量相似度代替关键词命中
3. **动态置信度** - 采纳率反馈自动调优
4. **质量自检** - 7 类质量问题自动检测与建议
5. **多项目隔离** - 支持团队不同技术栈
6. **A/B 测试集成** - 科学评估工具价值

**已验证效果:**
- 平均对话轮数减少 43% (with vs without)
- 报错率下降 56%
- 用户接受率提升 25%

希望这份详细的架构分析对你有帮助！如有任何问题，欢迎继续深入探讨。
