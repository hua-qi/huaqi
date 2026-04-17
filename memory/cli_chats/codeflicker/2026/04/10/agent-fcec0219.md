---
session_id: agent-fcec0219
date: 2026-04-10
time_start: 2026-04-10T03:40:25.438Z
time_end: 2026-04-10T03:44:06.157Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-04-10 11:40:25] [user]: 请深入探索 /Users/lianzimeng/workspace/huaqi-growing 代码库，重点关注以下内容：

1. 找到所有 Telos 相关文件（TelosEngine, TelosManager, TelosContextBuilder, DistillationPipeline 等）
2. 找到 chat_nodes.py 和 build_context 节点
3. 找到 scheduler 相关代码（定时任务）
4. 找到 PeoplePipeline 相关代码
5. 找到 RawSignal 相关代码
6. 找到 Person 模型定义
7. 找到测试文件结构
8. 找到 package.json 或 pyproject.toml 等项目配置文件，了解测试命令
9. 找到 UserProfile 相关代码

对每个找到的文件，请读取其内容，特别关注：
- 类定义和方法签名
- 文件路径
- 现有的 Step1/Step3/Step4/Step5 实现
- confidence 字段的使用
- 如何触发提炼流程

请返回详细的文件路径、关键类/函数名、以及对现有实现的理解总结。
[2026-04-10 11:40:28] [assistant]: 我将深入探索这个代码库，系统性地找到所有关键文件并分析其内容。让我开始进行全面的搜索。
[2026-04-10 11:40:32] [assistant]: 现在让我并行搜索关键文件：
[2026-04-10 11:40:36] [assistant]: 很好！现在让我继续搜索其他关键文件：
[2026-04-10 11:40:40] [assistant]: 现在让我搜索 Telos 核心文件和相关模型：
[2026-04-10 11:40:45] [assistant]: 很好！现在让我搜索 PeoplePipeline 和相关文件：
[2026-04-10 11:40:50] [assistant]: 现在让我搜索 RawSignal 和 Person 相关内容：
[2026-04-10 11:40:54] [assistant]: 现在让我开始读取核心文件。首先读取 Telos 相关文件：
[2026-04-10 11:40:59] [assistant]: 继续读取更多核心文件：
[2026-04-10 11:41:03] [assistant]: 现在读取 chat_nodes.py 和 raw_signal pipeline：
[2026-04-10 11:42:10] [assistant]: 现在读取 scheduler 和其他重要文件：
[2026-04-10 11:42:15] [assistant]: 现在让我探索 People 相关代码和其他文件：
[2026-04-10 11:42:20] [assistant]: 现在让我搜索 PeoplePipeline 和检查测试文件结构：
[2026-04-10 11:42:25] [assistant]: 现在让我读取 UserProfile 和其他重要文件：
[2026-04-10 11:42:30] [assistant]: 现在让我读取更多测试文件和设计文档：
[2026-04-10 11:42:34] [assistant]: 现在让我检查 meta.py 和其他相关文件：
[2026-04-10 11:42:39] [assistant]: 现在让我查看 raw_signal store 和其他相关文件：
[2026-04-10 11:42:44] [assistant]: 现在让我收集关键的路径信息和创建最终总结：
[2026-04-10 11:42:48] [assistant]: 完美！现在让我查看一些测试配置信息：
[2026-04-10 11:42:53] [assistant]: 现在让我创建一个完整的总结文档。让我收集一些其他关键信息：
[2026-04-10 11:44:06] [assistant]: 完美！现在我有足够的信息来生成详细的总结。让我创建最终的探索报告：

## 深入探索报告：huaqi-growing 代码库

我已完成对 `/Users/lianzimeng/workspace/huaqi-growing` 代码库的全面探索。以下是详细的发现总结。

---

## 1. Telos 系统核心文件

### 1.1 文件路径与基本信息

