---
session_id: agent-362a4a0d
date: 2026-03-30
time_start: 2026-03-30T15:36:41.828Z
time_end: 2026-03-30T15:38:38.992Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-03-30 23:36:41] [user]: 请全面分析 /Users/lianzimeng/workspace/huaqi-growing 这个项目的架构，包括：
1. 顶层目录结构和各模块职责
2. huaqi_src 下各子包的作用
3. 核心数据流（用户输入 → 处理 → 输出）
4. 关键文件列表

请返回尽量详细的结构分析，不需要写代码，只需要描述架构。
[2026-03-30 23:36:46] [assistant]: 我来为你进行全面的架构分析。首先，我会检索相关经验，然后系统地探索项目结构。
[2026-03-30 23:36:49] [assistant]: 现在让我系统地探索项目结构：
[2026-03-30 23:36:52] [assistant]: 让我读取关键文档文件来了解架构：
[2026-03-30 23:36:57] [assistant]: 现在让我进一步深入关键模块：
[2026-03-30 23:37:01] [assistant]: 现在让我检查 CLI 层和一些核心流程：
[2026-03-30 23:37:05] [assistant]: 让我再检查一些重要的核心模块和数据流：
[2026-03-30 23:37:09] [assistant]: 现在让我获取测试文件来了解使用模式：
[2026-03-30 23:37:18] [assistant]: 完美！现在让我创建最后一份总结，并保存经验：
[2026-03-30 23:38:38] [assistant]: 现在让我为你生成一份完整的架构分析报告：

---

# Huaqi 项目架构全面分析

## 📋 项目概述

**Huaqi (花旗)** 是一个个人 AI 同伴系统，核心理念为**"不是使用 AI，而是养育 AI"**，通过长期对话积累对用户的理解，让 AI 逐渐成为真正懂用户的数字伙伴。

**技术栈**：Python 3.9+、LangGraph Agent、APScheduler、Chroma 向量库、BM25 混合检索

---

## 1️⃣ 顶层目录结构

```
huaqi-growing/                    # 源码仓库根目录
├── cli.py                         # CLI 入口（18 行薄包装）
├── pyproject.toml                 # 项目配置、依赖管理
├── requirements.txt               # 依赖锁定
├── README.md                      # 快速上手指南
├── CHANGELOG.md                   # 版本变更
├── QUICK_START.md                 # 5 分钟快速开始
│
├── huaqi_src/                     # 核心源码包（约 100+ 个 .py 文件）
│   ├── agent/                     # LangGraph Agent 工作流层
│   ├── cli/                       # 命令行交互层（Typer）
│   ├── core/                      # 核心业务逻辑层
│   ├── memory/                    # 记忆存储与混合检索
│   ├── pipeline/                  # 内容流水线（采集→处理→发布）
│   ├── scheduler/                 # 定时任务调度
│   ├── collectors/                # 数据采集器
│   ├── people/                    # 人脉关系图
│   ├── reports/                   # 报告生成
│   ├── world/                     # 外部世界信息源
│   └── integrations/              # 第三方集成（WeChat 等）
│
├── docs/                          # 文档体系
│   ├── design/                    # 产品与技术设计文档
│   ├── features/                  # 功能模块文档
│   ├── guides/                    # 用户与开发者指南
│   └── ops/                       # 运维与部署文档
│
├── spec/                          # 架构规范与决策记录
│   ├── IMPORT_GUIDE.md
│   ├── MULTI_USER_DESIGN.md
│   └── decisions/                 # ADR（架构决策记录）
│
├── templates/                     # 用户数据模板
│   ├── memory/
│   ├── personality/
│   └── skills/
│
├── tests/                         # 测试套件
│   ├── agent/                     # Agent 测试
│   ├── core/                      # 核心模块测试
│   ├── cli/                       # CLI 命令测试
│   ├── collectors/                # 采集器测试
│   ├── people/                    # 人脉模块测试
│   ├── reports/                   # 报告生成测试
│   ├── world/                     # 信息源测试
│   └── unit/                      # 单元测试
│
└── scripts/                       # 运维脚本
    └── migrate_v3_to_v4.py        # 版本迁移脚本
```

