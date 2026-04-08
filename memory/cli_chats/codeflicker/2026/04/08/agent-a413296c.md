---
session_id: agent-a413296c
date: 2026-04-08
time_start: 2026-04-08T07:03:48.081Z
time_end: 2026-04-08T07:09:09.542Z
project: users-lianzimeng
git_branch: 
---

[2026-04-08 15:03:48] [user]: 请深入分析 /Users/lianzimeng/workspace/claude-code/src/ 目录下的源码，重点找出以下内容：

1. 代码生成的核心流程（主循环、任务编排逻辑）
2. 涉及到哪些 Agent（子 Agent、协调 Agent 等）
3. 涉及到哪些 Tool（工具定义、工具调用方式）
4. Query/Task 的生命周期
5. 关键文件：tools.ts、tools/、tasks.ts、query.ts、coordinator/ 等

请尽量详细，返回：
- 核心流程的调用链
- 所有 Tool 的名称和用途
- Agent 的类型和职责
- 关键数据结构
[2026-04-08 15:03:50] [assistant]: 我会深入分析这个目录下的源码。让我先探索目录结构，然后逐步分析各个关键文件。
[2026-04-08 15:05:17] [assistant]: 让我列出目录结构，获取更清晰的视图：
[2026-04-08 15:05:20] [assistant]: 现在让我查看关键文件。首先检查根级别的核心文件：
[2026-04-08 15:05:24] [assistant]: 现在让我查看 tasks.ts、coordinator/、以及 tools 相关文件：
[2026-04-08 15:05:30] [assistant]: 让我继续查看 coordinator 模式、tools 目录结构以及关键的 query 实现：
[2026-04-08 15:05:33] [assistant]: 现在让我查看核心的 query.ts 和 Agent 工具实现：
[2026-04-08 15:05:40] [assistant]: 现在让我获取核心部分的信息：
[2026-04-08 15:05:43] [assistant]: 让我查看 AgentTool 的实现，这是关键的子 Agent 工具：
[2026-04-08 15:06:16] [assistant]: 现在让我创建一个总结文档。让我搜集更多关键信息：
[2026-04-08 15:06:29] [assistant]: 现在让我获取 tasks 目录的信息：
[2026-04-08 15:08:00] [assistant]: 现在我有足够的信息来创建一份详细的分析。让我生成报告：
[2026-04-08 15:08:40] [assistant]: 让我用另一种方式生成报告：
[2026-04-08 15:09:09] [assistant]: 完美！现在让我创建最终的总结文档：

## 总结

我已经深入分析了 `/Users/lianzimeng/workspace/claude-code/src/` 目录下的源码。以下是详细的分析结果：

---

## **1. 代码生成的核心流程**

### 主循环调用链
```
ask(params) 
  → new QueryEngine(config)
  → engine.submitMessage(prompt)
    → processUserInput()           [处理 slash 命令、模型选择]
    → query(params)                [核心查询函数]
      → queryLoop(params)          [无限迭代循环]
        ├─ 系统提示词构造 (fetchSystemPromptParts)
        ├─ API 调用: claude()
        ├─ 消息流处理: message_start → content_block_* → message_stop
        ├─ 工具执行: runTools() → StreamingToolExecutor
        ├─ 消息持久化: recordTranscript()
        └─ 返回结果或继续循环
```

### 关键类和文件
| 文件 | 行数 | 用途 |
|-----|------|------|
| QueryEngine.ts | 1296 | 查询引擎类（SDK & 无头模式） |
| query.ts | 1729 | 主查询循环（async generator） |
| Tool.ts | 793 | 工具接口定义 |
| tools.ts | 390 | 工具集合和装配 |
| Task.ts | 126 | 任务类型定义 |
| tasks.ts | 40 | 任务工厂函数 |

---

## **2. Agent 类型和职责**

### Agent 工具 (src/tools/AgentTool/)
生成子代理（worker agent）来并行执行任务

