---
session_id: agent-a441129d
date: 2026-04-08
time_start: 2026-04-08T07:27:36.211Z
time_end: 2026-04-08T07:28:17.414Z
project: users-lianzimeng
git_branch: 
---

[2026-04-08 15:27:36] [user]: 请读取以下文件的完整内容并返回：
1. /Users/lianzimeng/workspace/claude-code/src/tools/AgentTool/built-in/verificationAgent.ts 或类似路径（搜索包含 verificationAgent 或 VERIFICATION_SYSTEM_PROMPT 的文件）
2. /Users/lianzimeng/workspace/claude-code/src/utils/systemPrompt.ts 的关键部分（系统提示如何组装 CLAUDE.md 内容的部分）
3. /Users/lianzimeng/workspace/claude-code/src/commands/init-verifiers.ts 的核心流程部分

如果路径不准确，请搜索相关文件。返回关键代码片段。
[2026-04-08 15:27:38] [assistant]: 我来帮你搜索和读取这些文件。让我先探索目录结构，然后定位这些文件。
[2026-04-08 15:27:42] [assistant]: 现在让我搜索具体的文件：
[2026-04-08 15:27:45] [assistant]: 很好！我找到了所有文件。现在让我读取它们的完整内容：
[2026-04-08 15:27:51] [assistant]: 完美！我已经成功找到并读取了所有三个文件。现在让我提供更完整的内容，包括其他相关的系统提示文件：
[2026-04-08 15:28:17] [assistant]: 完美！现在我已经收集了所有关键文件。让我为你整理一份完整的总结：

## 文件读取总结

### 1. **verificationAgent.ts** - 验证 Agent 的核心定义
📄 **路径**: `/Users/lianzimeng/workspace/claude-code/src/tools/AgentTool/built-in/verificationAgent.ts`

**关键内容**:
- **VERIFICATION_SYSTEM_PROMPT**: 153行的详细验证策略提示
  - 核心原则：验证专家的工作是"打破"实现而非确认工作
  - 两个已知的失败模式：
    1. 验证回避 - 读代码但不运行测试
    2. 被前80%迷惑 - 看到polished UI就通过，忽略剩余20%的问题
  - **严格限制**：禁止修改项目文件、安装依赖、运行git写操作
  - 可以使用临时脚本到/tmp目录
  
- **验证策略按类型**：
  - 前端: 启动dev服务器→使用浏览器自动化→curl资源子集→运行前端测试
  - 后端/API: 启动服务→curl/fetch端点→验证响应格式→测试错误处理
  - CLI: 运行代表性输入→验证stdout/stderr/exit codes→测试边界条件
  - 基础设施/配置: 验证语法→干运行→检查env vars
  
- **通用基线步骤**（5步）：
  1. 读CLAUDE.md/README
  2. 运行构建
  3. 运行测试套件
  4. 运行linters/类型检查器
  5. 检查回归

- **输出格式要求**：每个检查必须有Command+Output+Result结构，最后以`VERDICT: PASS/FAIL/PARTIAL`结尾

- **Agent定义**:
```typescript
export const VERIFICATION_AGENT: BuiltInAgentDefinition = {
  agentType: 'verification',
  disallowedTools: [AGENT_TOOL_NAME, EXIT_PLAN_MODE_TOOL_NAME, FILE_EDIT_TOOL_NAME, FILE_WRITE_TOOL_NAME, NOTEBOOK_EDIT_TOOL_NAME],
}
```

---

### 2. **systemPrompt.ts** - 系统提示组装逻辑
📄 **路径**: `/Users/lianzimeng/workspace/claude-code/src/utils/systemPrompt.ts`

**关键函数**: `buildEffectiveSystemPrompt()`

**优先级顺序**（高到低）:
```
0. overrideSystemPrompt - 覆盖所有其他提示（如循环模式）
   ↓
1. Coordinator系统提示 - 如果激活coordinator模式
   ↓
2. Agent系统提示 - 如果设置了mainThreadAgentDefinition
   - Proactive模式: 追加到默认提示
   - 普通模式: 替换默认提示
   ↓
3. 自定义系统提示 - 通过--system-prompt指定
   ↓
4. 默认系统提示 - 标准Claude Code提示
   ↓
5. appendSystemPrompt - 总是追加到最后（除非有override）
```

**关键代码逻辑**:
```typescript
// Proactive模式：agent指令追加到默认提示
if (agentSystemPrompt && isProactiveActive) {
  return asSystemPrompt([
    ...defaultSystemPrompt,
    `\n# Custom Agent Instructions\n${agentSystemPrompt}`,
    ...(appendSystemPrompt ? [appendSystemPrompt] : []),
  ])
}

