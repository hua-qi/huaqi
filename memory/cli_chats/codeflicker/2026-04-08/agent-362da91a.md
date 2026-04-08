---
session_id: agent-362da91a
date: 2026-04-08
time_start: 2026-04-08T07:24:43.802Z
time_end: 2026-04-08T07:27:03.981Z
project: users-lianzimeng
git_branch: 
---

[2026-04-08 15:24:43] [user]: 深度分析 /Users/lianzimeng/workspace/claude-code/src/ 目录，我需要回答以下4个核心问题：

1. **代码规范参考**：是否有参考代码规范？代码规范从哪里来？
   - 重点查找：systemPrompt 构造逻辑、CLAUDE.md / .claude 文件加载逻辑、memdir 目录下的内容、constants/prompts.ts、constants/system.ts、constants/systemPromptSections.ts
   - 查找是否有读取项目 eslint/tsconfig/prettier 配置的逻辑

2. **语法错误保证**：如何保证代码没有语法错误？
   - 重点查找：工具执行后是否有 lint/typecheck 步骤、hooks.ts、query/stopHooks.ts、是否有自动运行 linter 的逻辑

3. **业务逻辑验证**：如何保证业务逻辑没问题？
   - 重点查找：是否有测试执行逻辑、verification agent、是否有运行测试的工具调用模式

4. **验收动作/工具**：有验收动作或工具吗？
   - 重点查找：verification 相关、review 相关命令（commands/review.ts）、init-verifiers.ts、AgentTool 中的 verification 类型

请详细阅读以下文件：
- src/constants/systemPromptSections.ts
- src/constants/prompts.ts  
- src/constants/system.ts
- src/query/stopHooks.ts
- src/schemas/hooks.ts
- src/commands/review.ts
- src/commands/init-verifiers.ts
- src/memdir/memdir.ts
- src/utils/systemPrompt.ts
- src/tools/ 目录下与 verification/lint/test 相关的文件

返回每个问题的详细答案，包括关键代码片段和文件路径。
[2026-04-08 15:24:53] [assistant]: 我会进行深度分析。让我系统地检查这个目录结构和关键文件。
[2026-04-08 15:24:58] [assistant]: 现在让我深入查看关键文件：
[2026-04-08 15:25:15] [assistant]: 现在让我查找工具相关的文件，尤其是验证和linting相关的：
[2026-04-08 15:25:23] [assistant]: 现在让我查看prompts.ts文件的关键部分，以及检查CLAUDE.md/claude相关的加载逻辑：
[2026-04-08 15:25:39] [assistant]: 现在让我查找关于hooks和lint相关的文件：
[2026-04-08 15:25:43] [assistant]: 现在让我查看关于CLAUDE.md的加载位置以及获取规范相关信息：
[2026-04-08 15:25:56] [assistant]: 现在让我创建一个总结。首先让我检查更多关于hooks和verify的细节：
[2026-04-08 15:26:00] [assistant]: 完美。现在我有足够的信息来创建详细的分析报告。让我编写完整答案：
[2026-04-08 15:27:03] [assistant]: 根据我对 `/Users/lianzimeng/workspace/claude-code/src/` 目录的深度分析，以下是对 4 个核心问题的详细答案：

---

## 1. 代码规范参考：规范从哪里来？

### 关键文件位置
- **`src/utils/claudemd.ts`** - CLAUDE.md 加载核心逻辑
- **`src/utils/markdownConfigLoader.ts`** - Markdown 配置文件加载器
- **`src/constants/systemPromptSections.ts`** - 系统提示部分定义
- **`src/utils/systemPrompt.ts`** - 系统提示构造逻辑

### 规范加载流程（优先级顺序）

```
1. Managed memory (管理员级) - /etc/claude-code/CLAUDE.md
   ↓
2. User memory (用户级) - ~/.claude/CLAUDE.md
   ↓
3. Project memory (项目级) - CLAUDE.md / .claude/CLAUDE.md / .claude/rules/*.md
   ↓
4. Local memory (本地私有) - CLAUDE.local.md
```

