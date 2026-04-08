---
session_id: agent-df84e5c4
date: 2026-03-30
time_start: 2026-03-30T09:33:38.832Z
time_end: 2026-03-30T09:35:35.413Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-03-30 17:33:38] [user]: 探索 /Users/lianzimeng/workspace/huaqi-growing 的代码结构，重点了解：
1. 整体目录结构（ls 根目录和 huaqi_src/ 目录）
2. 现有的 tools.py 内容（工具如何定义）
3. 现有的 collectors/ 目录（如果存在）
4. 现有的 reports/ 目录（如果存在）
5. 现有的 world/ 目录（如果存在）
6. 现有的 agent/graph/chat.py（Agent 如何组织）
7. 数据模型 - 查找 HuaqiDocument 或类似结构
8. SQLite 相关代码（storage 或 db 相关文件）
9. Chroma 相关代码
10. 现有测试的结构（tests/ 目录）
11. pyproject.toml 或 requirements.txt（依赖）
12. config 相关文件

请返回：每个关键文件的路径和主要内容摘要（特别是类定义、函数签名、import 结构）。这是为了写实施计划，需要了解现有代码的模式和约定。
[2026-03-30 17:33:44] [assistant]: 我将系统地探索这个代码库的结构。让我开始进行多层次的深入了解。
[2026-03-30 17:33:47] [assistant]: 现在让我读取关键文件。开始从最重要的文件开始：
[2026-03-30 17:33:51] [assistant]: 继续读取更多关键文件：
[2026-03-30 17:33:55] [assistant]: 继续读取更多关键文件：
[2026-03-30 17:33:58] [assistant]: 继续读取更多文件：
[2026-03-30 17:34:04] [assistant]: 现在读取测试文件和更多关键代码：
[2026-03-30 17:34:09] [assistant]: 现在读取测试相关和配置相关的文件：
[2026-03-30 17:34:13] [assistant]: 现在读取一些关键的scheduler和pipeline相关文件：
[2026-03-30 17:34:17] [assistant]: 现在让我读取一些更关键的内存和搜索相关文件：
[2026-03-30 17:34:22] [assistant]: 最后让我快速查看一些其他重要的文件结构和配置：
[2026-03-30 17:34:27] [assistant]: 现在让我查看一些关键的适应性理解和配置文件：
[2026-03-30 17:34:30] [assistant]: 完美！现在让我创建一个综合总结文档。让我组织所有信息：
[2026-03-30 17:35:35] [assistant]: 现在让我准备一份详细的代码结构总结：

## 代码库完整探索总结

我已深入探索了整个代码库。以下是完整的代码结构和关键文件摘要，用于实施计划。

---

## **1. 整体架构总览**

```
huaqi-growing/
├── huaqi_src/                  # 核心源代码
│   ├── agent/                  # LangGraph Agent 架构 ✨
│   ├── cli/                    # CLI 命令行界面
│   ├── core/                   # 核心系统 (配置、LLM、用户理解等)
│   ├── collectors/             # 数据收集器 (Document Model)
│   ├── integrations/           # 外部集成 (WeChat Webhook)
│   ├── memory/                 # 记忆系统 (向量DB, 搜索)
│   ├── pipeline/               # 内容流水线 (RSS抓取、小红书生成)
│   ├── reports/                # 报告生成 (晨间简报)
│   ├── scheduler/              # 定时任务系统 (APScheduler)
│   └── world/                  # 世界新闻系统
├── tests/                      # 测试套件 (127个Python文件)
├── pyproject.toml              # 项目配置
├── requirements.txt            # 依赖
└── docs/                       # 文档和设计
```

**总计：127个 Python 文件，结构清晰、模块化程度高**

---

## **2. 关键子系统详解**

### **2.1 Agent 系统 (huaqi_src/agent/)**

