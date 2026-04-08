---
session_id: agent-196f62b2
date: 2026-03-30
time_start: 2026-03-30T07:47:19.786Z
time_end: 2026-03-30T07:50:19.979Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-03-30 15:47:19] [user]: 请探索 /Users/lianzimeng/workspace/huaqi-growing 项目，重点阅读 README.md、CHANGELOG.md、pyproject.toml 以及 huaqi_src 目录结构，总结该项目目前具备的所有功能模块和特性。请返回一个清晰的功能列表。
[2026-03-30 15:47:23] [assistant]: 我来帮你探索这个项目。首先，让我读取关键文件并了解项目结构。
[2026-03-30 15:47:34] [assistant]: 现在让我深入探索 huaqi_src 目录的关键模块，以获得更完整的功能信息：
[2026-03-30 15:47:39] [assistant]: 现在让我读取一些关键的功能文档，以获得更详细的功能信息：
[2026-03-30 15:47:45] [assistant]: 现在让我查看更多关键的功能模块，特别是内存系统、管道系统和调度系统：
[2026-03-30 15:47:51] [assistant]: 现在让我查看核心引擎和内存系统的文档：
[2026-03-30 15:48:01] [assistant]: 让我再查看一些重要的功能文档：
[2026-03-30 15:48:53] [assistant]: 现在让我读取一个完整的特性文档和配置文件，以获得更多的功能细节：
[2026-03-30 15:49:26] [assistant]: 完美！现在我有了足够的信息。让我生成一份全面的功能列表总结：
[2026-03-30 15:50:19] [assistant]: 完美！现在我有了足够的信息来生成一份详尽的功能列表。让我整理并输出：

---

## 📋 Huaqi (花旗) 项目功能模块总结

基于对 README.md、CHANGELOG.md、pyproject.toml、huaqi_src 目录结构和功能文档的深入分析，以下是项目目前具备的所有功能模块和特性的详细清单。

### 🎯 **核心产品定位**

**Huaqi** 是一个个人 AI 同伴系统，核心理念是"不是使用 AI，而是养育 AI"。通过长期对话积累对用户的理解，主动关心用户成长，成为真正懂你的 AI 同伴。

---

## I. 🗣️ **对话系统** (Agent 交互层)

### 1. LangGraph Agent 架构 (v0.2.0+)
- **基于 LangGraph StateGraph** 的编程范式，替代传统 while 循环
- **流式对话输出**：实时渲染用户输入和 AI 回复
- **会话持久化**：基于 `AsyncSqliteSaver` 的 Checkpoint 机制
  - 支持会话新建、列表展示、会话恢复
  - 命令：`huaqi chat -l` (列出历史) / `huaqi chat -s <thread_id>` (恢复会话)
- **人机协同中断恢复**：LangGraph 工作流支持在任意节点设置中断点
  - 命令：`huaqi resume <task_id> [response]`

### 2. 对话工作流节点（7-8个核心节点）
```
intent_classifier
    ↓
context_builder (构建系统提示词)
    ↓
memory_retriever (混合检索)
    ↓
tools_node (工具调用路由)
    ↓
user_analyzer (维度分析：情绪/能量/焦虑/动机)
    ↓
generate_response (LLM 流式回复)
    ↓
extract_user_info (画像提取与更新)
    ↓
save_conversation (持久化)
```

### 3. 对话中的斜杠命令（15+个）
- **技能管理**: `/skill <名称>` (添加) / `/log <技能> <小时>` (记录练习) / `/skills` (查看列表)
- **目标管理**: `/goal <标题>` (设定) / `/goals` (查看列表)
- **日记系统**: `/diary` (写日记) / `/diary list` (查看) / `/diary search <词>` (搜索) / `/diary import <路径>` (批量导入)
- **状态查询**: `/status` (详细状态) / `/report` (周报) / `/report insights` (洞察)
- **会话控制**: `/reset` (新建会话) / `/state` (查看会话ID和轮数) / `/clear` (清屏) / `/help` (帮助)
- **关怀系统**: `/care` (手动触发) / `/care status` (统计)

### 4. 键盘快捷键支持
- `↑/↓`: 历史记录导航
- `Tab`: 命令补全 + 历史词组补全
- `Ctrl+R`: 搜索历史
- `Ctrl+O` / `Esc+Enter`: 多行输入
- `Ctrl+C`: 取消输入

---

## II. 🧠 **内存与回忆系统**