**文件路径代码片段** (src/utils/claudemd.ts, L1-30)：
```typescript
/**
 * Files are loaded in the following order:
 *
 * 1. Managed memory (eg. /etc/claude-code/CLAUDE.md) - Global instructions for all users
 * 2. User memory (~/.claude/CLAUDE.md) - Private global instructions for all projects
 * 3. Project memory (CLAUDE.md, .claude/CLAUDE.md, and .claude/rules/*.md in project roots)
 * 4. Local memory (CLAUDE.local.md in project roots) - Private project-specific instructions
 */
```

### 系统提示构造逻辑

**文件**：`src/utils/systemPrompt.ts` (L20-124)

```typescript
export function buildEffectiveSystemPrompt({
  mainThreadAgentDefinition,
  toolUseContext,
  customSystemPrompt,
  defaultSystemPrompt,
  appendSystemPrompt,
  overrideSystemPrompt,
}): SystemPrompt {
  // 优先级顺序：
  // 0. Override system prompt (覆盖所有其他提示)
  // 1. Coordinator system prompt (协调器模式)
  // 2. Agent system prompt (代理提示，proactive 模式下追加)
  // 3. Custom system prompt (自定义系统提示)
  // 4. Default system prompt (默认提示)
  // + appendSystemPrompt (总是追加在末尾)
}
```

### 提示部分缓存管理

**文件**：`src/constants/systemPromptSections.ts`

```typescript
/**
 * Create a memoized system prompt section.
 * Computed once, cached until /clear or /compact.
 */
export function systemPromptSection(
  name: string,
  compute: ComputeFn,
): SystemPromptSection {
  return { name, compute, cacheBreak: false }
}

/**
 * Create a volatile system prompt section that recomputes every turn.
 * This WILL break the prompt cache when the value changes.
 */
export function DANGEROUS_uncachedSystemPromptSection(
  name: string,
  compute: ComputeFn,
  _reason: string,
): SystemPromptSection {
  return { name, compute, cacheBreak: true }
}
```

### memdir 内存系统

**文件**：`src/memdir/memdir.ts`

- **Auto Memory**: 持久化内存，用户特定项目
- **Team Memory**: 跨组织共享内存 (feature-gated 'TEAMMEM')
- **MEMORY.md 索引**: 最多 200 行或 25KB，超限时截断并警告
- **内存类型**：user / feedback / project / reference（四类闭集）

```typescript
export const ENTRYPOINT_NAME = 'MEMORY.md'
export const MAX_ENTRYPOINT_LINES = 200
export const MAX_ENTRYPOINT_BYTES = 25_000
```

### 项目级规范发现

**文件**：`src/utils/markdownConfigLoader.ts`

```typescript
/**
 * Traverses from the current directory up to the git root,
 * collecting all .claude directories along the way.
 *
 * Stopping at git root prevents commands/skills from parent directories
 * outside the repository from leaking into projects.
 */
export function getProjectDirsUpToHome(
  subdir: ClaudeConfigDirectory,  // 'commands'|'agents'|'output-styles'|'skills'
  cwd: string,
): string[] {
  // 从 CWD 向上遍历到 git root
}
```

**关键概念**：`.claude/` 目录中的子目录：
- `.claude/commands/` - 斜杠命令
- `.claude/agents/` - 自定义代理
- `.claude/skills/` - 验证技能（包含 "verifier" 的文件夹被自动发现）
- `.claude/rules/*.md` - 条件规则（按文件路径匹配）

---

## 2. 语法错误保证：如何保证代码没有语法错误？

### 核心验证机制：Verification Agent

**文件**：`src/tools/AgentTool/built-in/verificationAgent.ts`

