---
session_id: agent-e1e22942
date: 2026-04-10
time_start: 2026-04-10T11:36:22.266Z
time_end: 2026-04-10T11:37:12.261Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: 
---

[2026-04-10 19:36:22] [user]: 请全面探索 /Users/lianzimeng/working/A2A-mcpUI 项目的结构，包括：
1. 顶级目录和文件列表
2. package.json / 依赖
3. 现有源码结构（src/目录下）
4. 现有测试结构
5. 任何配置文件（tsconfig, vite, etc.）
6. README 内容

请返回一份详尽的结构报告。
[2026-04-10 19:36:24] [assistant]: 我将全面探索这个项目的结构。让我按步骤进行。
[2026-04-10 19:36:28] [assistant]: 有趣！项目结构不包含 package.json 和 README.md。让我进一步探索文档和配置文件。
[2026-04-10 19:36:30] [assistant]: 现在让我读取设计文档：
[2026-04-10 19:36:37] [assistant]: 现在让我使用 glob 模式来检查是否有源代码目录或其他项目文件：
[2026-04-10 19:36:39] [assistant]: 让我检查是否有 packages 或 src 目录：
[2026-04-10 19:36:45] [assistant]: 最后让我列出所有文件：
[2026-04-10 19:37:12] [assistant]: 完美！现在我有了完整的视图。让我整理一份详尽的结构报告。

---

## A2A-mcpUI 项目结构详尽报告

### 1. 顶级目录和文件列表

```
/Users/lianzimeng/working/A2A-mcpUI/
├── a2a-protocol.md                           # A2A 协议总结
├── a2a-protocol-fields.md                    # A2A 协议完整字段解析
├── mcp-ui-protocol.md                        # MCP-UI 协议总结
├── mcp-ui-protocol-fields.md                 # MCP-UI 协议完整字段解析
├── design.md                                 # A2A 协议渲染 Card 设计方案
└── docs/
    └── designs/
        ├── 2026-04-10-a2a-mcpui-multi-agent-demo.md    # 多 Agent Demo 设计
        └── 2026-04-10-mcp-ui-minimal-demo.md           # MCP-UI 最小验证 Demo
```

**共 7 个文件，2 层目录结构。** 当前不包含 src/ 代码、package.json、测试文件或配置文件。

---

### 2. Package.json / 依赖

**不存在。** 项目当前为纯文档项目，无 package.json 文件。

根据设计文档规划，未来工程结构包括：
- 前端项目：需要 React、Webpack 5 等依赖
- Agent 项目：需要 Python、python-a2a SDK、MCP 库等
- 测试框架等

---

### 3. 现有源码结构（src/ 目录下）

**不存在。** 当前项目无源码实现，仅为设计文档阶段。

根据设计规划，未来的 monorepo 结构：
```
packages/
├── codeflicker-frontend/        # React Web UI（端口 3000）
├── codeflicker-agent/           # Python 代理（端口 3002）
├── stargate-agent/              # Python 代理（端口 3001）
├── employee-chart-card/         # 模块联邦卡片（端口 3004）
└── resource-center-mock/        # 资源中心 Mock（端口 3003）
```

---

### 4. 现有测试结构

**不存在。** 项目当前无测试代码。

---

### 5. 任何配置文件（tsconfig, vite, etc.）

**不存在。** 项目当前无配置文件。

---

### 6. README 内容

**不存在。** 项目无 README.md。

---

## 详尽内容总结

### 📄 核心文档说明

#### a2a-protocol.md（104 行）
- A2A（Agent2Agent）协议概述
- 解决的核心问题：多 Agent 协作、互操作性、安全隐患
- A2A vs MCP 的关系与差异
- 请求生命周期（Discovery → Auth → Task/Stream）

#### a2a-protocol-fields.md（367 行）
完整的 A2A 协议字段解析，包含：
- **AgentCard**：Agent 元数据（名称、能力、安全方案）
- **Part**：内容容器（文本、文件、结构数据）
- **Message**：单轮通信单元（role、parts、metadata）
- **Task**：有状态工作单元（状态机、artifacts、history）
- **Artifact**：任务产出物
- **TaskState 枚举**：submitted → working → completed/failed（8 种状态）
- **JSON-RPC 方法一览**：message/send、message/stream、tasks/get 等
- **SSE 流式事件类型**：Task、TaskStatusUpdateEvent、TaskArtifactUpdateEvent
- **错误码体系**（-32700 ~ -32006）
- **标识符体系**：contextId、taskId、messageId、artifactId

#### mcp-ui-protocol.md（80 行）
- MCP-UI 是 MCP 的 UI 层扩展
- 核心 API：`createUIResource()`
- 内容类型：rawHtml、externalUrl
- 编码方式：text、blob
- MCP Apps 推荐模式
- HTML 端通信

#### mcp-ui-protocol-fields.md（321 行）
完整的 MCP-UI 协议字段解析，包含：
- **UIResource**：核心数据结构（uri、mimeType、text/blob、_meta）
- **createUIResource 参数**：uri、content、encoding、metadata、uiMetadata
- **Tool 的 _meta 字段**：resourceUri 绑定
- **postMessage 消息协议**：
  - Host → Guest：tool-input、tool-result、host-context-changed
  - Guest → Host：tools/call、ui/message、ui/open-link