**文件结构：**
- `state.py` - LangGraph 状态定义
- `tools.py` - Tool 节点（搜索工具）
- `nodes/chat_nodes.py` - 对话节点实现
- `nodes/interrupt_nodes.py` - 人机协同中断节点
- `graph/chat.py` - 对话流程编排

**关键数据结构：**

```python
# Agent State 定义 (state.py)
class AgentState(TypedDict):
    messages: Annotated[List[BaseMessage], add_messages]  # 对话历史
    user_id: str
    personality_context: Optional[str]                   # 人格画像
    recent_memories: Optional[List[str]]                 # 相关记忆
    intent: Optional[str]                                # chat/diary/content/skill
    intent_confidence: float
    workflow_data: Dict[str, Any]                        # 节点间传递数据
    interrupt_requested: bool                            # 人机协同
    interrupt_reason: Optional[str]
    interrupt_data: Optional[Dict[str, Any]]
    error: Optional[str]
    retry_count: int
    response: Optional[str]                              # 最终回复

# 继承的专用 State
class ContentPipelineState(AgentState)
class DiaryWorkflowState(AgentState)
class InsightWorkflowState(AgentState)
```

**工作流结构 (graph/chat.py)：**

```
intent_classifier → context_builder → memory_retriever 
                                    → user_analyzer
                                    → chat_response
                                       ↓
                            (tools_condition)
                            /            \
                     tools node     extract_user_info
                        ↓              ↓
                  chat_response    require_user_confirmation
                                    ↓
                            save_conversation → END
```

**已有工具：**
- `search_diary_tool()` - 搜索日记
- `search_work_docs_tool()` - 搜索工作文档
- `search_events_tool()` - 搜索交互记录（微信、CLI）
- `search_worldnews_tool()` - 搜索世界新闻

---

### **2.2 数据收集器 (huaqi_src/collectors/)**

**HuaqiDocument - 统一数据模型：**

```python
@dataclass
class HuaqiDocument:
    doc_id: str
    doc_type: str                  # work_doc, diary, event, etc.
    source: str                    # 来源 URL/路径
    content: str
    timestamp: datetime.datetime
    summary: str = ""
    people: list[str] = []         # 相关人员
    metadata: dict[str, Any] = {}
    
    def to_dict(self) -> dict:
        # 序列化为字典
```

**收集器：**
- `InboxProcessor` - 处理 inbox 中的工作文档并归档

---

### **2.3 数据存储系统**

#### **3.2.1 SQLite 本地存储 (db_storage.py)**

```python
class LocalDBStorage:
    def __init__(self, db_path: Optional[str] = None)
    def insert_event(self, event: Event)
    def get_recent_events(self, limit: int = 10) -> List[Event]
    def search_events(self, query: str, limit: int = 5) -> List[Event]
```

**Event 模型 (event.py)：**
```python
@dataclass
class Event:
    timestamp: int
    source: str           # wechat, cli, web, etc.
    actor: str           # 用户ID/来源标识
    content: str         # 自动脱敏 (sk-xxx → sk-***)
    context_id: str = ""
```

#### **3.2.2 向量数据库 - Chroma (chroma_client.py)**

```python
class ChromaClient:
    def __init__(
        self,
        persist_directory: Optional[Path] = None,
        collection_name: str = "memories",
        embedding_function: Optional[Any] = None,
    )
    
    def add(doc_id, content, metadata, embedding) -> bool
    def add_batch(doc_ids, contents, metadatas, embeddings) -> bool
    def search(query, top_k=5, where=None) -> List[Dict]
    def search_by_vector(embedding, top_k=5, where=None) -> List[Dict]
    def get(doc_id) -> Optional[Dict]
    def update(doc_id, content, metadata) -> bool
    def delete(doc_id) -> bool
    def count() -> int
    def export_all() -> List[Dict]
    def clear() -> bool
```

**元数据过滤示例：**
```python
# 按类型过滤
results = chroma_client.search(
    query="关键词",
    where={"type": "diary"}
)
```

#### **3.2.3 Embedding 服务 (embedder.py)**