| 组件 | 文件路径 | 关键类/函数 | 功能描述 |
|------|--------|-----------|--------|
| **Models** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/models.py` | `TelosDimension`, `HistoryEntry`, `DimensionLayer` | 定义维度数据结构，包含9个标准维度（beliefs, models, narratives, goals, challenges, strategies, learned, people, shadows） |
| **Engine** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/engine.py` | `TelosEngine`, `Step1Output`, `Step3Output`, `Step4Output`, `Step5Output` | 实现5步提炼流水线（Step1分析信号→Step3决定更新→Step4生成内容→Step5评估成长事件） |
| **Manager** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/manager.py` | `TelosManager` | 维度文件持久化、INDEX重建、git自动提交 |
| **Context** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/context.py` | `TelosContextBuilder`, `SystemPromptBuilder` | 将Telos数据注入Agent上下文 |
| **Meta** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/meta.py` | `MetaManager`, `CorrectionRecord`, `DimensionOperation` | 跟踪用户反馈纠错、维度演化历史 |
| **GrowthEvents** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/growth_events.py` | `GrowthEvent`, `GrowthEventStore` | 记录用户真正的成长突破事件（层级、触发信号、新内容） |
| **Init** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/__init__.py` | 导出所有核心类 | 供外部导入 |

### 1.2 维度定义

**标准9个维度（按层级）：**

| 维度 | 层级 | 说明 |
|-----|------|------|
| `beliefs` | core | 信念和价值观 |
| `models` | core | 世界观和心智模型 |
| `narratives` | core | 关于自己的叙事和自我认知 |
| `goals` | middle | 短/中/长期目标 |
| `challenges` | middle | 正在面对的困难和卡点 |
| `strategies` | middle | 应对问题的方式和习惯 |
| `learned` | surface | 学到的知识和洞察 |
| `people` | surface | **【注：文档显示应独立处理】** |
| `shadows` | surface | 阴影自我与未表达的部分 |

---

## 2. RawSignal 与 DistillationPipeline

### 2.1 RawSignal 数据模型

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/data/raw_signal/models.py`

**核心类：**
```python
class RawSignal(BaseModel):
    id: str                    # UUID
    user_id: str              # 用户ID
    source_type: SourceType   # journal|ai_chat|wechat|reading|audio|video|image|calendar|absence
    timestamp: datetime       # 信号时间
    ingested_at: datetime     # 摄入时间
    content: str              # 内容
    metadata: Optional[Any]   # 源类型特定元数据
    raw_file_ref: Optional[str] # 原始文件引用
    processed: bool = False   # 是否已过Step1分析
    distilled: bool = False   # 是否已过Step3-5提炼
    vectorized: bool = False  # 是否已向量化
```

**Source Types：** journal, ai_chat, wechat, reading, audio, video, image, calendar, absence

### 2.2 DistillationPipeline

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/data/raw_signal/pipeline.py`

**核心类：** `DistillationPipeline`

**关键方法：**
```python
def process(self, signal: RawSignal) -> Dict[str, Any]:
    # 1. 调用 engine.step1_analyze(signal) → Step1Output
    # 2. 标记信号为 processed=1
    # 3. 对每个涉及维度（signal_strength + threshold判断）：
    #    - 查询近30天该维度的相关信号
    #    - 调用 engine.run_pipeline() 执行Step3-5
    #    - 如生成GrowthEvent，存入event_store
    # 返回处理结果摘要
```

**重要参数：**
- `signal_threshold: int = 3` - 触发提炼所需的最少信号数
- `days_window: int = 30` - 统计窗口（天数）

**触发逻辑：**
- **STRONG信号**：立即进入提炼流程（不考虑阈值）
- **非STRONG信号**：只有当近30天内该维度信号数 ≥ threshold时才触发

---

## 3. Chat Nodes 与上下文构建

### 3.1 Chat Nodes

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/agent/nodes/chat_nodes.py`

**核心节点函数：**

| 函数 | 功能 |
|-----|------|
| `classify_intent()` | 意图识别：diary、skill、content、chat、unknown |
| **`build_context()`** | **组装系统提示词（关键节点）** |
| `extract_user_info()` | 从消息提取用户信息更新UserProfile |
| `retrieve_memories()` | 混合检索：Chroma向量库 + 当日Markdown扫描 |
| `generate_response()` | 异步调用LLM生成回复 |
| `save_conversation()` | 保存对话到Markdown + Chroma索引 |