**核心文件及行数**:
- agentToolUtils.ts (687 行) - 工具过滤、权限管理
- runAgent.ts (35,768 行) - 代理执行引擎
- forkSubagent.ts (8,678 行) - Fork 代理
- resumeAgent.ts (9,339 行) - 恢复代理
- loadAgentsDir.ts (26,220 行) - 加载自定义代理

**Agent 类型**:
- `generalPurpose` - 通用工人代理
- `verification` - 验证代理
- `explorer` - 探索/研究代理
- `plan` - 计划代理
- `guide` - 指南代理
- 自定义代理

**代理生命周期**:
1. 创建: `SpawnAgent { description, prompt, subagent_type }`
2. 运行: `runAgent()` 创建子 QueryEngine
3. 通知: `<task-notification>` XML 返回结果
4. 继续: `SendMessageTool` 发送后续消息
5. 停止: `TaskStopTool` 中断

### Coordinator Mode (多 Agent 编排)
**文件**: `src/coordinator/coordinatorMode.ts`
- 启用方式: `CLAUDE_CODE_COORDINATOR_MODE=1` 环境变量
- 协调器工具: Agent, SendMessage, TaskStop
- Worker 能力: 受限工具集 (Bash, Read, Edit, MCP, Skills)

---

## **3. 所有 Tool 列表 (50+ 个)**

### 核心工具 (始终启用)
1. **AgentTool** - 生成子代理
2. **TaskOutputTool** - 获取任务输出
3. **BashTool** - Shell 命令
4. **FileReadTool** - 读文件
5. **FileEditTool** - 编辑文件
6. **FileWriteTool** - 创建文件
7. **NotebookEditTool** - Jupyter notebook
8. **WebFetchTool** - 网页获取
9. **TodoWriteTool** - 任务列表
10. **WebSearchTool** - 网络搜索
11. **TaskStopTool** - 停止任务
12. **AskUserQuestionTool** - 交互提问
13. **SkillTool** - 项目技能
14. **EnterPlanModeTool** - 进入计划
15. **BriefTool** - 生成摘要

### 搜索和资源工具
- **GlobTool**, **GrepTool** - 文件搜索
- **ToolSearchTool** - 工具发现
- **ListMcpResourcesTool** - MCP 资源列表
- **ReadMcpResourceTool** - 读取 MCP 资源

### 特殊工具
- **REPLTool** - Jupyter VM (ANT)
- **WorkflowTool** - 工作流脚本
- **WebBrowserTool** - Web 浏览
- **SendMessageTool** - 继续代理
- **ExitPlanModeV2Tool** - 退出计划

### 特性 Gate 工具
SleepTool, CronTools, MonitorTool, WorkflowTool, SnipTool, CtxInspectTool, TerminalCaptureTool, LSPTool, PowerShellTool 等

---

## **4. Query/Task 生命周期**

### Query 消息流
```
UserMessage → processUserInput() 
  → query()/queryLoop()
    → API: claude()
      → message_start
        → content_block_delta (流式文本)
      → message_stop
    → Tool 调用检测
      → runTools() 并行执行
        → tool_result 消息生成
    → 继续循环或终止
  → recordTranscript() 持久化
  → yield SDKMessage
```

### Task 类型 (src/Task.ts)
- `local_bash` - 本地 shell 命令
- `local_agent` - 本地子代理
- `remote_agent` - 远程代理 (CoWork)
- `in_process_teammate` - 进程内队友
- `local_workflow` - 工作流脚本
- `monitor_mcp` - MCP 监控

### Task ID 生成
格式: `[prefix][8-char-random-base36]`
- 前缀: `b` (bash), `a` (agent), `r` (remote), `t` (teammate), `w` (workflow), `m` (mcp)
- 示例: `a7q8x9z1`, `b4f2e9x3`

---

## **5. 关键数据结构**