```python
class EmbeddingService:
    def __init__(
        self,
        model_name: str = "BAAI/bge-small-zh",  # 默认中文轻量模型
        device: str = "auto",                   # cuda/mps/cpu
        normalize_embeddings: bool = True,
        cache_dir: Optional[Path] = None,
    )
    
    def embed(text: str) -> List[float]
    def embed_batch(texts: List[str], batch_size=32) -> List[List[float]]
    def compute_similarity(embedding1, embedding2) -> float
    def rerank(query: str, documents: List[str], top_k) -> List[tuple]
```

#### **3.2.4 混合搜索 (hybrid_search.py)**

```python
class HybridSearch:
    def __init__(
        self,
        chroma_client: Optional[ChromaClient] = None,
        embedder: Optional[EmbeddingService] = None,
        alpha: float = 0.7,   # 向量权重
        beta: float = 0.1,    # 时间衰减系数
        use_bm25: bool = True,
        use_vector: bool = True,
    )
    
    def search(
        query: str,
        top_k: int = 5,
        doc_type: Optional[str] = None,
        use_bm25: Optional[bool] = None,
        use_vector: Optional[bool] = None,
    ) -> List[Dict]:
        # 融合公式：
        # final_score = alpha * vector_score + (1-alpha) * bm25_score + recency_boost
```

**搜索结果格式：**
```python
{
    "id": "doc_id",
    "content": "文档内容",
    "metadata": {...},
    "score": 0.87,           # 最终融合分数
    "vector_score": 0.82,    # 向量相似度
    "bm25_score": 0.89,      # BM25 分数
    "recency_score": 0.95,   # 时间衰减分数
}
```

---

### **2.4 LLM 系统 (core/llm.py)**

**统一 LLM 接口：**

```python
class BaseLLMProvider(ABC):
    def chat(messages: List[Message]) -> LLMResponse
    def chat_stream(messages: List[Message]) -> Iterator[str]
    def count_tokens(text: str) -> int

# 实现的提供商
class OpenAIProvider(BaseLLMProvider)  # 包括兼容 API
class ClaudeProvider(BaseLLMProvider)
class DummyProvider(BaseLLMProvider)    # 测试用

class LLMManager:
    def add_config(config: LLMConfig)
    def set_active(provider_name: str)
    def chat(messages, stream=False) -> Union[LLMResponse, Iterator[str]]
    def quick_chat(prompt: str, system: str) -> str
    def list_providers() -> List[str]
```

**消息和响应：**
```python
@dataclass
class Message:
    role: MessageRole  # system/user/assistant/tool
    content: str
    metadata: Dict[str, Any]
    reasoning_content: Optional[str]  # 支持 DeepSeek thinking

@dataclass
class LLMResponse:
    content: str
    model: str
    usage: Dict[str, int]  # {input_tokens, output_tokens}
    latency_ms: float
    metadata: Dict[str, Any]
    reasoning_content: Optional[str]
    tool_calls: Optional[List[Dict[str, Any]]]
```

---

### **2.5 用户理解系统**

#### **2.5.1 动态维度 Schema (core/schema.py)**

```python
class DimensionSchema:
    dimension_id: str
    dimension_name: str
    dimension_type: DimensionType  # category/score/scale/text/list/boolean/json
    description: str
    allowed_values: Optional[List[str]]        # CATEGORY 用
    validation_rules: Dict[str, Any]
    source: DimensionSource  # system/dynamic/promoted
    is_enabled: bool
    priority: int = 5        # 1-10, 影响 LLM 关注度
    extraction_prompt: str   # LLM 提取提示词

class SchemaRegistry:
    def register(schema: DimensionSchema)
    def unregister(dimension_id: str)
    def get(dimension_id: str) -> Optional[DimensionSchema]
    def list_all() -> List[DimensionSchema]
    def list_enabled() -> List[DimensionSchema]
    def list_by_priority(min_priority: int) -> List[DimensionSchema]
    def add_listener(listener)
    def save_to_file(path: str)
    def load_from_file(path: str)
```