**build_context节点（关键）：**
```python
def build_context(state: AgentState) -> Dict[str, Any]:
    # 1. 获取 personality_context（来自个性引擎）
    # 2. 获取 user_profile_context（来自UserProfileManager）
    # 3. 调用 build_system_prompt() 组装
    # 返回 workflow_data["system_prompt"]
    
    # 【当前问题】TelosContextBuilder 未接入，缺少 telos_snapshot 注入
```

### 3.2 TelosContextBuilder（已实现但未接入）

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/context.py`

**核心类：**
```python
class TelosContextBuilder:
    def build_telos_snapshot(self) -> str:
        # 读取 INDEX.md 或生成维度列表
        # 返回格式示例：
        # # 当前 TELOS 快照
        # - beliefs（core层，置信度 0.8）：...前30字
        # - goals（middle层，置信度 0.75）：...前30字
    
    def build_relevant_history(self, query: str, top_k: int = 5) -> List[str]:
        # 向量搜索相关历史记忆
    
    def inject(self, state: AgentState, query: Optional[str] = None) -> AgentState:
        # 将 telos_snapshot 和 relevant_history 注入state
        # 返回修改后的state

class SystemPromptBuilder:
    def build(
        self,
        telos_snapshot: str,
        relevant_history: List[str],
        interaction_mode: str,  # chat|onboarding|report|distill
    ) -> str:
        # 组装完整的system prompt
```

---

## 4. RawSignal Store 与存储适配器

### 4.1 RawSignal Store

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/data/raw_signal/store.py`

**核心类：** `RawSignalStore`

**关键方法：**
```python
def save(self, signal: RawSignal) -> None
def get(self, signal_id: str) -> Optional[RawSignal]
def query(self, filter: RawSignalFilter) -> List[RawSignal]  # 查询符合条件的信号
def mark_processed(self, signal_id: str) -> None  # 标记为已分析
def mark_distilled(self, signal_id: str) -> None  # 标记为已提炼
def mark_vectorized(self, signal_id: str) -> None  # 标记为已向量化
def count(self, filter: RawSignalFilter) -> int  # 统计符合条件的信号数
```

### 4.2 RawSignalFilter

```python
class RawSignalFilter(BaseModel):
    user_id: str
    source_type: Optional[SourceType] = None
    processed: Optional[int] = None  # 0=未处理, 1=已处理
    distilled: Optional[int] = None
    vectorized: Optional[int] = None
    since: Optional[datetime] = None   # 起始时间
    until: Optional[datetime] = None   # 结束时间
    limit: int = 100
    offset: int = 0
```

---

## 5. Scheduler 与定时任务

### 5.1 Scheduler 相关文件

| 文件路径 | 类/函数 | 功能 |
|---------|--------|------|
| `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/scheduler/apscheduler_adapter.py` | `APSchedulerAdapter` | APScheduler包装器 |
| `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/scheduler/pipeline_job.py` | `PipelineJobManager`, `scheduled_content_pipeline()` | 内容流水线定时任务 |
| `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/cli/commands/scheduler.py` | CLI命令 | 定时任务管理命令 |

### 5.2 APScheduler Adapter

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/scheduler/apscheduler_adapter.py`

```python
class APSchedulerAdapter:
    def __init__(self)
    def start(self) -> None                     # 启动调度器
    def stop(self) -> None                      # 停止调度器
    def add_interval_job(self, func, seconds: int, job_id: str) -> None  # 添加定时任务
```

### 5.3 Pipeline Job Manager

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/scheduler/pipeline_job.py`

**关键方法：**
```python
class PipelineJobManager:
    async def run_pipeline(
        self,
        limit: int = 5,
        require_review: bool = True,  # 是否需要人工审核
    ) -> Dict[str, Any]
    
    def list_pending_reviews(self) -> List[Dict[str, Any]]  # 列出待审核任务
    def get_pending_task(self, task_id: str) -> Optional[Dict[str, Any]]
    def approve_item(self, task_id: str, item_index: int, notes: str = None) -> bool
    def reject_item(self, task_id: str, item_index: int, notes: str = None) -> bool
    async def publish_approved(self, task_id: str) -> int  # 发布已通过的内容

async def scheduled_content_pipeline(**kwargs)  # 定时执行的入口函数
```

