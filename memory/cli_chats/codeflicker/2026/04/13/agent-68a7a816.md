---
session_id: agent-68a7a816
date: 2026-04-13
time_start: 2026-04-13T11:27:03.607Z
time_end: 2026-04-13T11:33:40.707Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-04-13 19:27:03] [user]: 请全面探索 /Users/lianzimeng/workspace/huaqi-growing 这个项目的代码结构，重点了解：
1. 项目的整体架构和目录结构
2. 核心功能模块（huaqi_src 下各个子目录的作用）
3. CLI 命令支持哪些操作
4. agent、scheduler、layers 各层的功能
5. 数据流是怎样的（数据如何采集、处理、存储）
6. telos 是什么概念，做了什么
7. 支持哪些集成（codeflicker 等）

请返回详细的功能列表和架构说明。
[2026-04-13 19:27:10] [assistant]: 我来帮你全面探索这个项目的代码结构。让我先搜索经验库，然后系统地分析项目架构。
[2026-04-13 19:27:25] [assistant]: 现在让我搜索经验库：
[2026-04-13 19:27:55] [assistant]: 好的，现在让我系统地探索项目结构。我先查看项目的主要目录：
[2026-04-13 19:27:59] [assistant]: 现在让我深入探索各个核心模块的结构和功能：
[2026-04-13 19:28:05] [assistant]: 现在让我查看 CLI 命令、telos 和其他关键模块：
[2026-04-13 19:28:09] [assistant]: 让我查看更多 CLI 命令和 telos 的详细信息：
[2026-04-13 19:28:42] [assistant]: 现在让我读取一些关键文件来了解 telos、scheduler 和 codeflicker：
[2026-04-13 19:28:50] [assistant]: 现在让我查看 codeflicker 集成和数据流相关内容：
[2026-04-13 19:28:54] [assistant]: 现在让我查看一些报告生成功能和 CLI 命令定义：
[2026-04-13 19:29:01] [assistant]: 让我查看报告生成和一些重要的特性文档：
[2026-04-13 19:29:05] [assistant]: 现在让我查看一些测试结构以了解项目的完整性：
[2026-04-13 19:29:11] [assistant]: 让我查看更多特性文档和检查数据层的结构：
[2026-04-13 19:29:18] [assistant]: 完美！现在让我生成最终的综合报告。我已经收集了足够的信息。
[2026-04-13 19:33:40] [assistant]: 现在让我为你生成完整的项目架构报告：

---

# Huaqi 项目全面架构分析报告

> **项目定位**：个人 AI 同伴系统 - "不是使用 AI，而是养育 AI"

---

## 一、整体架构概览

### 1.1 系统三层架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     用户交互层 (CLI)                             │
│              Typer + Rich + Prompt Toolkit                      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   编排层 (LangGraph Agent)                       │
│  Agent工作流 · 定时任务(APScheduler) · Hook系统 · 状态持久化    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│        三层核心业务逻辑 (数据层 ← 能力层 ← 成长层)              │
│ Data(收集存储)  Capabilities(核心能力)  Growth(知识图谱)        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│            基础设施层 (本地存储 + 配置管理)                      │
│  Markdown + YAML · Chroma向量 · BM25搜索 · Git同步 · 配置热重载 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 核心目录结构