**内置维度（20+个）：**
- 情绪维度：`emotion.primary`, `emotion.intensity`
- 意图维度：`intent.surface`, `intent.deep`
- 话题维度：`topics`, `stances`, `facts`
- 内容偏好：`content.topic_interest`, `content.depth_preference`, 等

```python
class DimensionValue:
    dimension_id: str
    value: Any
    confidence: float = 1.0
    evidence: Optional[List[str]]
    timestamp: str
```

#### **2.5.2 自适应分析引擎**

```python
class AdaptiveUserUnderstanding:
    async def analyze(
        message: str,
        conversation_id: str = "",
        context: Optional[Dict[str, Any]] = None,
    ) -> AnalysisResult
    
    def get_context_for_response(
        current_result: Optional[AnalysisResult] = None,
    ) -> str
```

#### **2.5.3 用户画像模型 (profile_models.py)**

```python
@dataclass
class UserIdentity:
    name: Optional[str]
    nickname: Optional[str]
    birth_date: Optional[str]
    location: Optional[str]
    occupation: Optional[str]
    company: Optional[str]
    email: Optional[str]

@dataclass
class UserPreferences:
    communication_style: Optional[str]
    response_length: Optional[str]
    topics_of_interest: List[str]
    disliked_topics: List[str]
    preferred_languages: List[str]

@dataclass
class UserBackground:
    education: Optional[str]
    skills: List[str]
    hobbies: List[str]
    family_info: Optional[str]
    life_goals: List[str]
    values: List[str]

@dataclass
class UserProfile:
    identity: UserIdentity
    preferences: UserPreferences
    background: UserBackground
    created_at: str
    updated_at: str
```

---

### **2.6 报告系统 (reports/)**

```python
class MorningBriefAgent:
    def __init__(self, data_dir: Optional[Path] = None)
    def _build_context(self) -> str
    def _generate_brief(self) -> str
    def run(self) -> str
```

**输入来源：**
- 世界新闻 (`world/{date}.md`)
- 近期日记 (`memory/diary/*.md`)
- LLM 生成（最多500字，包含世界热点、用户状态观察、鼓励话语）

**输出：**
- 存储到 `reports/daily/{date}-morning.md`

---

### **2.7 世界新闻系统 (world/)**

```python
class WorldNewsStorage:
    def __init__(self, data_dir: Optional[Path] = None)
    def save(docs: list[HuaqiDocument], date: Optional[datetime.date] = None)
    def search(query: str, days: int = 7) -> list[str]
```

**数据格式：**
- 存储为 Markdown：`world/{date}.isoformat()}.md`
- 按 `---` 分隔符分段（每段一条新闻）

---

### **2.8 定时任务系统 (scheduler/)**

```python
class SchedulerManager:
    def __init__(
        self,
        db_path: Optional[Path] = None,
        timezone: str = "Asia/Shanghai",
    )
    
    def add_cron_job(job_id, func, cron)
    def add_interval_job(job_id, func, seconds/minutes/hours)
    def add_date_job(job_id, func, run_date)
    def pause_job(job_id)
    def resume_job(job_id)
    def remove_job(job_id)
    def start()
    def shutdown()
```

**已有任务：**
- `morning_brief` - 每天 8:00 生成晨间简报

---

### **2.9 内容流水线 (pipeline/)**

**数据模型：**

```python
class ContentSource(Enum):
    X = "x"
    RSS = "rss"
    MANUAL = "manual"

class ContentStatus(Enum):
    RAW = "raw"
    SUMMARIZED = "summarized"
    TRANSLATED = "translated"
    READY = "ready"
    PUBLISHED = "published"
    FAILED = "failed"

@dataclass
class ContentItem:
    id: str
    source: ContentSource
    title: str
    content: str
    url: Optional[str]
    author: Optional[str]
    published_at: Optional[datetime]
    status: ContentStatus
    summary: Optional[str]
    translated_content: Optional[str]
    xiaohongshu_content: Optional[str]
    xiaohongshu_tags: List[str]
    metadata: Dict[str, Any]
```