### 1. 混合记忆检索 (Agentic Memory Retrieval)
- **BM25 + Embedding 混合搜索**：70% BM25 精确匹配 + 30% BGE-small-zh 语义理解
- **三层上下文组装**：
  1. **被动注入**：用户画像 + 动态情绪维度 → System Prompt
  2. **被动关联**：向量检索相关历史对话 → Context Window
  3. **主动探索**：Agent 通过 Tool Calling 查询日记、技能等深层记忆

### 2. 对话历史存储
- **Markdown 格式**：人类可读，支持 Git 版本控制
- **向量化索引**：每轮对话自动添加到 Chroma 向量库
- **存储路径**：`~/.huaqi/memory/conversations/YYYY/MM/`
- **会话索引**：`sessions_index.yaml` 记录会话元数据

### 3. 日记系统
- **Markdown 原生格式**：支持自定义前缀（情绪、标签、能量、焦虑等）
- **批量导入**：`/diary import <路径>` 支持从文件/目录导入
- **搜索与标签**：关键词搜索、标签过滤
- **AI 自动分析**：自动提取洞察
- **存储路径**：`~/.huaqi/memory/diary/YYYY-MM/YYYY-MM-DD.md`

### 4. 检索工具 (Tools)
- **`search_diary_tool`**：通过关键词在日记库中精准搜索
- **`search_events_tool`**：查询本地 SQLite 中的系统交互事件历史
- 工具通过 `.bind_tools()` 绑定给 LLM，支持流式输出与工具调用共存

---

## III. 👤 **用户画像与性格系统**

### 1. 用户画像 (User Profile)
- **身份字段**：姓名、昵称、出生日期、位置、职位、公司
- **背景字段**（列表）：技能、爱好、人生目标、价值观
- **AI 自动提取**：后台异步线程分析日记、对话，自动提取与更新
  - 支持 3 次重试机制，超时 30s
  - 失败时降级到规则匹配
- **命令**：`huaqi profile show` / `huaqi profile set <field> <value>`

### 2. 人格画像与演化 (Personality Engine)
- **Big Five 人格模型**：外向性、尽责性、开放性、宜人性、神经质
- **沟通风格**：陪伴者、导师、朋友等角色定位
- **自动演化**：基于用户日记与对话历史自动更新
- **人工审核机制**：`huaqi personality review` 批准或拒绝更新提案
- **命令**：`huaqi personality show` / `huaqi personality update`

### 3. 用户理解维度分析 (Adaptive Understanding)
- **4 大维度**：
  1. 情绪 (Emotion): 极度消极 ~ 极度积极
  2. 能量 (Energy): 极度疲倦 ~ 精力充沛
  3. 焦虑 (Anxiety): 极度焦虑 ~ 完全放松
  4. 动机 (Motivation): 极度消极 ~ 强烈动机
- **实时计算**：每轮对话在 `user_analyzer` 节点自动分析
- **存储**：写入 workflow_data，影响后续回复方式

---

## IV. 📈 **成长追踪系统**

### 1. 技能追踪
- **技能库**：管理正在学习的技能列表
- **练习记录**：记录每次练习时间（小时制）
- **进度可视化**：累计练习时间、频率分析
- **命令**：`/skill <名称>` / `/log <技能> <小时>` / `/skills`

### 2. 目标管理
- **短期/长期目标**：灵活的目标管理
- **进度追踪**：完成度、里程碑跟踪
- **命令**：`/goal <标题>` / `/goals`

### 3. 模式学习与周报 (Pattern Learning)
- **模式识别**：
  - 情绪周期（星期几、时间段的情绪规律）
  - 话题趋势（持续关注、新兴话题）
  - 行为习惯（活跃时段、交互频率、记录规律）
- **周报生成**：每周自动生成，包含洞察和建议
- **命令**：`/report` / `/report weekly` / `/report insights`

---

## V. 🚀 **内容流水线** (Content Pipeline)

### 1. 数据源 (Sources)
- **X/Twitter 集成**：从 X API 抓取用户关注内容
- **RSS 订阅**：支持多源 RSS Feed 采集
- **可扩展**：插件化 BaseSource 架构

### 2. 处理器 (Processors)
- **摘要生成**：`Summarizer` 浓缩长文本
- **翻译**：`Translator` 多语言支持
- **小红书格式化**：`XiaoHongShuFormatter` 自动优化文案、添加标签、表情符号
- **流程**：采集 → 摘要 → 翻译 → 格式化 → 发布

### 3. 发布平台 (Publishers)
- **小红书**：支持小红书 API 发布（需配置）
- **可扩展**：插件化 BasePublisher 架构