### 关键目录分离

⚠️ **重要**：源码目录与用户数据目录完全独立

| 目录类型 | 内容 | 示例路径 |
|---------|------|----------|
| **源码目录** | git 仓库，包含代码、文档、配置 | `~/workspace/huaqi-growing/` |
| **用户数据目录** | 运行时用户数据（配置、日记、对话、向量库） | `~/.huaqi/` 或 `~/workspace/huaqi/` |

用户数据目录结构：
```
~/.huaqi/
├── config.yaml                    # 用户配置
├── memory/                        # 记忆数据（独立 git 仓库）
│   ├── diary/                     # 日记（Markdown）
│   ├── conversations/             # 对话历史
│   ├── personality.yaml           # 用户画像
│   ├── growth.yaml                # 技能与目标
│   └── .git/                      # Git 版本控制
├── drafts/                        # 内容草稿
├── vector_db/                     # 向量数据库（可再生）
├── models/                        # 本地模型（可再生）
└── logs/                          # 日志文件
```

---

## 2️⃣ huaqi_src 下各子包的作用

### 2.1 agent/ - LangGraph Agent 工作流层

**职责**：对话流程编排、意图识别、工作流状态管理、人机协同

**关键文件**：
- `state.py` - 统一状态定义（AgentState、ContentPipelineState、DiaryWorkflowState 等）
- `graph/chat.py` - 对话工作流状态图
- `nodes/chat_nodes.py` - 对话节点实现（意图分类、上下文构建、记忆检索、生成回复等）
- `nodes/interrupt_nodes.py` - 人机协同中断节点
- `tools.py` - Agent 工具定义
- `chat_agent.py` - Agent 主类

**依赖关系**：↓ 依赖 core/ 和 memory/

**状态驱动工作流示例**：
```
用户输入 
  → intent_classifier (意图识别)
  → context_builder (构建上下文：人格 + 记忆)
  → memory_retriever (检索相关记忆)
  → chat_response (LLM 生成回复)
  → save_conversation (保存对话历史)
  → END
```

---

### 2.2 cli/ - 命令行交互层

**职责**：用户命令处理、REPL 主循环、终端 UI、子命令组织

**核心结构**：
```
cli/
├── __init__.py              # 主 app 定义，所有子命令挂载点
├── context.py               # 全局单例缓存（ConfigManager、LLMManager 等）
├── chat.py                  # 对话模式主循环、斜杠命令处理
├── ui.py                    # 命令补全、输入框等 UI 组件
├── inbox.py                 # 收件箱管理
└── commands/                # 各子命令模块
    ├── config.py            # config 子命令（设置 LLM、数据目录等）
    ├── profile.py           # profile 子命令（查看/修改用户画像）
    ├── personality.py       # personality 子命令（人格配置）
    ├── pipeline.py          # pipeline 子命令（流水线管理）
    ├── system.py            # system 和 daemon 子命令
    ├── people.py            # people 子命令（人脉管理）
    ├── collector.py         # collector 子命令（数据采集）
    └── proxy.py             # 代理配置
```

**关键命令**：
```bash
huaqi chat                    # 启动对话模式（默认 LangGraph）
huaqi config set llm          # 配置 LLM
huaqi profile show            # 查看用户画像
huaqi pipeline run            # 执行内容流水线
huaqi daemon start            # 启动后台定时任务
```

**cli.py 入口**（只有 22 行）：
```python
# cli.py 是薄包装，直接导入并调用 huaqi_src.cli 的 app
from huaqi_src.cli import app
if __name__ == "__main__":
    app()
```

---

### 2.3 core/ - 核心业务逻辑层

**职责**：配置管理、用户模型、LLM 调用、分析引擎、数据持久化

**按领域划分的关键模块**：

#### 配置管理
- `config_paths.py` - 数据目录路径管理
- `config_simple.py` - 配置加载（无外部依赖的纯实现）
- `config_manager.py` - ConfigManager 单例
- `config_hot_reload.py` - 配置热重载（watchdog 监听）

#### 用户模型
- `profile_models.py` - UserProfile 数据模型（UserIdentity、Preferences、Background）
- `profile_manager.py` - 用户画像持久化与更新
- `profile_narrative.py` - LLM 生成叙事性画像
- `profile_extractor.py` - 启动时后台提取用户数据