- **AppRenderer Props**（客户端组件）：client、toolName、sandbox、toolInput 等
- **URI 规范**：`ui://<component-name>/<instance-id>`
- **MIME Type**：`text/html;profile=mcp-app`

#### design.md（301 行）
A2A 协议渲染 Card 的完整设计方案，包含：
- **背景与目标**：实现 CodeFlicker 调 Stargate，渲染业务自定义卡片
- **各模块职责**：6 个模块的分工矩阵
- **A2A 协议扩展**：`kind: "mcp_ui_resource"` 新增 part 类型
- **完整时序图**（Mermaid）：
  - 调用 & Card Instance 生产阶段
  - 前端渲染阶段（换 token → MF 加载 → 渲染）
  - 卡片交互阶段
- **卡片表单提交时序图**：
  - 方案 A：卡片直调 Stargate API
  - 方案 B：经由 Agent 链路处理（postMessage）
- **关键设计决策**：
  - 为何使用 cardInstanceId：敏感数据不明文传输
  - 为何使用 Module Federation：灵活加载、交互能力无限制
  - Token 注入方式：串行执行
- **HTML 片段渲染**：
  - 复用 `mcp_ui_resource` 结构，通过 mimeType 区分
  - iframe sandbox 隔离
  - postMessage 通信协议（html_ready、html_resize、html_action）
- **Card 与 HTML 片段对比表**
- **待评审项**（3 个）

#### 2026-04-10-a2a-mcpui-multi-agent-demo.md（199 行）
多 Agent Demo 的详尽设计方案，包含：
- **关键决策表**：8 个决策项（技术验证、真实 LLM、Python + Node.js、标准 A2A 等）
- **探索的替代方案**：HTML 片段方案、token 跳过方案
- **工程结构**：5 个子包（资源中心、卡片、两个 Agent、前端）
- **A2A 协议扩展结构示例**
- **各模块职责详解**：
  - resource-center-mock：纯静态 HTTP 服务
  - employee-chart-card：Webpack5 MF + ECharts
  - stargate-agent：Python MCP-UI Server + A2A Agent
  - codeflicker-agent：MCP 客户端 + A2A 调用方
  - codeflicker-frontend：极简 React UI + AppRenderer
- **完整业务时序**：用户输入 → 意图识别 → A2A 调用 → 卡片生成 → 前端渲染
- **卡片内两种交互路径**：直调 API vs postMessage
- **数据流转记录器**：SSE 日志实时展示
- **鉴权设计表**：简化方案（mock 实现）

#### 2026-04-10-mcp-ui-minimal-demo.md（196 行）
MCP-UI 最小验证 Demo 的设计方案，包含：
- **关键决策表**：与多 Agent Demo 类似，但更明确（官方 python-a2a SDK、经 CF-Agent 代理等）
- **替代方案探索**
- **工程结构**：同上（5 子包）
- **服务依赖启动顺序**：resource-center-mock → stargate-agent → codeflicker-agent → codeflicker-frontend
- **完整调用链路**（流程图）
- **各服务关键接口详解**
- **前端三区域布局**：对话列表、输入框、右侧 Panel
- **AppRenderer 配置代码示例**
- **卡片组件 Props 接口**
- **两种交互路径表**
- **数据流转记录器日志格式示例**
- **环境变量配置**：LLM Key、模型选择等
- **鉴权策略**（Demo 简化版）

---

## 项目特点总结

✓ **纯文档设计项目**：处于架构设计阶段，无实现代码  
✓ **完整的协议规范文档**：A2A 和 MCP-UI 两个协议的全面解析  
✓ **详尽的设计方案**：包括业务场景、时序图、关键决策、替代方案分析  
✓ **两个 Demo 设计**：一个多 Agent 完整方案，一个最小验证方案  
✓ **模块化思维**：清晰的职责分工、端口划分、通信协议定义  
✓ **实现路线明确**：Monorepo 结构、技术栈（Python + React）、依赖关系确定  

---

## 文件详情表

| 文件路径 | 行数 | 内容类型 | 关键内容 |
|---------|------|--------|---------|
| a2a-protocol.md | 104 | 协议总结 | A2A 基础概念、与 MCP 关系 |
| a2a-protocol-fields.md | 367 | 字段解析 | 9 个大段落（AgentCard、Part、Message、Task 等） |
| mcp-ui-protocol.md | 80 | 协议总结 | MCP-UI 扩展、核心 API、推荐模式 |
| mcp-ui-protocol-fields.md | 321 | 字段解析 | 11 个大段落（UIResource、postMessage 等） |
| design.md | 301 | 设计方案 | 核心的 Card 渲染设计 + HTML 片段扩展 |
| 2026-04-10-a2a-mcpui-multi-agent-demo.md | 199 | Demo 设计 | 完整的多 Agent 技术验证方案 |
| 2026-04-10-mcp-ui-minimal-demo.md | 196 | Demo 设计 | 最小化验证 Demo 设计 |
| **总计** | **1568** | 文档 | 完整的技术框架设计 |
