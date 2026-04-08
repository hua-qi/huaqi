---
session_id: aaff681c
date: 2026-04-03
time_start: 2026-04-03T08:26:05.275Z
time_end: 2026-04-03T09:02:18.298Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-03 16:26:05] [user]: 根据 @docs/design/adr/ADR-001-implicit-evolution-architecture.md 和 @docs/designs/2026-04-03-implicit-evolution-design.md 梳理重构后的项目整体流程。
再找出漏洞。目标：确定一个能够完美运行的经验沉淀和召回 mcp
[2026-04-03 16:26:49] [assistant]: ### 重构后的项目整体流程梳理

基于 ADR 和设计文档，重构后的系统采用了**“端侧极简 + 异步处理 + 显式反馈”**的架构，整体流程如下：

1. **端侧检索（同步）**：大模型（Agent）调用 `search_best_practices`（传入 `query`, `session_id` 等），同步获取历史经验及其唯一 ID。
2. **端侧提交（Fire and Forget）**：Agent 完成任务后，调用 `finalize_task`。由于参数被极度精简，Agent 只需传入 `session_id`、自由文本格式的总结 `learnings`，以及它显式判断有效/无效的经验 ID 数组（`adopted_experience_ids`, `rejected_experience_ids`）。MCP Tool 接收后立即向 Agent 返回“成功”状态，释放大模型的工作流。
3. **后台抽取（异步）**：后台异步进程拦截请求，调用内置轻量级 LLM 将 `learnings` 智能拆解并结构化为 `task_description`、`solution_summary` 等标准字段后落库。如果 LLM 抽取崩溃，则触发兜底，将原文存入且标记为 `DRAFT` 状态。
4. **后台反馈与打点（异步）**：后台 Feedback Engine 读取传入的 ID 数组，静默过滤掉大模型幻觉产生的无效 ID 后，针对真实存在的历史经验执行置信度（Confidence）的加权与降权，完成系统的自我进化闭环。

---

### 系统漏洞分析 (Vulnerabilities)

尽管架构显著降低了 Agent 的认知负担并消除了阻塞，但存在以下致命的逻辑漏洞：

1. **漏洞一：零检索场景下的“沉淀阻断”**
   `finalize_task` 将 `session_id` 设为**必填**。如果 Agent 面对一个全新问题，没有调用 `search_best_practices` 而是直接编写代码并解决了问题，它将没有合法的 `session_id` 可以提交。这会导致全新的优秀经验因校验失败而无法沉淀。
2. **漏洞二：异步抽取严重缺乏“任务上下文” (Context Loss)**
   精简入参后，后台 Extractor 只能依靠一句 `learnings` 来提取信息。如果 Agent 提交的复盘极其简略（例如：“改用 v2 接口解决了 403 报错”），后台轻量 LLM 根本无法反推出 `task_description`。这会产生大量没有前置触发条件的“无头经验”，未来根本无法被正确召回。
3. **漏洞三：“DRAFT” 降级数据成为垃圾黑洞**
   异常兜底机制将抽取失败的经验标记为 `DRAFT` 落库。但系统并没有设计重试机制或人工清洗链路。伴随后台轻量 LLM 的偶尔抽风，这些低质量的脏数据会在数据库中无声堆积，污染底层数据并消耗存储。
4. **漏洞四：大模型“阅后即焚”导致的反馈断链**
   将相似度推断彻底废除、完全信任大模型的显式 ID 数组，极度依赖大模型的记忆力。在多轮长对话后，Agent 极易忘记或混淆初始检索返回的经验 ID，导致有效经验得不到加权，系统进化飞轮停滞。

---

### 目标：能够完美运行的经验沉淀 MCP 改进方案

为了打造一个完美的经验沉淀与召回闭环，建议针对上述漏洞进行如下修正：