```
huaqi-growing/
├── cli.py                               # CLI入口（薄包装）
├── pyproject.toml                       # 项目配置
├── huaqi_src/
│   ├── agent/                           # LangGraph Agent工作流
│   │   ├── state.py                     # 全局State定义
│   │   ├── graph/chat.py                # 对话状态图
│   │   ├── nodes/chat_nodes.py          # 节点实现
│   │   ├── tools.py                     # Agent工具注册
│   │   └── hooks.py                     # 对话后Hook
│   │
│   ├── cli/                             # CLI命令层
│   │   ├── __init__.py                  # 命令组装
│   │   ├── chat.py                      # 对话主循环
│   │   ├── context.py                   # 全局组件缓存
│   │   ├── ui.py/ui_utils.py            # UI工具集
│   │   └── commands/                    # 各子命令模块
│   │       ├── config.py                # 配置管理
│   │       ├── profile.py               # 用户画像
│   │       ├── personality.py           # 人格管理
│   │       ├── pipeline.py              # 内容流水线
│   │       ├── scheduler.py             # 定时任务
│   │       ├── report.py                # 报告查看
│   │       ├── people.py                # 人际管理
│   │       ├── world.py                 # 外部信息
│   │       ├── study.py                 # 学习追踪
│   │       ├── collector.py             # 数据采集
│   │       ├── onboarding.py            # 用户引导
│   │       └── system.py                # 系统管理
│   │
│   ├── config/                          # 统一配置管理
│   │   ├── manager.py                   # ConfigManager
│   │   ├── paths.py                     # 路径函数
│   │   ├── hot_reload.py                # 配置热重载
│   │   ├── users.py                     # 用户管理
│   │   └── adapters/                    # 适配器基类
│   │
│   ├── layers/                          # 三层核心业务（核心！）
│   │   ├── data/                        # 数据层：收、存、不丢
│   │   │   ├── collectors/              # 外部数据采集
│   │   │   │   ├── cli_chat_watcher.py  # CLI对话监听
│   │   │   │   ├── work_data_source.py  # 工作数据采集
│   │   │   │   └── work_sources/        # 工作数据源
│   │   │   │       ├── codeflicker.py   # Codeflicker集成
│   │   │   │       ├── huaqi_docs.py    # Huaqi文档
│   │   │   │       └── kuaishou_docs.py # 快手文档
│   │   │   ├── raw_signal/              # 统一摄取入口
│   │   │   │   ├── models.py            # RawSignal数据模型
│   │   │   │   ├── pipeline.py          # 提炼管道
│   │   │   │   └── store.py             # 存储
│   │   │   ├── memory/                  # 记忆系统
│   │   │   │   ├── search/              # BM25+LLM混合检索
│   │   │   │   ├── vector/              # Chroma向量数据库
│   │   │   │   └── storage/             # Markdown存储
│   │   │   ├── profile/                 # 用户画像
│   │   │   │   ├── models.py            # 画像数据模型
│   │   │   │   ├── manager.py           # 持久化管理
│   │   │   │   └── narrative.py         # LLM叙事生成
│   │   │   ├── events/                  # 事件流
│   │   │   │   ├── models.py            # Event模型
│   │   │   │   └── store.py             # SQLite存储
│   │   │   ├── diary/                   # 日记存储
│   │   │   ├── world/                   # 外部世界信息(RSS)
│   │   │   ├── flexible/                # 灵活键值存储
│   │   │   └── git/                     # Git自动提交
│   │   │
│   │   ├── capabilities/                # 能力层：帮用户干活
│   │   │   ├── llm/                     # LLM抽象(多提供商)
│   │   │   ├── personality/             # 五维人格引擎
│   │   │   ├── growth/                  # 技能与目标追踪
│   │   │   ├── learning/                # 学习助手
│   │   │   ├── pattern/                 # 模式学习与周报
│   │   │   ├── care/                    # 主动关怀系统
│   │   │   ├── hooks/                   # Hook事件系统
│   │   │   ├── pipeline/                # 内容流水线
│   │   │   │   ├── sources/             # 数据源(X, RSS)
│   │   │   │   ├── processors/          # 处理器(摘要, 翻译)
│   │   │   │   └── platforms/           # 平台适配(小红书)
│   │   │   ├── reports/                 # 定时报告生成
│   │   │   │   ├── morning_brief.py     # 晨报
│   │   │   │   ├── daily_report.py      # 日报
│   │   │   │   ├── weekly_report.py     # 周报
│   │   │   │   ├── quarterly_report.py  # 季报
│   │   │   │   └── providers/           # 数据提供者注册表
│   │   │   ├── onboarding/              # 用户引导
│   │   │   └── codeflicker/             # Codeflicker集成
│   │   │
│   │   └── growth/                      # 成长层：理解、提炼、更新
│   │       └── telos/                   # TELOS知识图谱(核心！)
│   │           ├── models.py            # 维度数据模型
│   │           ├── engine.py            # 5步提炼引擎
│   │           ├── manager.py           # 维度管理
│   │           ├── growth_events.py     # 成长事件
│   │           ├── meta.py              # 元数据
│   │           └── dimensions/people/   # 人物关系维度
│   │
│   ├── scheduler/                       # 定时任务系统
│   │   ├── manager.py                   # APScheduler管理器
│   │   ├── jobs.py                      # 任务定义
│   │   ├── job_runner.py                # 任务执行
│   │   ├── scheduled_job_store.py       # 任务存储
│   │   ├── handlers.py                  # 任务处理器
│   │   ├── execution_log.py             # 执行日志
│   │   └── distillation_job.py          # Telos蒸馏任务
│   │
│   └── integrations/                    # 外部集成
│       └── wechat_webhook.py            # 微信Webhook(已封存)
│
├── docs/                                # 文档
│   ├── design/ARCHITECTURE.md           # 架构设计
│   ├── design/adr/                      # 架构决策记录
│   ├── features/                        # 功能说明
│   ├── guides/                          # 使用指南
│   └── plans/                           # 开发计划
│
└── tests/                               # 测试
    ├── unit/                            # 单元测试
    ├── integration/                     # 集成测试
    └── e2e/                             # 端到端测试
```