#### LLM 与分析
- `llm.py` - LLM 多提供商管理（OpenAI、Anthropic 等）
- `analysis_engine.py` - 分析引擎（日记分析、对话分析）
- `deep_analysis.py` - 深度分析
- `adaptive_understanding.py` - 自适应用户理解（门面）

#### 个性与成长
- `personality_simple.py` - 五维人格引擎（Big Five 模型）
- `personality_updater.py` - 人格更新提案管理
- `growth_simple.py` - 技能与目标追踪
- `diary_simple.py` - 日记存储与检索

#### 特殊系统
- `hooks_simple.py` - Hook 事件系统（对话、日记等事件触发）
- `schema.py` - 动态维度 Schema（可扩展的用户数据维度）
- `git_auto_commit.py` - Git 自动提交（数据版本控制）
- `event.py` - 统一事件结构（自动脱敏）
- `flexible_store.py` - 灵活键值存储

#### 其他
- `ui_utils.py` - 终端 UI 工具（Rich 封装）
- `db_storage.py` - 数据库存储
- `dimension_manager.py` - 维度管理
- `pattern_learning.py` - 模式学习与周报生成
- `proactive_care.py` - 主动关怀系统
- `proactive_exploration.py` - 主动探索

**设计特点**：
- 无横向依赖（core 不依赖其他 huaqi_src 子模块）
- 仅依赖标准库和第三方库
- 每个文件一个明确的职责域
- 统一导出公开 API（`__init__.py` 列表导出）

---

### 2.4 memory/ - 记忆存储与检索层

**职责**：记忆读写、向量检索、文本搜索、混合检索、向量数据库管理

**三层结构**：

```
memory/
├── search/                  # 检索算法
│   ├── hybrid_search.py     # BM25 + 向量混合检索
│   ├── llm_search.py        # LLM 重排序
│   └── text_search.py       # 纯文本搜索
├── storage/                 # 持久化存储
│   └── markdown_store.py    # Markdown 格式存储
└── vector/                  # 向量数据库封装
    ├── chroma_client.py     # Chroma 客户端
    ├── embedder.py          # Embedding 服务封装
    └── hybrid_search.py     # 向量 + BM25 混合检索
```

#### 混合检索设计（核心创新）

**融合公式**：
```
final_score = α × vector_score + (1-α) × bm25_score + recency_boost

其中：
- α = 0.7（向量权重）
- BM25：精确匹配关键词
- 向量：语义相似度
- 时间衰减：近期内容优先
```

**优势**：
- ✅ 无需强制 Embedding 依赖（支持仅用 BM25 + LLM 实现）
- ✅ 兼容所有 LLM 提供商
- ✅ 本地优先，零外部依赖
- ✅ 可选向量检索，按需启用

#### 三层记忆架构

| 层次 | 存储 | 用途 | 特点 |
|------|------|------|------|
| **Session Memory** | 内存 | 当前会话上下文 | 临时、快速 |
| **Working Memory** | YAML | 近期重要信息 | 可编辑、结构化 |
| **Long-term Memory** | Markdown + Chroma | 历史记录 | 持久化、可检索 |

---

### 2.5 pipeline/ - 内容流水线层

**职责**：内容采集、处理、发布（X/Twitter → 摘要 → 小红书文案）

**三层架构**：
```
sources/              processors/              platforms/
(数据采集)            (内容处理)               (平台适配)
  ↓                     ↓                       ↓
X/Twitter API    →  Summarizer        →   XiaoHongShu
RSS Feeds         Translator             Draft Editor
Mock Data         XiaoHongShuFormatter   Publish API
```

**核心组件**：
- `models.py` - ContentItem、ContentStatus、PipelineResult 数据模型
- `core.py` - ContentPipeline 主类（流程编排）
- `sources/` - 数据源适配器（BaseSource、XMockSource、RSSMockSource）
- `processors/` - 处理器（BaseProcessor、Summarizer、Translator、XiaoHongShuFormatter）
- `platforms/` - 发布平台（BasePublisher、XiaoHongShuPublisher）