// 普通模式：agent替换默认提示
return asSystemPrompt([
  ...(agentSystemPrompt 
    ? [agentSystemPrompt]
    : customSystemPrompt
      ? [customSystemPrompt]
      : defaultSystemPrompt),
  ...(appendSystemPrompt ? [appendSystemPrompt] : []),
])
```

---

### 3. **init-verifiers.ts** - 验证器初始化命令
📄 **路径**: `/Users/lianzimeng/workspace/claude-code/src/commands/init-verifiers.ts`

**5阶段工作流**:

#### **Phase 1: 自动检测**
- 扫描顶级目录识别不同项目区域
- 检测每个区域的：
  - 项目类型和技术栈
  - 应用类型（Web/CLI/API）
  - 现有验证工具
  - Dev服务器配置

#### **Phase 2: 验证工具设置**
- **Web应用**: 检测/安装Playwright或MCP浏览器工具
- **CLI工具**: 检查asciinema，验证Tmux可用
- **API服务**: 检查curl/httpie可用性

#### **Phase 3: 交互式Q&A**
- 为每个区域确认验证器名称（格式: `verifier-[project]-[type]`）
- 项目特定问题（dev服务器命令、URL、就绪信号等）
- 身份验证需求（表单登录/API令牌/OAuth）

#### **Phase 4: 生成验证器技能**
- 位置: `.claude/skills/<verifier-name>/SKILL.md`
- 模板结构：
  ```markdown
  ---
  name: <verifier-name>
  description: <description>
  allowed-tools: [...]
  ---
  
  # <Verifier Title>
  ## Project Context
  ## Setup Instructions
  ## Authentication (if needed)
  ## Reporting
  ## Cleanup
  ## Self-Update
  ```

- **allowed-tools按类型**:
  - **verifier-playwright**: Bash(npm/yarn/pnpm/bun), mcp__playwright__*, Read, Glob, Grep
  - **verifier-cli**: Tmux, Bash(asciinema), Read, Glob, Grep
  - **verifier-api**: Bash(curl/http), Bash(npm/yarn), Read, Glob, Grep

#### **Phase 5: 确认创建**
- 报告技能位置和发现机制
- 说明"verifier"在文件夹名称中用于自动发现

---

### 4. **systemPromptSections.ts** - 系统提示分段机制
📄 **路径**: `/Users/lianzimeng/workspace/claude-code/src/constants/systemPromptSections.ts`

**核心概念**:
- `systemPromptSection()`: 创建缓存的系统提示分段（单次计算，缓存到/clear）
- `DANGEROUS_uncachedSystemPromptSection()`: 创建易失性分段，每次重新计算（破坏提示缓存）
- `resolveSystemPromptSections()`: 解析所有分段并返回提示字符串
- `clearSystemPromptSections()`: 清除所有分段状态（在/clear和/compact上调用）

---

### 5. **systemPromptType.ts** - 系统提示类型
📄 **路径**: `/Users/lianzimeng/workspace/claude-code/src/utils/systemPromptType.ts`

**品牌化类型定义**:
```typescript
export type SystemPrompt = readonly string[] & {
  readonly __brand: 'SystemPrompt'
}

export function asSystemPrompt(value: readonly string[]): SystemPrompt {
  return value as SystemPrompt
}
```
- 目的：防止循环初始化依赖问题
- 字符串数组的品牌化包装，用于类型安全

---

### 6. **builtInAgents.ts** - 内置Agent注册
📄 **路径**: `/Users/lianzimeng/workspace/claude-code/src/tools/AgentTool/builtInAgents.ts`

**Agent列表**:
- GENERAL_PURPOSE_AGENT
- STATUSLINE_SETUP_AGENT
- EXPLORE_AGENT (条件启用)
- PLAN_AGENT (条件启用)
- CLAUDE_CODE_GUIDE_AGENT (非SDK入口点)
- **VERIFICATION_AGENT** (Feature: `tengu_hive_evidence`)

**条件加载逻辑**:
```typescript
if (feature('VERIFICATION_AGENT') && 
    getFeatureValue_CACHED_MAY_BE_STALE('tengu_hive_evidence', false)) {
  agents.push(VERIFICATION_AGENT)
}
```

---

## 核心工作流总结

```
用户任务
  ↓
buildEffectiveSystemPrompt() 组装提示
  ├─ 优先级: override > coordinator > agent > custom > default > append
  └─ Proactive模式特殊处理：agent追加而非替换
  ↓
[如需验证]
  ↓
init-verifiers.ts 创建验证器
  ├─ Phase 1: 自动检测项目类型
  ├─ Phase 2: 设置验证工具（Playwright/Tmux/curl）
  ├─ Phase 3: 交互式Q&A
  ├─ Phase 4: 生成SKILL.md到.claude/skills/
  └─ Phase 5: 确认和报告
  ↓
VERIFICATION_AGENT 执行验证
  ├─ 严格限制：只读，不修改项目
  ├─ 5步基线：读README→构建→测试→lint→检查回归
  ├─ 按类型执行：前端/API/CLI/基础设施/数据库等
  ├─ 对抗性探针：并发/边界/幂等性/孤立操作
  └─ 输出：Command/Output/Result格式→最终VERDICT
```

这些文件构成了Claude Code验证系统的完整流程！