1. **解耦 Session 依赖**：将 `finalize_task` 的 `session_id` 设为选填，或者在 MCP Server 层面自动追踪对话上下文，无需 Agent 显式传递。
2. **微调精简入参（保留任务描述）**：在 `finalize_task` 中新增（或在 `learnings` 描述中强约束）提供 `task_description`。大模型对自己正在做的任务最清楚，让它提供一句话的 Task 并不构成负担，能极大提升后台结构化落库的质量。
3. **闭环上下文拼接**：在后台异步执行 Extractor 时，利用 `session_id` 去数据库反查 Agent 之前的 Search `query`，将 `query` 与 `learnings` 拼接后一并交给轻量 LLM 进行结构化抽取，弥补上下文断层。
4. **完善 DRAFT 回收机制**：后台引入定时清理或二次聚合进程，对 `DRAFT` 状态的数据进行同类项合并或二次 LLM 抽取重试。如果经过 3 次重试仍失败，则彻底归档或删除。
[2026-04-03 16:30:54] [user]: 1. 将你的修正方案沉淀到相关文档里；
2. 现在系统是如何实现不同系统的专业知识不互相污染的？
[2026-04-03 16:51:52] [assistant]: 1. **修正方案已沉淀**：我已经将针对“会话解耦（选填 `session_id`）”、“新增 `task_description` 以维持上下文”、“后台 `query` 拼接机制”以及“`DRAFT` 兜底重试与垃圾回收”的完善方案，更新到了 `ADR-001-implicit-evolution-architecture.md` 以及 `2026-04-03-implicit-evolution-design.md` 两份设计文档中。

2. **系统防止知识互相污染的实现机制**：
当前系统主要通过 **“项目命名空间 (Project Namespace) 硬隔离”** 来避免多项目的专业知识互相污染：
*   **上下文感知**：系统引入了 `ProjectManager` (`src/project_config.py`)，它会在本地 `~/.xp/` 目录下维护多项目配置，并随时感知当前的工作区属于哪个项目 (`current_project`)。
*   **知识强绑定**：所有被后台 Extractor 抽取并落盘的经验（`Experience` 模型），都会被强制打上一个 `project` 字段（如项目名称或 `default`），使每条知识天然具备项目归属属性。
*   **检索时的硬隔离 (Hard Filtering)**：在召回阶段（`src/domain/search.py`），`SearchService` 会在向量相似度计算之前执行严格的初步过滤：
    ```python
    if not cross_project:
        candidates = [e for e in candidates if e.project == self._project]
    ```
    除非显式开启 `cross_project=True` 跨项目检索，否则系统只会把当前项目下的经验候选集送去进行向量匹配和召回。这从底层阻断了 A 项目的私有框架或特殊解决方式出现在 B 项目的问答中，彻底避免了跨项目知识“幻觉”与污染。
[2026-04-03 16:57:06] [user]: 不要结合当前项目代码，仅就文档分析一下如何实现不同系统的专业知识不互相污染的？
[2026-04-03 16:57:19] [assistant]: 仅从架构设计层面来看，实现不同系统专业知识不互相污染通常依赖以下三种核心机制：
1. **元数据硬过滤 (Metadata Pre-filtering)**：在经验沉淀时强制绑定 `system_id` 或专属 `tags`，在召回阶段（向量匹配前）执行严格的元数据硬过滤，逻辑上阻断跨域召回。
2. **向量索引物理隔离 (Index/Collection Isolation)**：为不同系统或项目创建完全独立的向量集合（Collections）或独立的数据库实例，实现底层存储的物理隔离。
3. **环境感知与动态路由 (Context-Aware Routing)**：MCP 根据当前开发环境自动识别所属系统，将请求动态路由至专属的领域知识服务，同时在 Prompt 层面强制注入系统边界约束，防止大模型自行发散。
[2026-04-03 17:02:04] [user]: 不要结合当前项目代码，仅就 @docs/design/adr/ADR-001-implicit-evolution-architecture.md  和 @docs/designs/2026-04-03-implicit-evolution-design.md 分析一下如何实现不同系统的专业知识不互相污染的？
[2026-04-03 17:02:18] [assistant]: 根据这两份文档的设计，目前系统在架构上**并没有**直接包含“不同系统专业知识不互相污染”的强隔离机制。

如果必须基于现有文档的设计推演，唯一能实现知识隔离的机制是：
**基于大模型显式声明的反馈飞轮淘汰机制**。

1. **依赖 Agent 的上下文判断**：当 A 系统的知识意外被召回给正在开发 B 系统的 Agent 时，由于这套“错误经验”对当前任务无用甚至有坑，Agent 在任务结束调用 `finalize_task` 时，会通过 `rejected_experience_ids` 将这条跨域经验显式提交。
2. **异步降权淘汰**：后台 Feedback Engine 收到该数组后，会执行 Confidence 降权。随着这种“跨界污染”发生多次，该经验在 B 系统的上下文中因为得分极低，将自动沉没，不再被优先召回。

**总结**：由于文档全面废除了复杂的向量匹配推断，转而信任 Agent 的显式反馈，它实质上是用**“达尔文式的自然选择（用脚投票）”**来被动地实现了知识适用范围的收敛，而非物理或逻辑上的主动隔离。