**工作流示例**：
```
定时触发/手动触发
  ↓
fetch() - 从 X/RSS 抓取内容
  ↓
quality_check() - 过滤低质量
  ↓
summarize() - AI 生成摘要
  ↓
generate_post() - 生成平台文案
  ↓
interrupt() - 人机协同确认
  ↓
publish() - 发布或保存草稿
```

---

### 2.6 scheduler/ - 定时任务层

**职责**：使用 APScheduler 调度定时任务、后台服务管理

**文件说明**：
- `manager.py` - SchedulerManager（APScheduler 封装）
- `handlers.py` - 任务处理器注册
- `pipeline_job.py` - 内容流水线定时任务
- `jobs.py` - 其他定时任务定义

**支持的触发器**：
- Cron（如 `0 8 * * *` 每天早上 8 点）
- Interval（如每 6 小时）
- Date（一次性任务）

**内置任务**：
- `morning_greeting` - 每日晨间问候
- `daily_summary` - 每日总结
- `content_pipeline` - 内容流水线定时执行

---

### 2.7 collectors/ - 数据采集器

**职责**：从多个源自动采集事件数据（CLI 对话、WeChat 消息等）

**核心组件**：
- `cli_chat_parser.py` - 解析 CLI 对话历史
- `cli_chat_watcher.py` - 监听 CLI 对话变化
- `inbox_processor.py` - 收件箱处理
- `document.py` - 文档采集

---

### 2.8 people/ - 人脉关系图

**职责**：建立和维护用户的人脉关系图

**文件**：
- `models.py` - Person、Relationship 数据模型
- `graph.py` - 关系图管理
- `extractor.py` - 从对话中提取人脉信息

---

### 2.9 reports/ - 报告生成

**职责**：生成各类分析报告

**报告类型**：
- `daily_report.py` - 每日报告
- `weekly_report.py` - 周报
- `quarterly_report.py` - 季度报告
- `morning_brief.py` - 晨间简报

---

### 2.10 world/ - 外部世界信息源

**职责**：采集外部信息（新闻、行业动态等）

**组件**：
- `fetcher.py` - 信息抓取
- `sources/` - 数据源（RSS 等）
- `storage.py` - 信息存储

---

### 2.11 integrations/ - 第三方集成

**职责**：与外部平台的集成

**已实现**：
- `wechat_webhook.py` - WeChat Webhook 集成

---

## 3️⃣ 核心数据流

### 3.1 对话流程

```
┌─────────────────────────────────────────────────────────────┐
│ 用户输入 "今天学了 LangGraph，感觉很有趣"                    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 意图识别 (intent_classifier)                                 │
│ → 输出: INTENT_CHAT                                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 构建上下文 (context_builder)                                 │
│ ← 从 core/personality_simple.py 获取人格画像                │
│ ← 从 core/profile_manager.py 获取用户背景                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 记忆检索 (memory_retriever)                                  │
│ ← 调用 memory/vector/hybrid_search.py 混合搜索              │
│    搜索相关的日记、技能、对话历史                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 用户理解分析 (analyze_user_understanding)                    │
│ ← 调用 core/analysis_engine.py 分析用户意图                 │
│ ← 检查 core/schema.py 中的维度信息                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ LLM 生成回复 (generate_response)                             │
│ 调用 core/llm.py 的 LLMManager                              │
│ 提示词 = 人格 + 背景 + 记忆 + 用户输入                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 信息提取 (extract_user_info)                                │
│ ← 从对话中提取：                                             │
│    - 情绪变化 (core/schema.py 情绪维度)                     │
│    - 技能更新 (core/growth_simple.py)                       │
│    - 人脉信息 (people/extractor.py)                         │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 触发 Hook 事件 (core/hooks_simple.py)                        │
│ → on_chat_complete 事件                                     │
│ → 调用所有监听器（画像更新、日志、Git 提交等）             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 保存对话历史 (save_conversation)                             │
│ ← 保存到 ~/.huaqi/memory/conversations/                    │
│ ← 同时更新向量库 (memory/vector/chroma_client.py)           │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 返回回复给用户                                              │
└─────────────────────────────────────────────────────────────┘
```

---