---

## 二、核心功能模块详解

### 2.1 Agent 层 - LangGraph 状态驱动工作流

**位置**：`huaqi_src/agent/`

**核心概念**：基于 LangGraph 的状态图驱动架构

**关键文件**：
- `state.py` - 全局 AgentState (TypedDict)
- `graph/chat.py` - 对话工作流状态图
- `nodes/chat_nodes.py` - 节点实现
- `tools.py` - Agent 工具注册
- `hooks.py` - 对话后 Hook

**AgentState 结构**：
```python
messages              # 对话历史 (支持增量累积)
user_id              # 用户ID
personality_context  # 人格画像上下文
recent_memories      # 相关历史记忆
intent               # 意图识别
workflow_data        # 工作流状态数据
interrupt_requested  # 人机协同标志
response             # 最终回复
```

**工作流**：
```
用户输入 → intent_classifier → context_builder → chat_response 
         → extract_insights → save_conversation → END
```

**特色**：
- ✅ 状态持久化，支持中断恢复（人机协同）
- ✅ 流程可视化，易于调试和扩展
- ✅ 支持 Interrupt 机制暂停用户确认

---

### 2.2 三层架构 - layers/ 核心业务逻辑

#### **Data 层** - 数据采集、存储、提炼

**目标**：所有数据输入的统一入口，不丢失任何原始信号

**核心子模块**：