```typescript
export const VERIFICATION_AGENT: BuiltInAgentDefinition = {
  agentType: 'verification',
  whenToUse: '使用此代理验证实现工作是否正确',
  color: 'red',
  background: true,
  disallowedTools: [
    AGENT_TOOL_NAME,
    EXIT_PLAN_MODE_TOOL_NAME,
    FILE_EDIT_TOOL_NAME,
    FILE_WRITE_TOOL_NAME,
    NOTEBOOK_EDIT_TOOL_NAME,  // 禁止修改代码
  ],
  getSystemPrompt: () => VERIFICATION_SYSTEM_PROMPT,
}
```

### 验证代理的必需步骤（通用基线）

**系统提示**（src/tools/AgentTool/built-in/verificationAgent.ts, L25-48）：

```typescript
=== REQUIRED STEPS (universal baseline) ===
1. Read the project's CLAUDE.md / README for build/test commands
2. Run the build (if applicable). A broken build is an automatic FAIL.
3. Run the project's test suite (if it has one). Failing tests are an automatic FAIL.
4. Run linters/type-checkers if configured (eslint, tsc, mypy, etc.).
5. Check for regressions in related code.
```

### Stop Hooks - 每轮结束执行验证

**文件**：`src/query/stopHooks.ts`

```typescript
export async function* handleStopHooks(
  messagesForQuery: Message[],
  assistantMessages: AssistantMessage[],
  systemPrompt: SystemPrompt,
  // ...
): AsyncGenerator<...> {
  // Stop hooks 在每次 AI 回复后执行
  // 1. 执行自动梦想 (auto-dream)
  // 2. 提取内存 (extract-memories)
  // 3. 执行提示建议 (prompt suggestion)
  // 4. 运行模板作业分类器
  
  const generator = executeStopHooks(
    permissionMode,
    toolUseContext.abortController.signal,
    undefined,
    stopHookActive ?? false,
    toolUseContext.agentId,
    toolUseContext,
    [...messagesForQuery, ...assistantMessages],
  )
}
```

### Hook 执行位置

**文件**：`src/utils/hooks.ts` (L3639+)

```typescript
/**
 * Execute all Stop hooks asynchronously.
 * Stop hooks are shell commands/prompts/agents that run after the model
 * completes. They can read files, run tests, check for regressions.
 * 
 * Hooks can be defined in:
 * - .claude/hooks.json (project-level)
 * - ~/.claude/hooks.json (user-level)
 * - Managed settings (organization-level)
 */
export async function* executeStopHooks(
  permissionMode?: string,
  signal?: AbortSignal,
  timeoutMs: number = TOOL_HOOK_EXECUTION_TIMEOUT_MS,
  stopHookActive: boolean = false,
  subagentId?: AgentId,
  toolUseContext?: ToolUseContext,
  messages?: Message[],
  agentType?: string,
): AsyncGenerator<...> {
  // 执行所有已注册的停止 hooks
}
```

### Hook 脚本（shell 命令）

**文件**：`src/schemas/hooks.ts`

支持的 Hook 类型：
1. **command** - Shell 命令 (bash/powershell)
   ```json
   {
     "type": "command",
     "command": "npm run lint && npm run test",
     "if": "Bash(npm run *)",
     "timeout": 60,
     "async": false
   }
   ```

2. **prompt** - LLM 评估
   ```json
   {
     "type": "prompt",
     "prompt": "Does the code pass linting?",
     "if": "Write(*)",
     "model": "claude-sonnet-4-6"
   }
   ```

3. **agent** - 代理验证
   ```json
   {
     "type": "agent",
     "prompt": "Verify that tests pass",
     "if": "FileEdit(*)",
     "timeout": 60,
     "model": "claude-haiku-4-6"
   }
   ```

4. **http** - HTTP 请求
   ```json
   {
     "type": "http",
     "url": "https://api.example.com/verify",
     "if": "FileWrite(api/*)"
   }
   ```

### Hook 配置文件位置

**文件**：`src/utils/hooks/hooksConfigManager.ts` 和 `src/schemas/hooks.ts`