### ToolUseContext
```typescript
type ToolUseContext = {
  options: { commands, tools, mainLoopModel, mcpClients, ... }
  getAppState(): AppState
  setAppState(f: (prev: AppState) => AppState): void
  abortController: AbortController
  readFileState: FileStateCache
  messages: Message[]
  agentId?: AgentId          // 仅子代理
  discoveredSkillNames?: Set<string>
  loadedNestedMemoryPaths?: Set<string>
  contentReplacementState?: ContentReplacementState
}
```

### Tool 接口关键方法
```typescript
type Tool<Input, Output, P> = {
  name: string
  call(args, context, canUseTool, parentMsg, onProgress): Promise<ToolResult>
  checkPermissions(input, context): Promise<PermissionResult>
  validateInput?(input, context): Promise<ValidationResult>
  isConcurrencySafe(input): boolean
  isReadOnly(input): boolean
  isDestructive?(input): boolean
  inputSchema: Zod
  renderToolUseMessage(input, options): React.ReactNode
  maxResultSizeChars: number
}
```

### ToolPermissionContext
- `mode`: 'default' | 'bypass' | 'always_ask' | 'plan'
- `alwaysAllowRules` / `alwaysDenyRules` / `alwaysAskRules`
- `shouldAvoidPermissionPrompts?` (后台代理)
- `awaitAutomatedChecksBeforeDialog?` (协调器工人)

---

## **6. 工具执行流程**

```
1. validateInput()          [工具特定验证]
2. checkPermissions()       [权限规则匹配]
3. canUseTool()            [交互式提示，如需要]
4. tool.call()             [实际执行]
5. mapToolResultToToolResultBlockParam()
6. 添加到用户消息，继续查询
```

### Agent 执行流程
```
1. AgentTool.call() 创建任务
2. generateTaskId('local_agent')
3. runAgent() 启动子 QueryEngine
4. filterToolsForAgent() 过滤允许的工具
5. 创建隔离的权限上下文
6. 流式消息处理
7. 生成 <task-notification> XML
8. 返回给父级协调器
```

---

## **7. 核心调用链速查**

### SDK 调用链
```
ask(params)
  → new QueryEngine(config)
  → engine.submitMessage(prompt)
    → processUserInput()
    → query(params)
      → queryLoop()
        → claude() [API]
        → runTools()
        → recordTranscript()
    → yield SDKMessage
```

### Agent 递归调用
```
Agent 工具调用
  → runAgent()
    → new QueryEngine(subagentConfig)
    → engine.submitMessage(workerPrompt)
      → query() [同上，但工具被过滤]
    → 生成 <task-notification>
      → 返回给父级
```

---

## **8. Feature Flags**

| Feature | 用途 |
|---------|------|
| COORDINATOR_MODE | 多代理编排 |
| HISTORY_SNIP | 动态消息裁剪 |
| TOKEN_BUDGET | 代币预算追踪 |
| REACTIVE_COMPACT | 流式紧凑化 |
| CONTEXT_COLLAPSE | 上下文优化 |
| WORKFLOW_SCRIPTS | 工作流脚本 |
| MONITOR_TOOL | MCP 监控 |
| WEB_BROWSER_TOOL | Web 浏览 |
| TEMPLATES | 任务分类 |
| BG_SESSIONS | 后台会话 |

---

## **总体架构特点**

Claude Code 是一个**多层 AsyncGenerator 系统**:

1. **最高层 (QueryEngine)** - 管理对话生命周期
2. **中层 (query/queryLoop)** - 迭代调用 API，处理消息和工具
3. **工具层 (Tool 接口)** - 沙箱执行，权限检查
4. **Agent 层 (AgentTool)** - 子代理生成和管理
5. **协调层 (Coordinator Mode)** - 多 Agent 并行编排

**核心特点**:
- 流式响应 (AsyncGenerator)
- 并行工具执行 (StreamingToolExecutor)
- 权限多层检查
- 消息持久化 (支持 --resume)
- 成本追踪 (USD 预算)
- 结构化输出 (JSON schema)
- 50+ 内置工具
- 动态 MCP 工具集成