---

## 6. Person 模型与 People Pipeline

### 6.1 Person 模型

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/dimensions/people/models.py`

```python
@dataclass
class Person:
    person_id: str
    name: str
    relation_type: str  # 家人|朋友|同事|导师|合作者|其他
    alias: list[str] = []           # 别名
    profile: str = ""               # 性格/职业/兴趣描述
    emotional_impact: str = "中性"  # 积极|中性|消极
    interaction_frequency: int = 0  # 互动频度
    notes: str = ""
    created_at: str
    updated_at: str

@dataclass
class Relation:
    from_person_id: str
    to_person_id: str
    relation_strength: int = 50  # 0-100
    topics: list[str] = []
    history_summary: str = ""
```

### 6.2 Person Extractor

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/dimensions/people/extractor.py`

```python
class PersonExtractor:
    def extract_from_text(self, text: str) -> list[Person]:
        # 调用LLM提取人物信息
        # 输出字段：name, relation_type, profile, emotional_impact, alias
        # 更新或创建Person对象，存入PeopleGraph
```

### 6.3 目录结构

```
/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/growth/telos/dimensions/people/
├── __init__.py
├── models.py           # Person, Relation
├── extractor.py        # PersonExtractor
└── graph.py           # PeopleGraph（管理Person集合）
```

**【注意】** 根据最新设计文档，`people` 将从标准9维度中移除，建立专属处理管道。

---

## 7. UserProfile 模型与管理器

### 7.1 UserProfile 模型

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/data/profile/models.py`

```python
@dataclass
class UserIdentity:
    name: Optional[str]           # 名字
    nickname: Optional[str]       # 昵称
    birth_date: Optional[str]     # 生日
    location: Optional[str]       # 所在地
    occupation: Optional[str]     # 职业
    company: Optional[str]        # 公司
    email: Optional[str]

@dataclass
class UserPreferences:
    communication_style: Optional[str]    # 沟通风格
    response_length: Optional[str]        # 回复长度偏好
    topics_of_interest: List[str]        # 感兴趣的话题
    disliked_topics: List[str]           # 不喜欢的话题
    preferred_languages: List[str] = ["zh"]

@dataclass
class UserBackground:
    education: Optional[str]      # 教育背景
    skills: List[str]             # 技能
    hobbies: List[str]            # 爱好
    family_info: Optional[str]    # 家庭信息
    life_goals: List[str]         # 人生目标
    values: List[str]             # 价值观

@dataclass
class UserProfile:
    identity: UserIdentity
    preferences: UserPreferences
    background: UserBackground
    created_at: str
    updated_at: str
    version: int
    extraction_history: List[Dict[str, Any]]
```

### 7.2 UserProfileManager

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/layers/data/profile/manager.py`

**核心类：** `UserProfileManager`

**关键方法：**
```python
@property
def profile(self) -> UserProfile  # 加载或创建profile

def save(self) -> None  # 保存为YAML文件

def update_identity(self, **kwargs) -> None  # 更新身份信息
def update_preferences(self, **kwargs) -> None  # 更新偏好
def update_background(self, **kwargs) -> None  # 更新背景

def get_system_prompt_addition(self) -> str  # 生成供System Prompt用的摘要

def extract_from_message(self, user_message: str, llm_manager=None) -> Dict[str, Any]
    # LLM智能提取用户信息

def _apply_extraction(self, extracted: Dict[str, Any], source_message: str)
    # 应用提取结果到画像
```

**存储位置：** `{memory_dir}/user_profile.yaml`

---

## 8. 现有实现的 Confidence 字段使用

### 8.1 在各数据结构中的用法