```typescript
// Hook 配置可以在这些位置定义：
// 1. ~/.claude/hooks.json (用户级)
// 2. .claude/hooks.json (项目级)
// 3. /etc/claude-code/hooks.json (管理员级)

// 使用 if 条件过滤执行
// 格式：ToolName(pattern)
// 例如："Bash(npm:*)" 仅在 npm 命令时执行
```

### 自动 Lint 检查（项目检测）

**验证代理的智能检测**（系统提示，L33-50）：

```typescript
=== VERIFICATION STRATEGY ===
Adapt your strategy based on what was changed:

**Frontend changes**: Start dev server → check browser automation tools
**Backend/API changes**: Start server → curl/fetch endpoints
**CLI/script changes**: Run with representative inputs
**Infrastructure/config changes**: Validate syntax → dry-run
**Library/package changes**: Build → full test suite
**Bug fixes**: Reproduce the original bug → verify fix → regression tests
**Refactoring**: Existing test suite MUST pass unchanged
```

验证代理会自动：
1. 检测项目的 build/test 脚本（package.json、Makefile 等）
2. 运行 eslint、tsc、mypy 等工具
3. 执行测试套件
4. 验证输出和边界条件

---

## 3. 业务逻辑验证：如何保证业务逻辑没问题？

### /init-verifiers 命令 - 自动化验证技能生成

**文件**：`src/commands/init-verifiers.ts`

这个命令引导用户为不同类型的应用创建验证技能：

**Phase 1: 自动检测**
```typescript
1. Scan top-level directories
   - 查找 package.json, Cargo.toml, pyproject.toml, go.mod
   - 识别不同的项目区域和应用类型
   
2. For each area, detect:
   a. Project type and stack (语言, 框架, 包管理器)
   b. Application type (Web app, CLI, API service)
   c. Existing verification tools (Jest, pytest, Playwright, etc.)
   d. Dev server configuration (启动方式, URL, 就绪信号)
   e. Installed verification packages
```

**Phase 2: 验证工具设置**
- Web 应用：Playwright / Chrome DevTools MCP / Claude Chrome Extension
- CLI 工具：Tmux / asciinema
- API 服务：curl / httpie

**Phase 3: 交互式问答**
- 生成器名称：`verifier-<project>-<type>`
- 项目特定问题（dev server 命令、URL、认证）

**Phase 4: 生成验证技能**
```yaml
# .claude/skills/<verifier-name>/SKILL.md
name: verifier-frontend-playwright
description: Verify frontend UI changes
allowed-tools:
  - Bash(npm:*)
  - mcp__playwright__*
  - Read
  - Glob
```

**Phase 5: 确认创建**
- 技能创建位置：`.claude/skills/` （自动发现）
- 文件夹名必须包含 "verifier"（大小写不敏感）

### Verification Agent 的实际工作流程

**入口文件**：`src/commands/init-verifiers.ts` 和验证代理

验证代理运行的命令示例：
```bash
# 构建
npm run build

# 运行测试
npm test

# Linting
npm run lint
eslint .
tsc --noEmit

# Playwright 验证（Web UI）
npx playwright test

# API 验证
curl -X GET http://localhost:3000/api/endpoint
```

### 业务逻辑验证的核心原则

**验证代理系统提示**（src/tools/AgentTool/built-in/verificationAgent.ts, L3-20）：

```typescript
const VERIFICATION_SYSTEM_PROMPT = `
You are a verification specialist. Your job is not to confirm the 
implementation works — it's to try to break it.

=== CRITICAL: DO NOT MODIFY THE PROJECT ===
You are STRICTLY PROHIBITED from:
- Creating, modifying, or deleting any files IN THE PROJECT DIRECTORY
- Installing dependencies or packages
- Running git write operations

You MAY write ephemeral test scripts to /tmp via Bash redirection.

=== BEFORE ISSUING PASS ===
Your report must include at least one adversarial probe:
- **Concurrency**: parallel requests to create-if-not-exists paths
- **Boundary values**: 0, -1, empty string, very long strings, unicode, MAX_INT
- **Idempotency**: same mutating request twice
- **Orphan operations**: delete/reference IDs that don't exist