| 模块 | 功能 | 关键类/函数 |
|------|------|----------|
| **collectors/** | 外部数据采集 | CLIChatWatcher, WorkDataSource, CodeflickerSource |
| **raw_signal/** | 统一信号摄取 | RawSignal, RawSignalStore, DistillationPipeline |
| **memory/** | 记忆检索系统 | HybridSearch (BM25+向量), MarkdownStore, ChromaClient |
| **profile/** | 用户画像 | UserProfile, ProfileManager, NarrativeGenerator |
| **events/** | 事件流 | Event, LocalDBStorage (SQLite) |
| **diary/** | 日记存储 | DiaryStore |
| **world/** | 外部信息 | RSSPipeline, WorldStorage |
| **flexible/** | 键值存储 | FlexibleStore (用于任意JSON) |
| **git/** | 版本控制 | GitAutoCommit |

**数据流**：
```
CLI对话/微信/RSS ← 采集器 → RawSignal 
                              ↓
                        提炼管道 (Telos)
                              ↓
                    更新维度 + 生成成长事件
```

#### **Capabilities 层** - AI 能力实现

**目标**：赋予 AI 帮助用户的各项能力

**核心子模块**：

| 模块 | 功能 | 关键特性 |
|------|------|---------|
| **llm/** | LLM 抽象 | 多提供商支持(OpenAI等), 参数管理, 接口统一 |
| **personality/** | 五维人格引擎 | 5个维度调节, 预设角色模式, 动态Prompt生成 |
| **growth/** | 成长追踪 | 技能管理, 目标追踪, 进度统计 |
| **learning/** | 学习助手 | 课程生成, 进度存储, 学习工具集 |
| **pattern/** | 模式学习 | 行为分析, 周报生成 |
| **care/** | 主动关怀 | 情绪监测, 主动问候, 事件触发 |
| **hooks/** | Hook 系统 | 对话后、日记后等事件的自定义处理 |
| **pipeline/** | 内容流水线 | 数据源采集 → 摘要 → 格式化 → 发布 |
| **reports/** | 报告生成 | 晨报/日报/周报/季报, 数据提供者注册表 |
| **codeflicker/** | Codeflicker 集成 | Claude Markdown 转换 |
| **onboarding/** | 用户引导 | 问卷采集, Telos 初始化 |

#### **Growth 层** - 知识理解与更新

**目标**：深层理解用户心智模型，动态建立、更新、存储用户的认知维度

**核心概念 - TELOS 知识图谱**：

TELOS 是 Huaqi 的「成长智能系统」，参考 Daniel Miessler 的 Personal AI Infrastructure。核心理念是将**原始信号蒸馏为具名的认知维度**，实现压缩、稳定性和可解释性。

**8 个标准认知维度**（按 DimensionLayer 分类）：

| 维度 | 层级 | 说明 |
|------|------|------|
| **beliefs** | CORE | 核心信念和价值观 |
| **models** | CORE | 心智模型和理论框架 |
| **narratives** | CORE | 个人故事和身份认知 |
| **goals** | MIDDLE | 当前目标和抱负 |
| **challenges** | MIDDLE | 面临的挑战和困难 |
| **strategies** | MIDDLE | 采用的策略和方法 |
| **learned** | SURFACE | 最近学到的知识 |
| **shadows** | SURFACE | 心理盲点和隐性假设 |

**第 9 个维度 - People 关系维度**：
- 独立于上述 8 个维度，存储人物关系数据
- 支持多个 Person 对象，更新语义为"新增/更新人物"
- 有专属 people_pipeline 处理

**5 步 Telos 提炼流程**：

```
Step 1: 信号分析      
  └─ LLM 分析原始信号涉及的维度、情绪、强度
     输出：dimensions[], emotion, intensity, signal_strength, new_dimension_hint

Step 2: 维度积累检查
  └─ 查询 30 天内涉及该维度的信号数
     是否达到阈值（强信号或 ≥3 条）

Step 3+4+5: 合并为统一流程（优化）
  └─ LLM 判断是否需要更新维度内容
     输出：should_update, new_content, consistency_score, is_growth_event

维度更新 + 成长事件记录
```

**TelosDimension 数据模型**：
```python
name              # 维度名称
layer             # DimensionLayer (CORE/MIDDLE/SURFACE)
content           # 当前认知内容
confidence        # 置信度 (0.0~1.0)
update_count      # 更新次数
history           # 版本历史
is_active         # 是否激活
is_custom         # 是否自定义维度
```

**置信度计算**（已改进）：
```
confidence = count_score * 0.4 + consistency_score * 0.6
  count_score = 近30天涉及该维度的信号数 / 10
  consistency_score = LLM判断的一致性分数 (0.0~1.0)
```

**维度存储格式**（Markdown + Frontmatter）：
```markdown
---
dimension: beliefs
layer: core
confidence: 0.85
updated_at: 2026-04-11
update_count: 5
---

## 当前认知
用户是一个追求精进的工程师，相信...

---

## 更新历史

### v4 · 2026-04-10
**变化**：对成长的理解更新
**触发**：用户提及新的学习方向
**置信度**：0.80
```

**GrowthEvent 成长事件**：
TELOS 提炼过程中可能识别出关键的成长事件，如「领悟、突破、放弃、新开始」等。

**优化亮点**：
- ✅ 无需外部 Embedding，可纯 BM25 + LLM 实现
- ✅ 维度时效检查，定时复审过时内容
- ✅ People 维度独立处理，避免强行混合
- ✅ 合并 Step 3/4/5 减少 LLM 调用，改善性能

---

### 2.3 记忆系统 - 混合检索引擎

**位置**：`huaqi_src/layers/data/memory/`

**三层记忆架构**：

| 层次 | 存储介质 | 用途 | 特点 |
|------|--------|------|-----|
| **Session Memory** | 内存 | 当前会话上下文 | 临时、快速 |
| **Working Memory** | YAML | 近期重要信息 | 可编辑、结构化 |
| **Long-term Memory** | Markdown + Chroma | 历史记录 | 持久化、可检索 |

**混合检索策略**：

```python
score = 0.7 * vector_similarity     # 向量语义匹配
      + 0.3 * bm25_score            # BM25 关键词匹配
      + 0.1 * recency_score         # 时间衰减权重

优势：
- 兼容所有 LLM（无需强制 Embedding）
- 本地优先，零依赖外部向量服务
- 精确匹配 + 语义匹配结合
```

**检索实现**：
- `llm_hybrid_search.py` - LLM 辅助判断相关性
- `text_search.py` - 纯文本 BM25 检索
- `llm_search.py` - LLM 直接检索

---

### 2.4 内容流水线 - X/RSS → 小红书

**位置**：`huaqi_src/layers/capabilities/pipeline/`

**流水线架构**：
```
Sources (采集)
  ↓
Processors (处理：摘要、翻译、格式化)
  ↓
Platforms (平台适配：小红书等)
  ↓
Publish (发布或保存草稿)
```

**已实现组件**：

| 层次 | 组件 | 说明 |
|------|------|------|
| **Sources** | XMockSource | Twitter/X 数据源（mock 模式） |
| | RSSMockSource | RSS 数据源（mock 模式） |
| **Processors** | Summarizer | 摘要生成 |
| | Translator | 翻译（可选） |
| | XiaoHongShuFormatter | 小红书文案格式化 |
| **Platforms** | XiaoHongShuPublisher | 小红书发布（草稿模式） |

**使用示例**：
```python
pipeline = (PipelineBuilder()
    .add_x_source(mock=True)
    .add_rss_source(mock=True)
    .add_summarizer(max_length=500)
    .add_xiaohongshu_formatter()
    .set_xiaohongshu_publisher(draft_dir="~/.huaqi/drafts")
    .build())

await pipeline.run(dry_run=True)  # 预览模式
```

---

### 2.5 调度器 - APScheduler 定时任务

**位置**：`huaqi_src/scheduler/`

**设计特色**（ADR-007）**：
- 所有定时任务统一为 Chat 任务（无\"内置函数\"任务）
- 任务定义纯数据化，存储在 `scheduled_jobs.yaml`
- 统一执行入口：`_run_scheduled_job(job_id, prompt, output_dir?)`
- Daemon 进程监听 YAML 变更，自动重载，无需重启

**内置预置任务**：
| 任务 | 默认频率 | 说明 |
|------|--------|------|
| `morning_brief` | 每天 08:00 | 晨间问候/简报 |
| `daily_report` | 每天 22:00 | 日终复盘 |
| `weekly_report` | 每周一 09:00 | 周总结 |
| `quarterly_report` | 每季度首日 | 季度回顾 |
| `distillation_job` | 每小时 | TELOS 提炼 (处理 RawSignal) |
| `content_pipeline` | 每天 12:00 | 内容流水线 (X/RSS 采集) |

**任务存储格式**（`scheduled_jobs.yaml`）：
```yaml
jobs:
  - id: morning_greeting
    display_name: "晨间问候"
    cron: "0 8 * * *"
    enabled: true
    prompt: "早上好，请根据我最近的状态给出今天的建议"
    output_dir: null

  - id: daily_summary
    display_name: "日终复盘"
    cron: "0 22 * * *"
    enabled: true
    prompt: "请生成我的今日复盘报告..."
    output_dir: "reports/daily"
```

---

### 2.6 报告生成系统 - DataProvider 注册表

**位置**：`huaqi_src/layers/capabilities/reports/`

**设计思路**：
- 各数据模块实现 `DataProvider` 接口并自注册
- 报告类通过 `build_context()` 统一收集上下文
- 新增数据模块时，无需修改报告文件

**数据提供者**：

| 提供者 | 来源数据 | 支持的报告 |
|--------|---------|----------|
| **WorldProvider** | RSS 世界新闻 | daily/weekly/quarterly |
| **DiaryProvider** | 日记条目 | daily/weekly/quarterly |
| **PeopleProvider** | 人际关系 | weekly/quarterly |
| **WorkLogProvider** | 工作日志 | daily |
| **LearningProvider** | 学习进度 | weekly/quarterly |
| **EventsProvider** | 事件流 | daily |
| **GrowthProvider** | 成长目标 | quarterly |

**报告类型**：
- `morning_brief` - 晨间简报（快速阅读）
- `daily_report` - 日终复盘
- `weekly_report` - 周总结
- `quarterly_report` - 季度回顾

---

### 2.7 Codeflicker 集成

**位置**：`huaqi_src/layers/data/collectors/work_sources/codeflicker.py`

**功能**：自动采集 Codeflicker IDE 中的对话历史

**支持格式**：
- `.jsonl` - Codeflicker 原生格式，逐行解析 JSON
- `.md` - Markdown 格式（兼容旧格式）

**集成方式**：
```python
source = CodeflickerSource(cli_chats_dir="~/.codeflicker/projects/")
documents = source.fetch_documents(since=datetime(2026, 4, 1))
```

**数据写入路径**：
```
{data_dir}/memory/cli_chats/codeflicker/
├── YYYY/MM/DD/
│   └── {session_id}.md
```

---

## 三、CLI 命令系统

**位置**：`huaqi_src/cli/commands/`

### 3.1 命令组织结构

CLI 采用 Typer 框架，命令分组管理，共 12 个命令组：

| 命令组 | 文件 | 主要命令 |
|--------|------|--------|
| **chat** | __init__.py | `huaqi chat` 启动对话 |
| **config** | config.py | `huaqi config show/set` |
| **profile** | profile.py | `huaqi profile show/set` |
| **personality** | personality.py | `huaqi personality update/review/show` |
| **pipeline** | pipeline.py | `huaqi pipeline show/run/preview/drafts/review` |
| **scheduler** | scheduler.py | `huaqi scheduler list/add/remove/enable/disable/run` |
| **report** | report.py | `huaqi report morning/daily/weekly` |
| **people** | people.py | `huaqi people list/show/add/note/delete` |
| **world** | world.py | `huaqi world fetch` |
| **study** | study.py | 学习追踪命令 |
| **collector** | collector.py | 数据采集管理 |
| **system** | system.py | `huaqi daemon/status/migrate/backup` |
| **inbox** | inbox.py | 邮件/消息管理 |

### 3.2 核心命令列表

```bash
# 对话
huaqi chat                              # 启动 LangGraph Agent 对话
huaqi chat --list-sessions              # 列出历史会话
huaqi chat --session <thread_id>        # 恢复会话
huaqi status                            # 查看系统状态

# 配置管理
huaqi config show                       # 查看配置
huaqi config set llm                    # 配置 LLM（交互向导）
huaqi config set data_dir <path>        # 设置数据目录

# 用户画像
huaqi profile show                      # 查看画像
huaqi profile set name 小蒙             # 修改字段

# 人格管理
huaqi personality show                  # 查看人格配置
huaqi personality update                # 更新人格
huaqi personality review                # 复审人格

# 内容流水线
huaqi pipeline show                     # 查看流水线状态
huaqi pipeline run                      # 执行流水线
huaqi pipeline preview                  # 预览内容
huaqi pipeline drafts                   # 查看草稿
huaqi pipeline review                   # 审核草稿

# 定时任务
huaqi scheduler list                    # 列出任务
huaqi scheduler add                     # 添加任务
huaqi scheduler remove <job_id>         # 移除任务
huaqi scheduler enable/disable <job_id> # 启用/禁用任务
huaqi scheduler run <job_id>            # 手动运行任务

# 报告生成
huaqi report morning                    # 生成晨报
huaqi report daily                      # 生成日报
huaqi report weekly                     # 生成周报

# 人际管理
huaqi people list                       # 列出人物
huaqi people show <person_id>           # 查看人物
huaqi people add                        # 添加人物
huaqi people note <person_id> <note>    # 添加备注
huaqi people delete <person_id>         # 删除人物

# 外部信息
huaqi world fetch                       # 采集外部信息

# 系统管理
huaqi daemon start/stop/status/list     # 后台任务管理
huaqi system migrate                    # 数据迁移
huaqi system backup                     # 数据备份
huaqi system hot-reload                 # 配置热重载

# 恢复中断的任务
huaqi resume <task_id>
```

---

## 四、数据流全景

### 4.1 完整数据流向

```
┌─────────────────────────────────────────────────────────────┐
│                      外部数据源                              │
│  CLI 对话 · Codeflicker · WeChat · RSS · 日记 · 用户输入   │
└────────────────────────┬────────────────────────────────────┘
                         │
                    RawSignal 统一摄取
                         ↓
        ┌────────────────────────────────────┐
        │   RawSignalStore (SQLite)          │
        │   (标记: processed, distilled)     │
        └────────────────┬───────────────────┘
                         │
                    DistillationPipeline
                    (定时或手动触发)
                         ↓
        ┌────────────────────────────────────┐
        │  Telos 5步提炼                      │
        │  Step1: 信号分析 → dimensions[]    │
        │  Step2: 积累检查 → 是否达阈值      │
        │  Step3+4+5: 维度更新 + 置信度      │
        └────────────────┬───────────────────┘
                         │
        ┌────────┬────────┴────────┬─────────┐
        ↓        ↓                 ↓         ↓
    维度更新  成长事件  新增自定义维度  People维度
    (Markdown)  (Event)    (create)      (pipeline)
        │        │                │         │
        └────────┴────────┬───────┴─────────┘
                         │
        ┌────────────────────────────────────┐
        │  Agent Context 注入                 │
        │  (Telos Index + 用户画像 + 记忆)  │
        └────────────────┬───────────────────┘
                         │
        ┌────────────────────────────────────┐
        │  LangGraph Chat Agent               │
        │  ├─ Intent 分类                     │
        │  ├─ 记忆检索                        │
        │  └─ LLM 生成回复                   │
        └────────────────┬───────────────────┘
                         │
        ┌────────────────────────────────────┐
        │  Post-Processing                    │
        │  ├─ 洞察提取                        │
        │  ├─ 对话保存                        │
        │  └─ Hook 触发                       │
        └────────────────────────────────────┘
```

### 4.2 数据库表结构

**RawSignal 表**（SQLite）：
```
id              UUID
user_id         用户ID
source_type     数据源类型 (journal/ai_chat/wechat/etc)
timestamp       信号时间
ingested_at     摄取时间
content         内容
metadata        JSON 元数据
processed       是否已分析 (boolean)
distilled       是否已提炼 (boolean)
vectorized      是否已向量化 (boolean)
embedding       向量值 (可选)
```

**Event 表**（SQLite）：
```
id              UUID
user_id         用户ID
event_type      事件类型 (growth/milestone/etc)
timestamp       事件时间
title           标题
narrative       叙述
metadata        JSON
```

**Telos 维度文件**（Markdown）：
```
{data_dir}/memory/telos/{user_id}/{dimension_name}.md
```

---

## 五、核心特色设计

### 5.1 无需强制 Embedding 的混合检索

**传统方案**：
```
必须 Embedding → Chroma 向量检索 → 依赖特定模型
```

**Huaqi 方案**：
```
BM25 文本检索 + 可选向量 + LLM 相关性判断
```

**优势**：
- ✅ 兼容所有 LLM（包括不提供 Embedding 的）
- ✅ 本地优先，无需外部向量服务
- ✅ 可选 Chroma，按需启用
- ✅ 时间衰减确保新信息优先

### 5.2 LangGraph 状态驱动架构

**vs 传统流程式**：

传统：
```python
# 命令式，状态隐式
def chat(user_input):
    intent = classify(user_input)
    if intent == "chat":
        response = generate(user_input)
    elif intent == "diary":
        response = handle_diary(user_input)
```

Huaqi：
```python
# 声明式，状态显式
workflow.add_node("classifier", classify_intent)
workflow.add_node("chat", generate)
workflow.add_conditional_edges("classifier", route_by_intent)
```

**优势**：
- ✅ 状态持久化，支持暂停/恢复
- ✅ 流程可视化，易于调试
- ✅ 人机协同 Interrupt 机制
- ✅ 易于扩展新节点

### 5.3 人类可读的本地存储

**数据格式**：
- Markdown（日记、对话、维度）
- YAML（配置、任务定义）
- JSON/JSONL（结构化数据）
- SQLite（事件流、原始信号）

**优势**：
- ✅ 直接编辑，无需特殊工具
- ✅ Git 版本控制友好
- ✅ Obsidian 兼容，与知识库互通
- ✅ 避免数据库锁定

### 5.4 插件化设计

**适配器模式**：
```python
# 平台适配器
class BasePlatformAdapter(ABC):
    async def publish(self, content) -> PublishResult

# 数据源适配器
class BaseDataSourceAdapter(ABC):
    async def fetch_latest(self, query: str) -> List[RawContent]
```

**优势**：
- ✅ 易于扩展新平台/数据源
- ✅ 遵循开闭原则
- ✅ 测试友好

### 5.5 配置热重载

```python
# Watchdog 监听配置变更
class ConfigReloader(FileSystemEventHandler):
    def on_modified(self, event):
        if event.src_path == config_path:
            reload_config()  # Daemon 无需重启
```

**优势**：
- ✅ 修改配置即时生效
- ✅ 无需重启 Daemon
- ✅ 支持多模块协同更新

### 5.6 统一配置管理

**新增工厂函数**（无参）：
```python
# 旧方式（需要传入 data_dir）
config = ConfigManager(data_dir=Path.home() / ".huaqi")

# 新方式（自动查找）
config = get_config_manager()  # 自动从 HUAQI_DATA_DIR 或默认位置
```

### 5.7 任务统一为 Chat 任务（ADR-007）

**之前**：内置函数任务 + Chat 任务 双轨制

**现在**：所有任务 = Chat 任务（纯数据驱动）

**优势**：
- ✅ 用户可通过对话/CLI 完成所有任务 CRUD
- ✅ 执行路径统一，维护成本低
- ✅ 任务定义纯数据化，YAML 可直接编辑
- ✅ Daemon 监听文件变更，自动重载

---

## 六、集成支持

### 6.1 Codeflicker 集成

**功能**：自动采集 IDE 中的对话记录

**支持格式**：
- JSONL（标准格式）
- Markdown（兼容格式）

**采集内容**：
- 用户提问
- 助手回复
- 时间戳
- Session ID
- Git 分支信息

### 6.2 微信集成（已封存）

**状态**：⚠️ 已封存

**原因**：
- WeChat 4.x macOS 使用 SQLCipher 加密
- macOS SIP 防护阻止数据库读取
- 无法在不破坏系统安全的前提下实现

**文件保留**：底层代码保留但不接入任何系统入口

### 6.3 外部 API 支持

- **LLM**：OpenAI、Claude、本地模型等（通过 LangChain）
- **向量库**：Chroma（可选）
- **向量模型**：sentence-transformers
- **调度**：APScheduler
- **CLI 框架**：Typer + Rich

---

## 七、测试覆盖

**位置**：`tests/`

**测试类型**：

| 测试类型 | 路径 | 覆盖范围 |
|---------|------|--------|
| **单元测试** | tests/unit/ | 各模块独立功能 |
| **集成测试** | tests/integration/ | 跨模块数据流 |
| **端到端** | tests/e2e/ | 完整用户场景（真实LLM） |

**主要测试模块**：
- `test_chat_nodes.py` - Agent 节点
- `test_tools.py` - Agent 工具
- `test_telos_engine.py` - Telos 提炼
- `test_raw_signal_*.py` - 信号处理
- `test_daily_report.py` - 报告生成
- `test_scheduler*.py` - 定时任务

---

## 八、架构层次依赖关系

```
┌─────────────────────────────────────────┐
│  CLI 层 (cli/) + Agent 层 (agent/)      │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│  Capabilities 层 (能力)                   │
│  ├─ llm/personality/growth/learning    │
│  ├─ pattern/care/hooks                 │
│  ├─ pipeline/reports/onboarding        │
│  └─ codeflicker                        │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│  Data 层 (数据)                          │
│  ├─ collectors/raw_signal              │
│  ├─ memory (混合检索)                   │
│  ├─ profile/events/diary               │
│  └─ world/flexible/git                 │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│  Growth 层 (理解)                        │
│  └─ telos/ (知识图谱)                   │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│  Config 层 + Scheduler 层                │
│  (配置管理 + 定时驱动)                  │
└─────────────────────────────────────────┘

禁止逆向依赖！
```

---

## 九、核心文件索引

| 文件路径 | 作用 | 优先级 |
|---------|------|------|
| `cli.py` | CLI 入口 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/cli/__init__.py` | 命令组装 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/agent/graph/chat.py` | Agent 工作流 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/agent/state.py` | Agent 状态 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/layers/growth/telos/engine.py` | Telos 提炼 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/layers/growth/telos/models.py` | 维度模型 | ⭐⭐⭐⭐ |
| `huaqi_src/layers/data/raw_signal/pipeline.py` | 提炼管道 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/layers/data/memory/search/hybrid_search.py` | 混合检索 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/config/manager.py` | 配置管理 | ⭐⭐⭐⭐ |
| `huaqi_src/scheduler/manager.py` | APScheduler 封装 | ⭐⭐⭐⭐ |
| `huaqi_src/layers/capabilities/reports/providers/__init__.py` | 数据提供者注册 | ⭐⭐⭐⭐ |
| `docs/design/ARCHITECTURE.md` | 架构详解 | ⭐⭐⭐⭐⭐ |
| `docs/design/adr/ADR-006-three-layer-architecture.md` | 三层架构决策 | ⭐⭐⭐⭐ |
| `docs/design/adr/ADR-007-scheduler-unified-chat-tasks.md` | 任务统一决策 | ⭐⭐⭐⭐ |

---

## 十、项目亮点总结

| 亮点 | 说明 |
|------|------|
| **LangGraph 状态驱动** | 流程可视化、支持中断恢复、人机协同 |
| **三层架构** | 清晰的职责分离：Data → Capabilities → Growth |
| **TELOS 知识图谱** | 将原始信号蒸馏为命名维度，实现深层理解 |
| **混合检索引擎** | BM25 + 向量 + LLM，兼容性强、效果优秀 |
| **本地优先存储** | Markdown + YAML，Git 友好，用户完全控制 |
| **插件化设计** | 适配器模式，易于扩展新能力 |
| **配置热重载** | Daemon 无需重启，修改即时生效 |
| **任务数据驱动** | 所有定时任务纯数据化，无需改代码 |
| **隐私保护** | 本地存储，无云上传，用户数据完全可控 |
| **工程文化** | 详细的 ADR、完整的文档、清晰的代码规范 |

---

**文档生成时间**：2026-04-13  
**项目版本**：v0.1.0  
**Python 版本**：3.9+