### 4. 命令与工作流
```bash
huaqi pipeline show              # 查看流水线状态
huaqi pipeline run               # 执行完整流水线
huaqi pipeline run --dry-run     # 预览模式
huaqi pipeline run --source x    # 仅抓取 X
huaqi pipeline drafts            # 查看草稿列表
huaqi pipeline review            # 人工审核界面
```

### 5. 草稿与审核机制
- **自动保存草稿**：发布前保存到 `~/.huaqi/drafts/`
- **人工审核**：`huaqi pipeline review` 批准或拒绝
- **分析内容**：包含原始来源、处理历史、建议标签

---

## VI. ⏰ **定时任务系统** (APScheduler Integration)

### 1. 调度器配置
- **基于 APScheduler >= 4.0.0a**
- **持久化存储**：任务保存到 SQLite (`~/.huaqi/scheduler.db`)
- **三种触发器**：
  - Cron: `0 9 * * MON` (每周一早上9点)
  - Interval: `hours=1` (每小时)
  - Date: 一次性任务

### 2. 内置任务
- **周报生成**：每周一次自动分析
- **流水线执行**：定期采集内容并生成草稿
- **关怀检查**：主动询问用户状态
- **命令**：`huaqi daemon start/stop/status/list`

### 3. 自定义钩子 (Hooks)
- 支持注册自定义回调函数
- 支持前置/后置钩子

---

## VII. 🔧 **配置管理系统**

### 1. 配置存储
- **YAML 格式**：`~/.huaqi/config.yaml`
- **支持的配置**：
  - `llm_default_provider`: OpenAI/Claude/Deepseek
  - `data_dir`: 数据目录路径
  - `memory.max_session_memory`: 最大会话记忆数
  - `git.remote_url`: Git 远程地址
  - `modules.network_proxy`: 网络代理模块开关
  - `modules.wechat`: 微信模块开关

### 2. 配置管理命令
```bash
huaqi config show                    # 查看所有配置
huaqi config set <KEY>               # 交互向导设置
huaqi config set <KEY> <VALUE>       # 单行设置
huaqi config set llm                 # LLM 配置向导
huaqi config set data_dir            # 修改数据目录（支持迁移）
```

### 3. 热重载机制
- **ConfigManager**：支持运行时重新加载配置
- **Watchdog**：监听配置文件变更
- **命令**：`huaqi system hot-reload`

### 4. 默认关闭（Opt-in）原则
- **隐私优先**：所有数据采集模块默认关闭
- **显式开启**：通过 CLI 或环境变量 (`HUAQI_ENABLE_NETWORK=1`) 开启
- **持久化记录**：模块开关状态保存到 config.yaml

---

## VIII. 🔐 **隐私与安全** (Core Engine)

### 1. 事件系统与脱敏 (Event Bus)
- **统一事件结构**：`Event` 数据类（timestamp, source, actor, content, context_id）
- **自动脱敏**：在实例化时通过正则表达式自动脱敏 `sk-` 开头的 API 密钥
- **来源追踪**：所有交互事件记录来源、时间戳、参与者

### 2. 本地数据库存储 (LocalDBStorage)
- **SQLite 引擎**：`~/.huaqi/events.db`
- **功能**：
  - `insert_event()`: 插入事件记录
  - `get_recent_events()`: 查询最近事件
  - `search_events()`: 模糊搜索
- **应用场景**：
  - 微信聊天记录采集（需显式开启）
  - CLI 交互历史记录
  - 系统事件日志

### 3. 隐私控制面板
- **模块开关管理**：`ConfigManager`
- **支持的模块**：
  - `wechat`: 微信数据采集
  - `network_proxy`: 网络请求拦截
- **检查机制**：启动时自动读取 config.yaml 中的模块配置

---

## IX. 🔗 **集成与扩展**

### 1. 微信 Webhook 集成
- **自动采集**：微信消息自动转化为事件
- **文件**：`huaqi_src/integrations/wechat_webhook.py`
- **需显式开启**：`modules.wechat: true`

### 2. LLM 多提供商支持
- **OpenAI**: GPT-4, GPT-3.5-turbo
- **Claude**: Anthropic Claude 系列
- **Deepseek**: 本地或托管
- **配置方式**：`huaqi config set llm`

### 3. 向量模型支持
- **BGE-small-zh**: 默认中文嵌入模型
- **Sentence Transformers**: 通过 `EmbeddingService` 支持
- **可替换**：配置文件指定自定义模型

### 4. Git 数据同步
- **自动提交**：每次数据变更自动提交到本地 Git
- **远程推送**：支持配置远程仓库 URL，自动推送
- **命令**：`huaqi config set git`

---

## X. 📊 **数据存储架构**