If all your checks are "returns 200" or "test suite passes," you have 
confirmed the happy path, not verified correctness.
```

### Verification 输出格式（必需）

```markdown
### Check: [what you're verifying]
**Command run:**
  [exact command executed]
**Output observed:**
  [actual terminal output — copy-paste, not paraphrased]
**Result: PASS** (or FAIL or PARTIAL)

VERDICT: PASS
or
VERDICT: FAIL
or
VERDICT: PARTIAL
```

### 验证代理禁用的工具

```typescript
disallowedTools: [
  AGENT_TOOL_NAME,                    // 禁止嵌套代理
  EXIT_PLAN_MODE_TOOL_NAME,           // 禁止退出计划模式
  FILE_EDIT_TOOL_NAME,                // 禁止编辑文件
  FILE_WRITE_TOOL_NAME,               // 禁止写文件
  NOTEBOOK_EDIT_TOOL_NAME,            // 禁止编辑笔记本
]
```

---

## 4. 验收动作/工具：有验收动作或工具吗？

### Review 命令

**文件**：`src/commands/review.ts`

```typescript
const review: Command = {
  type: 'prompt',
  name: 'review',
  description: 'Review a pull request',
  progressMessage: 'reviewing pull request',
  source: 'builtin',
  async getPromptForCommand(args): Promise<ContentBlockParam[]> {
    return [{ type: 'text', text: LOCAL_REVIEW_PROMPT(args) }]
  },
}

// /ultrareview 是远程 bughunter 路径（需要 Claude Code on the web）
const ultrareview: Command = {
  type: 'local-jsx',
  name: 'ultrareview',
  description: '~10–20 min · Finds and verifies bugs in your branch',
  isEnabled: () => isUltrareviewEnabled(),
}
```

### Task 系统 - 进度跟踪

**工具文件**：
- `src/tools/TaskCreateTool/` - 创建任务
- `src/tools/TaskUpdateTool/` - 更新任务状态
- `src/tools/TaskListTool/` - 列出任务
- `src/tools/TaskGetTool/` - 获取任务详情
- `src/tools/TaskStopTool/` - 停止任务

### TodoWrite 工具 - 计划跟踪

**工具文件**：`src/tools/TodoWriteTool/TodoWriteTool.ts`

```typescript
// 用于多步骤任务进度跟踪
// 例如：init-verifiers 命令中的多阶段过程
{
  "todos": [
    {
      "id": "1",
      "content": "Scan project directories",
      "status": "completed",
      "priority": "high"
    },
    {
      "id": "2", 
      "content": "Detect verification tools",
      "status": "in_progress",
      "priority": "high"
    }
  ]
}
```

### Team/Teammate 验证 Hooks

**文件**：`src/query/stopHooks.ts` (L216-334)

```typescript
// Teammate 特定的验证：
if (isTeammate()) {
  // 1. Run TaskCompleted hooks for in-progress tasks
  const inProgressTasks = tasks.filter(
    t => t.status === 'in_progress' && t.owner === teammateName,
  )
  
  for (const task of inProgressTasks) {
    const taskCompletedGenerator = executeTaskCompletedHooks(
      task.id,
      task.subject,
      task.description,
      teammateName,
      teamName,
      permissionMode,
      signal,
      undefined,
      toolUseContext,
    )
  }
  
  // 2. Run TeammateIdle hooks
  const teammateIdleGenerator = executeTeammateIdleHooks(
    teammateName,
    teamName,
    permissionMode,
    signal,
  )
}
```

### Hooks 系统中的验收/验证 Hook 事件

**文件**：`src/types/hooks.ts` 和 `src/entrypoints/agentSdkTypes.ts`

支持的 Hook 事件：
```typescript
// Pre-execution hooks (before tool use)
'PreToolUse'

// Post-execution hooks (after tool use)
'PostToolUse'
'PostToolUseFailure'