| 组件 | 字段位置 | 数据类型 | 来源 | 说明 |
|------|---------|--------|------|------|
| **TelosDimension** | `confidence: float` | 0.0-1.0 | engine.step3_decide() | 维度更新的置信度，影响该维度的权重 |
| **HistoryEntry** | `confidence: float` | 0.0-1.0 | engine.step3_decide() | 历史记录中保存该版本的置信度 |
| **GrowthEvent** | 无专属confidence | - | - | 通过所在维度的confidence间接表示 |

### 8.2 置信度的当前问题（根据设计文档）

1. **LLM直接输出**：Step3中LLM直接给出confidence数字，无明确依据
2. **用户纠错**：固定扣0.15，不区分纠错力度
3. **时间衰减缺失**：无机制处理长期无新信号导致的内容过时

### 8.3 新设计方案

```python
# 置信度由两部分组成：
confidence = count_score * 0.4 + consistency_score * 0.6

count_score = min(recent_signal_count / 10, 1.0)
    # recent_signal_count = 近30天该维度的信号数量
    
consistency_score = 0.0-1.0
    # LLM在合并Step3+4+5时判断信号方向一致性
```

---

## 9. 五步提炼流程详解

### 9.1 Step1：信号分析

**文件：** engine.py 中的 `step1_analyze()` 方法

**输入：** RawSignal

**LLM Prompt关键部分：**
```
- 判断信号涉及哪些维度
- 情感：positive|negative|neutral
- 强度：0.0-1.0
- 信号强度：strong|medium|weak
- 如不属于现有维度，给出 new_dimension_hint
```

**输出类型：** `Step1Output`
```python
class Step1Output(BaseModel):
    dimensions: List[str]              # 涉及的维度
    emotion: str                        # positive|negative|neutral
    intensity: float                    # 0.0-1.0
    signal_strength: SignalStrength     # strong|medium|weak
    strong_reason: Optional[str]        # 为什么是strong
    summary: str                        # 信号摘要
    new_dimension_hint: Optional[str]   # 可能的新维度
```

### 9.2 Step3：更新决策

**方法：** `step3_decide(dimension, signal_summaries, days)`

**输入：**
- dimension：要评估的维度名
- signal_summaries：近期相关信号摘要列表
- days：时间窗口

**LLM判断：**
- 是否应该更新该维度
- 更新类型：reinforce|challenge|new
- 置信度
- 建议的新内容

**输出类型：** `Step3Output`
```python
class Step3Output(BaseModel):
    should_update: bool
    update_type: Optional[UpdateType]   # reinforce|challenge|new
    confidence: float
    reason: str
    suggested_content: Optional[str]
```

### 9.3 Step4：内容生成

**方法：** `step4_generate(dimension, step3_result, signal_summaries)`

**功能：**
1. 调用LLM生成新内容
2. 创建HistoryEntry记录
3. 更新TelosDimension
4. 触发INDEX.md重建

**输出类型：** `Step4Output`
```python
class Step4Output(BaseModel):
    new_content: str
    history_entry: Dict[str, str]  # {"change": "...", "trigger": "..."}
```

### 9.4 Step5：成长事件判断

**方法：** `step5_judge_growth(dimension, layer, step4_result)`

**判断标准：**
- 核心层维度变化 → 几乎总是值得
- 中间层维度的方向性转变 → 值得
- 表面层的日常积累 → 通常不值得

**输出类型：** `Step5Output`
```python
class Step5Output(BaseModel):
    is_growth_event: bool
    narrative: Optional[str]   # 温暖的成长叙事
    title: Optional[str]       # 事件标题
```

### 9.5 完整流程编排

**方法：** `run_pipeline(signal, step1_result, signal_summaries, days)`

```python
def run_pipeline(...) -> Dict[str, Any]:
    results = {"updated": False, "growth_event": None}
    
    for dimension in step1_result.dimensions:
        layer = STANDARD_DIMENSION_LAYERS.get(dimension)
        
        # 执行Step3-5
        step3 = self.step3_decide(dimension, signal_summaries, days)
        if not step3.should_update:
            continue
        
        step4 = self.step4_generate(dimension, step3, signal_summaries)
        results["updated"] = True
        
        step5 = self.step5_judge_growth(dimension, layer, step4)
        if step5.is_growth_event:
            results["growth_event"] = step5
    
    return results
```