### 3.2 内容流水线流程

```
定时触发 (APScheduler) / 手动触发 (huaqi pipeline run)
                    ↓
            ┌───────────────┐
            │ 源采集         │ (pipeline/sources/)
            │ - X API       │
            │ - RSS 订阅     │
            │ - Mock 数据    │
            └───────────────┘
                    ↓
            ┌───────────────┐
            │ 质量过滤       │
            │ 去重、去垃圾    │
            └───────────────┘
                    ↓
            ┌───────────────┐
            │ 处理器链       │ (pipeline/processors/)
            │ ① Summarizer  │ → 调用 core/llm.py
            │ ② Translator  │ → 多语言翻译
            │ ③ Formatter   │ → 小红书格式化
            └───────────────┘
                    ↓
            ┌───────────────┐
            │ 人机协同       │ (agent/nodes/interrupt_nodes.py)
            │ 等待用户确认   │
            │ (interrupt)   │
            └───────────────┘
                    ↓
    ┌───────────────┴───────────────┐
    │ 用户同意                      │ 用户拒绝
    ↓                               ↓
发布到平台                        存为草稿
(pipeline/platforms/)              (draft_dir)
```

---

### 3.3 人格更新流程

```
日记写入 / 对话结束
        ↓
   分析触发
   (core/analysis_engine.py)
        ↓
┌──────────────────────┐
│ 提取关键信息          │
│ - 情绪                │
│ - 价值观              │
│ - 行为偏好            │
│ (core/schema.py)      │
└──────────────────────┘
        ↓
┌──────────────────────┐
│ 人格冲突检测          │
│ 对比现有 personality │
│ .yaml 的五维数据      │
│ (core/personality_  │
│  simple.py)          │
└──────────────────────┘
        ↓
    ┌───┴───┐
    │无冲突 │有冲突
    ↓       ↓
增量更新  人机确认
         (interrupt)
    ↓       ↓
    └───┬───┘
        ↓
┌──────────────────────┐
│ 保存新版本            │
│ ~/.huaqi/memory/     │
│ personality.yaml     │
└──────────────────────┘
        ↓
┌──────────────────────┐
│ Git 自动提交         │
│ (core/git_auto_     │
│  commit.py)          │
└──────────────────────┘
```

---

### 3.4 系统启动流程

```
huaqi chat / huaqi daemon start
        ↓
初始化上下文 (cli/context.py ensure_initialized())
        ↓
┌────────────────────────────┐
│ 加载配置                    │
│ ← config_paths.py 获取数据目录 │
│ ← config_simple.py 读取 YAML   │
│ ← config_hot_reload.py 启动监听 │
└────────────────────────────┘
        ↓
┌────────────────────────────┐
│ 初始化 LLM                  │
│ ← core/llm.py              │
│ ← 加载 API 密钥              │
└────────────────────────────┘
        ↓
┌────────────────────────────┐
│ 加载用户画像                │
│ ← core/profile_manager.py   │
│ ← core/personality_simple.py│
└────────────────────────────┘
        ↓
┌────────────────────────────┐
│ 初始化记忆系统              │
│ ← memory/vector/chroma      │
│ ← memory/search/hybrid      │
└────────────────────────────┘
        ↓
┌────────────────────────────┐
│ 启动后台任务调度器           │
│ ← scheduler/manager.py      │
│ ← 加载定时任务               │
└────────────────────────────┘
        ↓
系统就绪，进入 REPL 或后台运行
```

---

## 4️⃣ 关键文件列表（30+）

### 顶层入口
| 文件 | 作用 | 重要度 |
|------|------|--------|
| `cli.py` | CLI 入口（薄包装） | ⭐⭐⭐⭐⭐ |
| `pyproject.toml` | 项目配置与依赖管理 | ⭐⭐⭐⭐⭐ |

### Agent 层
| 文件 | 作用 | 重要度 |
|------|------|--------|
| `huaqi_src/agent/state.py` | Agent 全局状态定义 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/agent/graph/chat.py` | 对话工作流状态图 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/agent/nodes/chat_nodes.py` | 对话节点实现 | ⭐⭐⭐⭐ |
| `huaqi_src/agent/chat_agent.py` | Agent 主类 | ⭐⭐⭐⭐ |
| `huaqi_src/agent/tools.py` | Agent 工具定义 | ⭐⭐⭐ |