// Turn completion hooks
'Stop'              // 每轮结束时执行
'TaskCompleted'     // 任务完成时
'TeammateIdle'      // 队友空闲时

// Session lifecycle hooks
'SessionStart'
'SessionEnd'
'SubagentStart'
'SubagentStop'

// Other events
'PermissionDenied'
'PermissionRequest'
'PreCompact'
'PostCompact'
'SetupHook'
'ConfigChange'
'CwdChanged'
'FileChanged'
'InstructionsLoaded'
'UserPromptSubmit'
'Notification'
'Elicitation'
'ElicitationResult'
```

### Stop Hook 执行流程（验收点）

**文件**：`src/query/stopHooks.ts` (主流程)

```typescript
// 1. 执行常规 Stop hooks
for await (const result of executeStopHooks(...)) {
  if (result.message) {
    yield result.message
    // 跟踪工具 ID、Hook 数量、错误、输出
  }
  
  if (result.blockingError) {
    // 阻塞性错误 - 防止继续
    blockingErrors.push(userMessage)
    hasOutput = true
  }
  
  if (result.preventContinuation) {
    // Hook 要求停止 - 创建附件跟踪
    yield createAttachmentMessage({
      type: 'hook_stopped_continuation',
      message: stopReason,
      hookName: 'Stop',
      hookEvent: 'Stop',
    })
  }
}

// 2. 创建 Hook 摘要系统消息
if (hookCount > 0) {
  yield createStopHookSummaryMessage(
    hookCount,
    hookInfos,        // Hook 命令和提示
    hookErrors,       // 错误列表
    preventedContinuation,
    stopReason,
    hasOutput,
    'suggestion',
    stopHookToolUseID,
  )
}

// 3. 返回结果
return {
  blockingErrors,
  preventContinuation
}
```

### 实际验收流程示例

当运行 `/init-verifiers` 后：

1. **自动检测** → TodoWrite 跟踪进度
2. **用户交互** → AskUserQuestion 确认选择
3. **生成技能文件** → .claude/skills/verifier-*/SKILL.md
4. **自动发现** → Verify 代理查找名称包含 "verifier" 的技能
5. **执行验证** → Verification 代理使用该技能进行验证
6. **Hook 跟踪** → Stop hooks 在每轮结束执行验证命令
7. **报告结果** → VERDICT: PASS/FAIL/PARTIAL

### 关键的 Hook 摘要信息

**文件**：`src/utils/messages.ts`

```typescript
// Hook 摘要包含：
- Hook 总数
- 每个 Hook 的命令和执行时间 (durationMs)
- 错误列表（非阻塞 + 阻塞）
- 是否阻止继续
- 输出是否存在
```

---

## 总结表格

| 问题 | 关键文件 | 核心机制 |
|-----|--------|--------|
| **1. 代码规范** | `claudemd.ts`<br>`markdownConfigLoader.ts`<br>`systemPrompt.ts` | 多层次加载：Managed → User → Project → Local<br>提示部分缓存管理<br>内存系统 (MEMORY.md) |
| **2. 语法错误** | `verificationAgent.ts`<br>`stopHooks.ts`<br>`hooks.ts` | Verification Agent（必需步骤）<br>Stop Hooks（每轮验证）<br>多种 Hook 类型（command/prompt/agent/http） |
| **3. 业务逻辑** | `init-verifiers.ts`<br>`verificationAgent.ts` | 自动化技能生成<br>验证代理测试所有边界情况和并发<br>禁止修改代码，仅验证 |
| **4. 验收动作** | `review.ts`<br>`stopHooks.ts`<br>`TaskCreateTool`<br>`TodoWriteTool` | Review 命令<br>Task 系统进度跟踪<br>Stop/TaskCompleted/TeammateIdle Hooks<br>Hook 摘要报告 |

这个系统形成了一个完整的代码质量保证链：从规范参考、语法检查、业务逻辑验证到最终的验收报告。