---

## 10. 测试文件结构

### 10.1 Telos 测试文件

**目录：** `/Users/lianzimeng/workspace/huaqi-growing/tests/unit/layers/growth/`

| 测试文件 | 测试类 | 测试数量 |
|---------|--------|--------|
| `test_telos_models.py` | `TestDimensionLayer`, `TestHistoryEntry`, `TestTelosDimension` | ~15 |
| `test_telos_engine.py` | `TestStep1Output`, `TestStep3Output`, `TestStep4Output`, `TestStep5Output`, `TestTelosEngineStep1`, `TestTelosEngineStep3`, `TestTelosEngineStep4`, `TestTelosEngineStep5`, `TestTelosEngineFullPipeline` | ~20+ |
| `test_telos_manager.py` | `TestTelosManagerInit`, `TestTelosManagerGet`, `TestTelosManagerListActive`, `TestTelosManagerUpdate`, `TestTelosManagerArchive`, `TestTelosManagerRebuildIndex` | ~15 |
| `test_meta.py` | 纠错、活跃维度、维度演化 | ~10 |
| `test_growth_events.py` | 事件保存、查询 | ~5 |

### 10.2 RawSignal 测试文件

**目录：** `/Users/lianzimeng/workspace/huaqi-growing/tests/unit/layers/data/`

| 测试文件 | 焦点 |
|---------|-----|
| `test_raw_signal_models.py` | 信号模型验证 |
| `test_raw_signal_store.py` | Store CRUD操作 |
| `test_raw_signal_pipeline.py` | DistillationPipeline流程 |

### 10.3 集成测试

**文件：** `/Users/lianzimeng/workspace/huaqi-growing/tests/integration/test_raw_signal_to_telos.py`

**场景：** 用户连续3天写日记，全部涉及"方向感缺失"
- Day 1：信号写入 → Step1识别 → 累积不足，无更新
- Day 2：同上，累积2条
- Day 3：累积3条，强度足够 → Step3-5完整流程 → 成长事件生成

---

## 11. 项目配置文件

### 11.1 pyproject.toml

**文件路径：** `/Users/lianzimeng/workspace/huaqi-growing/pyproject.toml`

**关键配置：**

```toml
[project]
name = "huaqi"
version = "0.1.0"
requires-python = ">=3.9"

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "pytest-cov>=4.0.0",
    "black>=23.0.0",
    "isort>=5.0.0",
    "mypy>=1.0.0",
    "ruff>=0.1.0",
]

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = "test_*.py"
python_classes = "Test*"
python_functions = "test_*"
asyncio_mode = "auto"
```

**核心依赖：**
- `langgraph>=0.2.0` - Agent架构
- `chromadb>=0.5.0` - 向量存储
- `sentence-transformers>=3.0.0` - 向量化
- `apscheduler>=4.0.0a` - 定时任务
- `pydantic>=2.0.0` - 数据验证

### 11.2 运行测试命令

```bash
# 运行所有测试
pytest tests/

# 运行特定层的测试
pytest tests/unit/layers/growth/
pytest tests/unit/layers/data/
pytest tests/unit/agent/

# 运行集成测试
pytest tests/integration/

# 带覆盖率
pytest --cov=huaqi_src tests/
```

---

## 12. 关键代码路径总结

### 12.1 完整数据流

```
RawSignal (journal/chat/wechat...)
    ↓
RawSignalStore.save()  [stored in SQLite]
    ↓
DistillationPipeline.process()
    ├─ Step1: TelosEngine.step1_analyze()
    │  └─ Extract dimensions, emotion, intensity
    ├─ Mark processed=1
    └─ For each dimension (if strong or count >= threshold):
        ├─ Step3: TelosEngine.step3_decide()
        ├─ Step4: TelosEngine.step4_generate()
        │  └─ Update TelosDimension + HistoryEntry
        ├─ Step5: TelosEngine.step5_judge_growth()
        └─ [If growth event] → GrowthEventStore.save()
```