### CLI 层
| 文件 | 作用 | 重要度 |
|------|------|--------|
| `huaqi_src/cli/__init__.py` | 主 app 定义与子命令挂载 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/cli/chat.py` | 对话主循环与斜杠命令 | ⭐⭐⭐⭐ |
| `huaqi_src/cli/context.py` | 全局单例缓存 | ⭐⭐⭐⭐ |
| `huaqi_src/cli/commands/config.py` | config 子命令 | ⭐⭐⭐ |
| `huaqi_src/cli/commands/profile.py` | profile 子命令 | ⭐⭐⭐ |
| `huaqi_src/cli/commands/pipeline.py` | pipeline 子命令 | ⭐⭐⭐ |

### Core 层（关键模块）
| 文件 | 作用 | 重要度 |
|------|------|--------|
| `huaqi_src/core/__init__.py` | Core 层公开 API 统一导出 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/core/config_paths.py` | 数据目录路径管理 | ⭐⭐⭐⭐ |
| `huaqi_src/core/config_simple.py` | 配置加载 | ⭐⭐⭐⭐ |
| `huaqi_src/core/llm.py` | LLM 多提供商管理 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/core/profile_models.py` | 用户画像数据模型 | ⭐⭐⭐⭐ |
| `huaqi_src/core/profile_manager.py` | 画像持久化与更新 | ⭐⭐⭐⭐ |
| `huaqi_src/core/personality_simple.py` | 五维人格引擎 | ⭐⭐⭐⭐ |
| `huaqi_src/core/growth_simple.py` | 技能与目标追踪 | ⭐⭐⭐ |
| `huaqi_src/core/diary_simple.py` | 日记存储与检索 | ⭐⭐⭐ |
| `huaqi_src/core/schema.py` | 动态维度 Schema 系统 | ⭐⭐⭐⭐ |
| `huaqi_src/core/analysis_engine.py` | 分析引擎 | ⭐⭐⭐ |

### Memory 层
| 文件 | 作用 | 重要度 |
|------|------|--------|
| `huaqi_src/memory/vector/hybrid_search.py` | 混合检索引擎 | ⭐⭐⭐⭐⭐ |
| `huaqi_src/memory/vector/chroma_client.py` | Chroma 向量库封装 | ⭐⭐⭐⭐ |
| `huaqi_src/memory/storage/markdown_store.py` | Markdown 存储 | ⭐⭐⭐ |

### Pipeline 层
| 文件 | 作用 | 重要度 |
|------|------|--------|
| `huaqi_src/pipeline/core.py` | 流水线主逻辑 | ⭐⭐⭐⭐ |
| `huaqi_src/pipeline/models.py` | 数据模型 | ⭐⭐⭐ |
| `huaqi_src/pipeline/sources/base.py` | 源适配器基类 | ⭐⭐⭐ |
| `huaqi_src/pipeline/processors/base.py` | 处理器基类 | ⭐⭐⭐ |
| `huaqi_src/pipeline/platforms/base.py` | 平台适配器基类 | ⭐⭐⭐ |

### Scheduler 层
| 文件 | 作用 | 重要度 |
|------|------|--------|
| `huaqi_src/scheduler/manager.py` | APScheduler 封装 | ⭐⭐⭐⭐ |
| `huaqi_src/scheduler/pipeline_job.py` | 流水线定时任务 | ⭐⭐⭐ |

### 文档
| 文件 | 作用 | 重要度 |
|------|------|--------|
| `docs/design/ARCHITECTURE.md` | 系统架构文档 | ⭐⭐⭐⭐⭐ |
| `docs/guides/code-organization.md` | 代码组织规范 | ⭐⭐⭐⭐⭐ |
| `docs/design/TECH_SPEC.md` | 技术规范 | ⭐⭐⭐⭐ |

---

## 5️⃣ 架构分层设计

### 四层架构

```
┌─────────────────────────────────────────────────────┐
│ 用户接口层 (User Interface)                          │
│ CLI (Typer) + Rich UI + REPL                        │
│ 职责: 命令解析、输入/输出、会话管理                  │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 编排层 (Orchestration)                              │
│ LangGraph Agent + APScheduler + Hook 系统            │
│ 职责: 工作流编排、事件驱动、任务调度                 │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 领域层 (Domain)                                     │
│ Personality + Growth + Profile + Analysis            │
│ 职责: 业务逻辑、数据验证、规则应用                   │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 基础设施层 (Infrastructure)                         │
│ LLM + Memory + Config + Storage + Git                │
│ 职责: 数据存储、检索、模型调用、版本控制             │
└─────────────────────────────────────────────────────┘
```

### 模块依赖关系

```
agent/         (对话编排)
  ↓
