---
session_id: agent-70c1de36
date: 2026-04-16
time_start: 2026-04-16T04:01:47.780Z
time_end: 2026-04-16T04:02:20.151Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: main
---

[2026-04-16 12:01:47] [user]: 探索 /Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src 目录下所有文件，重点了解：
1. App.tsx 和 App.js 的内容
2. CardMessage 组件
3. 时序图相关组件（SequenceDiagram.js 等）
4. useEventLog hook
5. chat-stream 接口是如何调用的，返回什么数据格式
6. 前端如何处理消息渲染

请返回每个关键文件的完整内容摘要。
[2026-04-16 12:01:55] [assistant]: 我将帮你全面探索该目录。让我先列出目录结构，然后并行读取所有关键文件。
[2026-04-16 12:02:01] [assistant]: 现在让我并行读取所有关键文件：
[2026-04-16 12:02:20] [assistant]: 现在我已经完整读取了所有文件。以下是详细的摘要报告：

---

## 前端代码完整摘要

### 1. App.tsx / App.js (主应用组件)

**路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/App.tsx`

**核心功能**:
- 主聊天界面，支持用户与 Agent 交互
- 支持两种模式切换: `endpoint` 和 `mcp`
- 使用 SSE (Server-Sent Events) 流式接收响应

**chat-stream 接口调用方式**:
```typescript
POST /chat-stream
Body: { message: string, mode: 'endpoint' | 'mcp' }
```

**返回数据格式 (SSE)**:
```typescript
interface StreamEvent {
  type: 'status' | 'complete' | 'error';
  state?: string;
  message?: string;
  result?: any;      // complete 时返回
  code?: number;     // error 时返回
}
```

**SSE 事件类型**:
- `status`: 更新状态文本 (`setStatusText`)
- `complete`: 完成响应，解析 `result` 为消息 parts
- `error`: 错误处理

**响应解析 `parseResponseParts`**:
```typescript
// result 结构
{
  text?: string;                    // 文本内容
  mcp_ui_resource?: {               // MCP UI 资源
    resourceUri: string;
    toolName?: string;
    toolResult?: {...};
    uiMetadata?: { 'preferred-frame-size'?: { width, height } };
  }
}
```

---

### 2. CardMessage 组件

**路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/CardMessage.tsx`

**功能**: 渲染 MCP-UI 资源卡片，使用 `@mcp-ui/client` 的 `AppRenderer`

**关键 Props**:
- `resourceUri`: 资源 URI
- `toolName`: 工具名称
- `toolResult`: 工具返回结果
- `uiMetadata`: UI 元数据 (包含 preferred-frame-size)

**关键 API 调用**:
1. **`/a2a-tool-call`**: 调用工具
   ```typescript
   POST /a2a-tool-call
   Body: { toolName: string, arguments: {} }
   Response: { toolResult: any }
   ```

2. **`/resource-proxy`**: 读取资源
   ```
   GET /resource-proxy?uri={uri}&source={app|host}
   ```

**事件处理**:
- `onCallTool`: 工具调用
- `onReadResource`: 资源读取
- `onSizeChanged`: iframe 高度变化 (自适应高度)
- `onMessage`: 用户在 iframe 中发送消息

---

### 3. SequenceDiagram 组件 (时序图)

**路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/SequenceDiagram.tsx`

**功能**: 基于 ECharts 渲染交互式时序图

**预定义参与者**:
```typescript
const ALL_PARTICIPANTS = [
  'Frontend', 'CF-Agent', 'CF-LLM', 
  'SG-Agent', 'SG-LLM', 'MCP-Server', 
  'ResourceCenter', 'BusinessAPI'
];
```

**核心特性**:
- 自动过滤实际使用的参与者
- 请求用实线箭头，响应用虚线箭头
- 悬停高亮同一 span_id 的请求/响应对
- 点击显示详情面板 (SpanDetailPanel)
- 支持 tooltip 显示事件详情

**样式常量**:
- `HEADER_HEIGHT = 44`
- `ROW_HEIGHT = 28`
- `ARROW_SIZE = 6`
- 请求箭头: `#2563eb` (蓝色)
- 响应箭头: `#64748b` (灰色)
- 悬停高亮: `#f59e0b` (橙色)