### 12.2 Agent对话流程（当前状态）

```
User Message
    ↓
chat_nodes.classify_intent()
    ↓
chat_nodes.build_context()  ← build_system_prompt()
    ├─ personality_context
    ├─ user_profile_context
    └─ ❌ MISSING: telos_snapshot (未接入)
    ↓
chat_nodes.retrieve_memories()
    ├─ Chroma向量检索
    └─ 当日Markdown扫描
    ↓
chat_nodes.generate_response()
    ├─ LLM inference with full_messages
    └─ Return response
    ↓
chat_nodes.save_conversation()
    ├─ MarkdownMemoryStore
    └─ _index_to_chroma()
```

### 12.3 Agent对话流程（应该是）

```
User Message
    ↓
chat_nodes.classify_intent()
    ↓
chat_nodes.build_context()  ← build_system_prompt()
    ├─ personality_context
    ├─ user_profile_context
    ├─ telos_snapshot (8个维度的当前认知)
    └─ relevant_history (向量检索的历史)
    ↓
chat_nodes.retrieve_memories()
    ├─ Chroma向量检索
    └─ 当日Markdown扫描
    ↓
chat_nodes.generate_response()
    ├─ LLM inference with full_messages
    └─ Return response
    ↓
chat_nodes.save_conversation()
    ├─ MarkdownMemoryStore
    └─ _index_to_chroma()
```

---

## 13. 设计文档中的优化方向

根据 `/Users/lianzimeng/workspace/huaqi-growing/docs/designs/2026-10-04-telos-design-and-implementation.md`：

### 13.1 流水线合并优化

**当前状态：**
- Step1（1次LLM） + Step3（1次）+ Step4（1次）+ Step5（1次） = 4次串行调用

**优化方案：**
- 将Step3+4+5合并为单次LLM调用，返回结构化JSON
- 支持多维度并行执行
- **最坏情况从4次 → 2次LLM时间**（Step1串行 + 其余并行）

### 13.2 置信度重构

**新公式：**
```
confidence = count_score * 0.4 + consistency_score * 0.6

count_score = min(recent_signal_count / 10, 1.0)
consistency_score = LLM_judged_value (0.0-1.0)
```

**用户纠错：** 由LLM判断纠错力度，调整consistency_score而非固定扣值

### 13.3 定期复审机制

- 检查超过N天无新信号的维度
- 触发轻量LLM复审
- 自动调整consistency_score

### 13.4 Agent上下文注入

- 各维度读取 `frontmatter + ## 当前认知`（跳过更新历史）
- 每维度5-10行，8个共50-80行，内容完整长度可控
- 不使用INDEX.md，改用单个维度文件

### 13.5 People维度独立处理

- 从标准9维度移除
- 建立专属PeoplePipeline
- 在Step1分叉后并行处理

### 13.6 冷启动问卷

- 5个问题覆盖所有维度
- 一次LLM批量提取
- 直接写入（跳过提炼管道）
- 初始置信度0.4（标记为"问卷初始化，待验证"）

---

## 14. 文件树总览