**流水线流程：**

```python
class ContentPipeline:
    async def run(
        self,
        since: Optional[datetime] = None,
        limit: int = 10,
        dry_run: bool = False,
    ) -> Dict[str, Any]
    
    # 流程：采集 → 摘要 → 翻译 → 格式化 → 发布
```

---

### **2.10 配置和路径系统 (core/config_paths.py)**

```python
# 全局配置
def get_data_dir() -> Optional[Path]          # 获取数据目录
def set_data_dir(path: Path) -> Path          # 设置数据目录
def require_data_dir() -> Path                # 必须获取，不存在则报错
def save_data_dir_to_config(data_dir: Path)   # 保存到全局配置

# 专用目录
def get_memory_dir() -> Path                  # ~/.huaqi/memory
def get_drafts_dir() -> Path                  # ~/.huaqi/drafts
def get_vector_db_dir() -> Path               # ~/.huaqi/vector_db
def get_models_cache_dir() -> Path            # ~/.huaqi/models
def get_pending_reviews_dir() -> Path         # ~/.huaqi/pending_reviews
def get_scheduler_db_path() -> Path           # ~/.huaqi/scheduler.db
```

**目录结构：**
```
~/.huaqi/
├── config.yaml            # 全局配置 (data_dir)
├── memory/
│   ├── diary/             # 日记
│   ├── conversations/     # 对话历史
│   ├── work_docs/         # 导入的工作文档
│   └── config.yaml        # 应用配置
├── drafts/                # 草稿
├── vector_db/             # Chroma 向量库
├── models/                # 预下载的 Embedding 模型
├── pending_reviews/       # 待用户审核的内容
├── checkpoints.db         # LangGraph 检查点
├── events.db              # 事件存储
└── scheduler.db           # APScheduler 任务存储
```

---

### **2.11 CLI 系统 (cli/)**

**命令结构：**

```python
# 主应用 (cli/__init__.py)
@app.command("chat")
def chat_command(
    use_langgraph: bool = True,
    session: Optional[str] = None,
    list_sessions: bool = False,
)

@app.command("status")
def show_status()

@app.command("daemon")
def daemon_command(action: str, foreground: bool = False)

@app.command("resume")
def resume_command(task_id: str, response: str = "confirm")
```

**子命令：**
- `config` - 配置管理
- `profile` - 用户画像
- `personality` - 个性化设置
- `pipeline` - 内容流水线
- `system` - 系统管理
- `inbox` - 收件箱管理

---

## **3. 依赖分析 (pyproject.toml)**

**核心依赖：**
```toml
# Agent 框架
langgraph >= 0.2.0
langchain >= 0.3.0
langchain-openai >= 0.2.0

# 向量存储 & 搜索
chromadb >= 0.5.0
sentence-transformers >= 3.0.0
rank-bm25 >= 0.2.0

# 定时任务
apscheduler >= 4.0.0a

# 数据获取
aiohttp >= 3.9.0
feedparser >= 6.0.0

# CLI & UI
typer >= 0.9.0
rich >= 13.0.0
prompt-toolkit >= 3.0.0

# 数据处理
pydantic >= 2.0.0
pydantic-settings >= 2.0.0
pyyaml >= 6.0.0

# 其他
gitpython >= 3.1.0
watchdog >= 4.0.0 (配置热重载)
structlog >= 24.0.0 (日志)
```

---

## **4. 测试结构 (tests/)**

共 **25 个测试文件**，覆盖：