core/          (业务逻辑)
  ↓
memory/        (存储检索)
  └→ 向量库 + BM25

pipeline/      (流水线)
  → core/      (LLM、配置)

scheduler/     (定时任务)
  → pipeline/ + core/

collectors/    (数据采集)
  → core/

people/        (人脉管理)
  → core/

reports/       (报告生成)
  → core/

world/         (世界信息)
  → core/

integrations/  (第三方)
  → core/

cli/           (命令行)
  → agent/ + core/
```

**核心原则**：
- 依赖只能向下（agent → core），不能反向
- core 不依赖任何其他 huaqi_src 子模块
- 每层高内聚、低耦合

---

## 6️⃣ 核心设计亮点

### 6.1 状态驱动架构（LangGraph）
- ✅ 状态持久化，支持中断恢复
- ✅ 流程可视化，易于调试
- ✅ 人机协同，interrupt 机制
- ✅ 易于扩展新节点

### 6.2 无需强制 Embedding 的混合检索
- ✅ 兼容所有 LLM（包括不提供 Embedding 的）
- ✅ BM25 + 向量融合，精准度高
- ✅ 可选启用，按需使用
- ✅ 本地优先

### 6.3 人类可读的数据格式
- ✅ Markdown + YAML 存储
- ✅ 直接编辑，无需工具
- ✅ Git 版本控制友好
- ✅ Obsidian 兼容

### 6.4 插件化架构
- ✅ 适配器模式支持扩展
- ✅ 开闭原则
- ✅ 测试友好

### 6.5 配置热重载
- ✅ Daemon 无需重启
- ✅ 修改配置即时生效
- ✅ 多模块协同更新

### 6.6 单用户简化设计
- ✅ 代码简洁
- ✅ 减少维护成本
- ✅ 符合个人使用场景

---

## 7️⃣ 技术栈一览

| 类别 | 技术 | 用途 |
|------|------|------|
| **语言** | Python 3.9+ | 主开发语言 |
| **CLI** | Typer ≥0.9.0 | 命令行框架 |
| **UI** | Rich ≥13.0, Prompt Toolkit ≥3.0 | 终端美化 |
| **Agent** | LangGraph ≥0.2.0 | 状态驱动 |
| **LLM** | LangChain ≥0.3.0, OpenAI ≥1.0 | 模型接口 |
| **调度** | APScheduler ≥4.0 | 定时任务 |
| **向量DB** | Chroma ≥0.5.0 | 向量存储 |
| **搜索** | rank-bm25 ≥0.2.0 | 文本搜索 |
| **数据** | Pydantic ≥2.0, YAML ≥6.0 | 数据验证 |
| **同步** | GitPython ≥3.1.0 | 版本控制 |
| **日志** | structlog ≥24.0 | 结构化日志 |
| **监控** | watchdog ≥4.0 | 文件监听 |

---

## 📊 总结

Huaqi 是一个高度模块化、分层清晰的个人 AI 系统：

1. **架构清晰** - 四层分层 + 明确的依赖关系
2. **设计优雅** - LangGraph 状态驱动 + 混合检索创新
3. **易于扩展** - 适配器模式 + 插件化设计
4. **隐私优先** - 本地存储 + Git 控制 + 用户完全可控
5. **开发友好** - 代码组织规范 + 完整文档 + 单元测试覆盖

整个系统围绕"养育 AI"的理念设计，让 AI 通过长期交互逐渐成为真正懂用户的数字伙伴。