```
/Users/lianzimeng/workspace/huaqi-growing/
├── huaqi_src/
│   ├── layers/
│   │   ├── growth/
│   │   │   └── telos/
│   │   │       ├── __init__.py
│   │   │       ├── models.py              # TelosDimension
│   │   │       ├── engine.py              # TelosEngine (Step1-5)
│   │   │       ├── manager.py             # TelosManager
│   │   │       ├── context.py             # TelosContextBuilder
│   │   │       ├── meta.py                # MetaManager
│   │   │       ├── growth_events.py       # GrowthEvent/Store
│   │   │       └── dimensions/
│   │   │           └── people/
│   │   │               ├── models.py      # Person, Relation
│   │   │               ├── extractor.py   # PersonExtractor
│   │   │               └── graph.py       # PeopleGraph
│   │   ├── data/
│   │   │   ├── raw_signal/
│   │   │   │   ├── models.py              # RawSignal
│   │   │   │   ├── store.py               # RawSignalStore
│   │   │   │   └── pipeline.py            # DistillationPipeline
│   │   │   ├── profile/
│   │   │   │   ├── models.py              # UserProfile
│   │   │   │   └── manager.py             # UserProfileManager
│   │   │   └── memory/
│   │   ├── capabilities/
│   │   └── ...
│   ├── agent/
│   │   ├── nodes/
│   │   │   └── chat_nodes.py              # 对话节点（build_context）
│   │   ├── state.py
│   │   └── ...
│   ├── scheduler/
│   │   ├── apscheduler_adapter.py        # 调度器适配器
│   │   └── pipeline_job.py               # 内容流水线定时任务
│   ├── cli/
│   │   ├── commands/
│   │   │   ├── scheduler.py
│   │   │   └── pipeline.py
│   │   └── context.py
│   └── config/
├── tests/
│   ├── unit/
│   │   ├── layers/
│   │   │   ├── growth/
│   │   │   │   ├── test_telos_models.py
│   │   │   │   ├── test_telos_engine.py
│   │   │   │   ├── test_telos_manager.py
│   │   │   │   └── test_meta.py
│   │   │   ├── data/
│   │   │   │   ├── test_raw_signal_models.py
│   │   │   │   ├── test_raw_signal_store.py
│   │   │   │   └── test_raw_signal_pipeline.py
│   │   │   └── agent/
│   │   │       └── test_chat_nodes.py
│   │   └── config/
│   │       ├── test_scheduler_adapter.py
│   │       ├── test_apscheduler.py
│   │       └── test_scheduler_job_config.py
│   └── integration/
│       ├── test_raw_signal_to_telos.py    # 完整数据流
│       ├── test_telos_to_agent.py
│       └── test_cold_start.py
├── pyproject.toml                          # 项目配置 + 依赖 + 测试选项
├── docs/
│   ├── designs/
│   │   └── 2026-10-04-telos-design-and-implementation.md  # 最新设计
│   └── ...
└── README.md
```

---

## 15. 关键类/函数速查表

| 用途 | 文件 | 类/函数 | 关键方法 |
|------|------|--------|--------|
| **维度数据** | models.py | `TelosDimension` | `to_markdown()`, `from_markdown()` |
| **维度管理** | manager.py | `TelosManager` | `init()`, `get()`, `update()`, `list_active()` |
| **提炼引擎** | engine.py | `TelosEngine` | `step1_analyze()`, `step3_decide()`, `step4_generate()`, `step5_judge_growth()`, `run_pipeline()` |
| **信号处理** | pipeline.py | `DistillationPipeline` | `process(signal)` |
| **上下文注入** | context.py | `TelosContextBuilder` | `build_telos_snapshot()`, `inject()` |
| **成长事件** | growth_events.py | `GrowthEventStore` | `save()`, `list_by_user()` |
| **用户画像** | profile/models.py | `UserProfile` | `get_summary()` |
| **用户管理** | profile/manager.py | `UserProfileManager` | `extract_from_message()`, `get_system_prompt_addition()` |
| **人物提取** | people/extractor.py | `PersonExtractor` | `extract_from_text()` |
| **Chat节点** | chat_nodes.py | `build_context()` | 组装system prompt |
| **定时任务** | pipeline_job.py | `PipelineJobManager` | `run_pipeline()`, `approve_item()`, `publish_approved()` |

---

## 总结

本次探索完整覆盖了 Telos 系统的核心架构：

✅ **已完成实现：**
- TelosEngine 5步提炼流程
- RawSignal 和 DistillationPipeline 完整流程
- TelosContextBuilder（已实现但未接入）
- UserProfile 和 Person 模型

⚠️ **设计中但未完全实现：**
- Telos 在Agent对话中的注入（build_context节点）
- People 独立管道的完整编排
- 定期复审机制
- 冷启动问卷流程

⚠️ **需要优化：**
- 流水线合并（Step3+4+5合并为单次LLM调用）
- 置信度重新设计
- 多维度并行执行

所有文件路径、类定义、方法签名均已精确定位和记录。