| 模块 | 测试文件 | 覆盖内容 |
|------|---------|---------|
| **agent/** | test_tools.py, test_graph.py, test_chat_nodes.py | 工具、图执行、节点 |
| **core/** | test_config_manager.py, test_db_storage.py, test_event.py | 配置、存储、事件 |
| **collectors/** | test_document.py, test_inbox_processor.py | 文档模型、处理器 |
| **cli/** | test_inbox_cli.py | CLI 命令 |
| **world/** | test_fetcher.py, test_storage.py | 新闻获取、存储 |
| **reports/** | test_morning_brief.py | 晨间简报 |
| **scheduler/** | test_jobs.py | 定时任务 |
| **unit/** | test_llm.py, test_llm_connection.py, test_basic_input.py | LLM、输入处理 |

---

## **5. 代码规范和约定**

### **5.1 设计模式**

1. **单例模式：**
   ```python
   _instance: Optional[ClassName] = None
   
   def get_instance() -> ClassName:
       global _instance
       if _instance is None:
           _instance = ClassName()
       return _instance
   ```

2. **工厂模式：**
   - `LLMManager` - 创建不同提供商的 LLM
   - `SchedulerManager` - 创建不同类型的任务

3. **观察者模式：**
   - `SchemaRegistry.add_listener()` - Schema 变更通知

4. **命令模式：**
   - 所有 CLI 命令都是独立的子命令

### **5.2 命名约定**

- **文件名**：全小写 + 下划线 (`config_manager.py`, `llm.py`)
- **类名**：PascalCase (`ConfigManager`, `LLMProvider`)
- **函数名**：snake_case (`get_data_dir`, `search_diary`)
- **常量**：大写 + 下划线 (`INTENT_CHAT`, `SUPPORTED_EXTENSIONS`)
- **私有方法/属性**：前缀 `_` (`_build_index`, `_instance`)

### **5.3 类型注解**

全项目使用 **类型注解**：
```python
from typing import List, Optional, Dict, Any, Union, Callable
from pathlib import Path
from dataclasses import dataclass

def search(query: str, top_k: int = 5) -> List[Dict[str, Any]]:
    pass
```

### **5.4 异常处理**

自定义异常：
```python
class LLMError(Exception):
    pass
```

---

## **6. 数据流示例**

### **6.1 对话流程**

```
User Input
    ↓
Agent Graph (LangGraph)
    ├─ intent_classifier
    ├─ context_builder
    │   ├─ 获取 personality_context
    │   ├─ 获取 profile_manager.get_system_prompt_addition()
    │   └─ 获取 adaptive_understanding.get_context_for_response()
    ├─ memory_retriever
    │   └─ HybridSearch.search() → [相关记忆]
    ├─ user_analyzer
    │   └─ AdaptiveUserUnderstanding.analyze()
    ├─ chat_response (LLM 调用)
    │   └─ 可能触发 tool_calls
    │       ├─ search_diary_tool
    │       ├─ search_work_docs_tool
    │       ├─ search_events_tool
    │       └─ search_worldnews_tool
    ├─ extract_user_info (更新用户理解)
    └─ save_conversation → END

Response Output
```

### **6.2 搜索流程**

```
Query: "我最近在干什么"
    ↓
HybridSearch.search()
    ├─ Vector Search (Chroma)
    │   └─ 使用 embedder 编码查询
    │   └─ 返回向量相似度 top K
    ├─ BM25 Search
    │   └─ 分词后 BM25 计分
    │   └─ 返回关键词匹配 top K
    └─ Fusion
        ├─ 融合两个排序
        ├─ 加入时间衰减
        └─ 返回最终 top K

Results
```

---

## **7. 关键文件路径速查**

| 功能 | 文件路径 |
|------|---------|
| **Agent 入口** | `/huaqi_src/agent/graph/chat.py` |
| **工具定义** | `/huaqi_src/agent/tools.py` |
| **Agent State** | `/huaqi_src/agent/state.py` |
| **LLM 接口** | `/huaqi_src/core/llm.py` |
| **向量 DB** | `/huaqi_src/memory/vector/chroma_client.py` |
| **搜索引擎** | `/huaqi_src/memory/vector/hybrid_search.py` |
| **Embedding** | `/huaqi_src/memory/vector/embedder.py` |
| **Schema 系统** | `/huaqi_src/core/schema.py` |
| **自适应理解** | `/huaqi_src/core/adaptive_understanding.py` |
| **用户画像** | `/huaqi_src/core/profile_models.py` |
| **配置路径** | `/huaqi_src/core/config_paths.py` |
| **数据收集** | `/huaqi_src/collectors/document.py` |
| **SQLite 存储** | `/huaqi_src/core/db_storage.py` |
| **定时任务** | `/huaqi_src/scheduler/manager.py` |
| **内容流水线** | `/huaqi_src/pipeline/core.py` |
| **晨间简报** | `/huaqi_src/reports/morning_brief.py` |
| **CLI 主入口** | `/huaqi_src/cli/__init__.py` |
| **测试配置** | `/tests/conftest.py` |

---

## **8. 启动和配置流程**

### **8.1 首次启动流程**

```
huaqi 命令
    ↓
cli/__init__.py: main()
    ├─ 检查 is_data_dir_set()
    ├─ 如未设置，运行 _wizard_set_data_dir()
    │   └─ 用户输入数据目录
    │   └─ save_data_dir_to_config()
    ├─ ensure_dirs() 创建所有目录
    ├─ 注册 SchedulerManager
    └─ run_langgraph_chat()
```

### **8.2 LLM 配置流程**

```
huaqi config set llm
    ↓
选择提供商 (openai/claude/dummy)
    ↓
输入 API Key
    ↓
输入模型名称
    ↓
LLMManager.add_config()
    ↓
LLMManager.set_active()
```

---

## **9. 扩展点和集成点**

### **9.1 添加新工具**

```python
# 在 huaqi_src/agent/tools.py 中：

from langchain_core.tools import tool

@tool
def new_tool(query: str) -> str:
    """工具描述"""
    # 实现
    return result

# 在 chat.py 中注册：
tools = [..., new_tool]
tool_node = ToolNode(tools)
```

### **9.2 添加新维度**

```python
# 在 core/schema.py 的 _register_builtin_schemas() 中：

registry.register(DimensionSchema(
    dimension_id="custom.my_dimension",
    dimension_name="我的维度",
    dimension_type=DimensionType.CATEGORY,
    description="描述",
    allowed_values=[...],
    extraction_prompt="提示词",
    priority=8,
))
```

### **9.3 添加新的 LLM 提供商**

```python
# 在 core/llm.py 中：

class MyProvider(BaseLLMProvider):
    name = "my_provider"
    
    def _validate_config(self):
        pass
    
    def chat(self, messages, **kwargs) -> LLMResponse:
        pass
    
    def chat_stream(self, messages, **kwargs) -> Iterator[str]:
        pass

# 注册：
LLMManager._providers["my_provider"] = MyProvider
```

### **9.4 添加新的定时任务**

```python
# 在 scheduler/jobs.py 中：

def _run_my_job():
    # 任务逻辑
    pass

# 在 register_default_jobs() 中：
manager.add_cron_job(
    job_id="my_job",
    func=_run_my_job,
    cron="0 9 * * *",  # 每天 9:00
)
```

---

## **10. 已知约束和注意事项**

1. **配置必填：**
   - LLM 配置（API Key）必须设置
   - 数据目录必须设置

2. **模型加载：**
   - Embedding 模型（BGE-small-zh）会在首次使用时自动下载和缓存
   - 需要网络访问 Hugging Face

3. **Chroma 特性：**
   - 使用余弦相似度（cosine space）
   - 支持元数据过滤

4. **LangGraph 状态：**
   - 所有节点共享 `AgentState`
   - 消息列表使用 `add_messages` reducer 自动追加

5. **单例管理：**
   - 所有 manager 都是单例模式
   - 第一次调用时初始化

---

这份总结应该足以支持实施计划的编写。所有关键数据模型、接口和扩展点都已清晰列出。
