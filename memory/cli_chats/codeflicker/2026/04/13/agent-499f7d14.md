---
session_id: agent-499f7d14
date: 2026-04-13
time_start: 2026-04-13T07:01:05.872Z
time_end: 2026-04-13T07:03:45.556Z
project: users-lianzimeng-workspace-xp-verify
git_branch: 
---

[2026-04-13 15:01:05] [user]: 请查阅 codeflicker CLI 的使用文档，了解：
1. system prompt 如何配置（配置文件路径、格式）
2. codeflicker 的 Agent 工作流程（任务开始/结束的时机）
3. MCP 工具调用的相关说明
4. hooks 机制（如果有）

请返回具体的配置方式和相关文档内容。
[2026-04-13 15:03:45] [assistant]: 根据 codeflicker CLI 官方文档，以下是您所问的四个方面的详细配置说明：

---

## 1. System Prompt 配置

### 配置文件路径
- 项目级: `<project-root>/.codeflicker/config.json`
- 用户级: `~/.codeflicker/config.json`

### 配置方式

**通过 CLI 命令查看和设置:**
```bash
# 查看当前 systemPrompt
flickcli config get systemPrompt

# 设置单行提示
flickcli config set systemPrompt "你是一名专业前端架构师，回答要简洁、可执行，并提供必要的代码示例。"

# 设置多行提示（从文件加载）
prompt=$(< /path/to/prompt.txt)
flickcli config set systemPrompt "$prompt"

# 设置全局配置（跨项目生效）
flickcli config set systemPrompt "$prompt" -g
```

**通过 config.json 文件配置:**
```json
{
  "systemPrompt": "你的自定义提示词内容..."
}
```

---

## 2. Agent 工作流程（任务开始/结束时机）

### 任务生命周期

| 阶段 | 说明 |
|------|------|
| **触发接收** | 用户提交任务，提取目标、约束条件与成功标准 |
| **任务规划** | 加载合适的子代理（Explore/GeneralPurpose/UsageGuide），将大任务拆分成子任务 |
| **执行阶段** | 根据子任务独立性决定并行或串行执行 |
| **监控与聚合** | 实时跟踪子任务进度，处理冲突或缺失信息 |
| **结束判定** | 满足接受标准、达到超时阈值、或用户取消 |
| **收尾与持久化** | 汇总结果、日志，写回项目结构 |

### 任务开始时机
- 用户明确启动时（如运行 /init 生成任务计划）
- 任务具备可拆分性且有可并行化的子任务
- 已加载并就绪的代理可用

### 任务结束时机
- **成功结束**: 所有关键子任务按要求产出并通过接受标准
- **失败/中止**: 关键子任务失败且不可重试，或总失败次数超过阈值
- **全局超时**: 任务总超时未完成
- **用户取消**: 手动取消信号

### 子代理加载优先级
1. 内置代理
2. 插件提供的代理
3. 全局个人代理 (`~/.codeflicker/agents/`)
4. 项目级代理 (`.codeflicker/agents/`)

---

## 3. MCP 工具调用配置

### 配置方式

**方式一：通过 CLI 添加**
```bash
# 本地 stdio 服务器
flickcli mcp add <name> -- <command> [args...]

# 示例：添加 Playwright MCP
flickcli mcp add playwright -- npx -y @executeautomation/playwright-mcp-server

# 全局生效
flickcli mcp add playwright -- npx -y @executeautomation/playwright-mcp-server -g
```

**方式二：通过 config.json 配置文件**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@executeautomation/playwright-mcp-server"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "github-mcp-server"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
```

### 常用 MCP 管理命令
```bash
# 列出所有 MCP 服务器
flickcli mcp ls
flickcli mcp ls -g   # 查看全局服务器

# 查看指定服务器详情
flickcli mcp get playwright

# 启用/禁用
flickcli mcp enable playwright
flickcli mcp disable playwright

# 移除服务器
flickcli mcp rm playwright

# 交互式会话中查看可用 MCP
/mcp
```

---

## 4. Hooks 机制

codeflicker 通过插件机制提供钩子，主要包括:
- **config**: 初始化阶段动态调整配置
- **context**: 对话前注入额外上下文（相当于 pre）
- **conversation**: 对话结束后执行（相当于 post）
- **destroy**: 清理资源

### 示例：实现 pre/post 钩子

插件目录结构:
```
~/.codeflicker/plugins/prepost-hooks/index.js
```

```javascript
module.exports = {
  name: 'prepost-hooks',
  
  // 前置：在对话中注入额外上下文
  context() {
    return {
      systemPromptExtras: "你是一个安全且合规的前端开发助手。始终遵循公司编码规范。",
      userRole: "Frontend Engineer",
      policy: "所有输出禁止包含敏感信息，输出可审计。"
    };
  },
  
  // 后置：对话结束后执行
  conversation(opts) {
    const { userPrompt, result, startTime, endTime } = opts;
    const durationMs = endTime.getTime() - startTime.getTime();
    
    console.log(`[prepost-hooks] duration: ${durationMs}ms`);
    
    // 检测敏感信息
    const text = String(result ?? '');
    if (text.toLowerCase().includes('password') || text.toLowerCase().includes('secret')) {
      console.warn('[prepost-hooks] 警告：结果中可能包含敏感信息');
    }
  }
};
```

### 使用插件
```bash
flickcli --plugin-dir ~/.codeflicker/plugins/prepost-hooks
```

---

## 快速参考表

| 功能 | 配置位置 | 命令/方式 |
|------|----------|-----------|
| System Prompt | config.json | `flickcli config set systemPrompt` |
| Agent 工作流 | AGENTS.md | 内置子代理自动管理 |
| MCP 配置 | config.json | `flickcli mcp add/ls/enable/rm` |
| Hooks | ~/.codeflicker/plugins/ | 插件机制 context/conversation |

如果您需要更具体的配置示例或针对特定场景的指导，请告诉我您的使用场景，我可以提供更精准的建议。