### 1. 数据目录结构
```
~/.huaqi/
├── config.yaml                    # 应用配置
├── personality.yaml               # AI 人格配置
├── growth.yaml                    # 技能与目标
├── sessions_index.yaml            # 会话索引
├── events.db                      # 事件数据库 (SQLite)
├── scheduler.db                   # 定时任务数据库 (SQLite)
├── checkpoints.db                 # LangGraph 检查点 (AsyncSqliteSaver)
├── memory/
│   ├── conversations/YYYY/MM/     # 对话历史 Markdown
│   └── diary/YYYY-MM/             # 日记 Markdown
├── drafts/                        # 内容流水线草稿
└── vector_db/                     # Chroma 向量数据库
```

### 2. 存储格式
- **文本数据**：Markdown（对话、日记）
- **配置数据**：YAML（配置、画像）
- **结构化数据**：SQLite（事件、任务）
- **向量数据**：Chroma（对话向量、日记向量）

---

## XI. 🎯 **系统管理命令**

### 1. 状态查询
```bash
huaqi status                       # 查看系统状态（技能、目标、人格）
huaqi system show                  # 详细系统状态（数据目录、Git 状态）
```

### 2. 数据迁移
```bash
huaqi system migrate               # 执行数据迁移（v3→v4）
huaqi system migrate --dry-run     # 预览迁移内容
huaqi system migrate --skip-backup # 跳过备份（不推荐）
```

### 3. 首次配置向导
- **自动触发**：首次运行 `huaqi` 时自动引导设置数据目录
- **交互式**：提示设置 LLM、Git 等关键配置

---

## XII. 📚 **依赖技术栈**

### 核心框架
- **LangGraph (>= 0.2.0)**: Agent 工作流编排
- **LangChain (>= 0.3.0)**: LLM 集成框架
- **Typer (>= 0.9.0)**: CLI 框架
- **Pydantic (>= 2.0.0)**: 数据验证

### 对话与输出
- **OpenAI API**: LLM 调用
- **Prompt Toolkit (>= 3.0.0)**: 交互式终端
- **Rich (>= 13.0.0)**: 富文本输出与格式化

### 存储与检索
- **Chroma (>= 0.5.0)**: 向量数据库
- **Sentence-Transformers (>= 3.0.0)**: 向量嵌入
- **Rank-BM25 (>= 0.2.0)**: BM25 文本搜索
- **SQLite**: 本地关系型数据库
- **GitPython (>= 3.1.0)**: Git 集成

### 任务与调度
- **APScheduler (>= 4.0.0a)**: 定时任务
- **Watchdog (>= 4.0.0)**: 文件监听（热重载）

### 数据处理
- **PyYAML (>= 6.0)**: YAML 序列化
- **Feedparser (>= 6.0.0)**: RSS 解析
- **aiohttp (>= 3.9.0)**: 异步 HTTP 请求
- **Structlog (>= 24.0.0)**: 结构化日志

---

## XIII. 🚀 **发展路线图** (已完成的阶段)

| Phase | 功能 | 状态 |
|-------|------|------|
| P1 | 基础对话系统 | ✅ |
| P2 | 记忆系统 (日记 + 对话历史) | ✅ |
| P3 | 技能追踪与目标管理 | ✅ |
| P4 | APScheduler 定时任务 | ✅ |
| P5 | 内容流水线 (X/RSS → 小红书) | ✅ |
| P6 | 人机协同中断恢复 | ✅ |
| P7 | 数据隔离与用户管理 | ✅ |
| P8 | 配置热重载与数据迁移 | ✅ |
| P9 | 核心分析引擎与用户画像系统 | ✅ |
| P10 | 模式学习与主动关怀系统 | ✅ |

---

## XIV. 📌 **核心创新点总结**

1. **Agent 为主的对话架构**：从传统 while 循环升级到 LangGraph StateGraph，支持人机协同与中断恢复
2. **混合记忆检索**：BM25 + Embedding 的智能组合，确保既能精确匹配又能语义理解
3. **多层上下文组装**：被动注入 + 被动关联 + 主动探索的三层机制
4. **隐私优先设计**：本地存储 + 自动脱敏 + 默认关闭原则
5. **内容流水线**：从多源采集、智能处理到人工审核的完整闭环
6. **自适应性格演化**：AI 人格根据用户数据动态调整，提升陪伴体验
7. **模式识别与周报**：长期数据分析生成个性化洞察和建议

---

**文档更新时间**: 2026-03-30  
**项目版本**: v0.2.0  
**总功能模块数**: 14+ 大类，100+ 细粒度功能