---

### 4. useEventLog Hook

**路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/hooks/useEventLog.ts`

**功能**: 通过 EventSource 订阅事件日志

```typescript
function useEventLog(urls: string[]): EventLogEntry[]
```

**使用方式**:
```typescript
const entries = useEventLog(['/events']);
```

**处理逻辑**:
- 创建多个 EventSource 连接
- 解析 JSON 数据为 `EventLogEntry`
- 按时间排序，保留最近 100 条
- 组件卸载时自动关闭连接

---

### 5. types.ts (类型定义)

**路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/types.ts`

```typescript
// 文本消息部分
interface TextPart {
  kind: 'text';
  text: string;
}

// MCP UI 资源部分
interface McpUiResourcePart {
  kind: 'mcp_ui_resource';
  resourceUri: string;
  toolName?: string;
  toolResult?: {
    content?: Array<{ type: string; text: string }>;
    data?: Array<{ year: number; count: number }>;
    token?: string;
  };
  uiMetadata?: { 'preferred-frame-size'?: { width: number; height: number } };
}

// 消息部分联合类型
type MessagePart = TextPart | McpUiResourcePart;

// 聊天消息
interface ChatMessage {
  id: string;
  role: 'user' | 'agent';
  parts: MessagePart[];
}

// 事件日志条目
interface EventLogEntry {
  time: string;
  source: string;
  target: string;
  type: string;
  detail: string;
  span_id?: string;
  direction?: 'request' | 'response';
}
```

---

### 6. ChatMessage 组件

**路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/ChatMessage.tsx`

**功能**: 渲染单条聊天消息

**渲染逻辑**:
- 用户消息: 右对齐，蓝色背景 (#0084ff)，白色文字
- Agent 消息: 左对齐，灰色背景 (#f0f0f0)
- 遍历 `parts` 数组:
  - `kind === 'text'`: 渲染文本气泡
  - `kind === 'mcp_ui_resource'`: 渲染 CardMessage 组件

---

### 7. SpanDetailPanel 组件

**路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/SpanDetailPanel.tsx`

**功能**: 显示 Span 详情侧边面板

**显示内容**:
- span_id
- 耗时 (计算 request/response 时间差)
- 去程: 时间、路径、类型、参数
- 回程: 时间、路径、结果

---

### 8. ResizableDivider 组件

**路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/ResizableDivider.tsx`

**功能**: 可拖拽调整大小的分隔条

---

## 关键数据流总结

```
用户输入 → POST /chat-stream (SSE)
         ↓
     [status 事件] → 显示状态文本
         ↓
     [complete 事件] → 解析 result
         ↓
     parseResponseParts(result) → MessagePart[]
         ↓
     ChatMessage 组件渲染
         ↓
     - TextPart → 文本气泡
     - McpUiResourcePart → CardMessage (AppRenderer iframe)
```

---

## 文件列表汇总

| 文件 | 行数 | 主要功能 |
|------|------|----------|
| `App.tsx` | 255 | 主应用，SSE 流式通信 |
| `CardMessage.tsx` | 76 | MCP-UI 资源渲染 |
| `SequenceDiagram.tsx` | 382 | ECharts 时序图 |
| `useEventLog.ts` | 36 | EventSource hook |
| `types.ts` | 35 | TypeScript 类型定义 |
| `ChatMessage.tsx` | 46 | 消息渲染组件 |
| `SpanDetailPanel.tsx` | 125 | Span 详情面板 |
| `ResizableDivider.tsx` | 42 | 可调整大小分隔条 |
| `main.tsx` | 6 | 应用入口 |

**注意**: 每个 `.tsx` 文件都有对应的 `.js` 编译产物，是 TypeScript 编译后的结果。
