---
session_id: agent-c86ded3e
date: 2026-04-14
time_start: 2026-04-14T07:50:58.146Z
time_end: 2026-04-14T07:53:21.449Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: main
---

[2026-04-14 15:50:58] [user]: 在 /Users/lianzimeng/working/A2A-mcpUI 项目中，快速了解以下内容：
1. 当前 MCP-UI 支持哪些类型（rawHtml、uri 等），相关类型定义在哪里
2. tool 与 UI 的绑定方式是怎样的（看 mcp-ui-server 相关代码）
3. 前端如何渲染这些 UI 资源（iframe、组件等）
4. resource-proxy 的作用

请返回关键文件路径和核心数据结构定义。
[2026-04-14 15:51:07] [assistant]: 我来帮你快速了解这个项目的 MCP-UI 相关结构。首先让我获取 git remote URL 并搜索相关经验。
[2026-04-14 15:51:18] [assistant]: 现在让我搜索相关经验：
[2026-04-14 15:51:38] [assistant]: 现在让我进行项目结构的快速探索，从多个角度搜索相关代码：
[2026-04-14 15:51:47] [assistant]: 让我搜索关键的类型定义和代码文件：
[2026-04-14 15:52:01] [assistant]: 现在让我读取关键的源代码文件：
[2026-04-14 15:52:07] [assistant]: 现在让我查看 tools.py 文件和其他关键代码：
[2026-04-14 15:52:16] [assistant]: 让我查看 stargate-agent 中的 resource-proxy 实现和相关代码：
[2026-04-14 15:52:24] [assistant]: 让我获取完整的 resource-proxy 相关代码：
[2026-04-14 15:52:32] [assistant]: 现在让我查看 codeflicker-agent 的实现来理解 resource-proxy 的完整作用：
[2026-04-14 15:52:38] [assistant]: 现在让我获取 resource-proxy 的完整实现：
[2026-04-14 15:52:43] [assistant]: 现在让我查看相关的设计文档和架构文档：
[2026-04-14 15:52:48] [assistant]: 完美！现在让我查看前端的 App.tsx 来了解完整的渲染流程：
[2026-04-14 15:52:58] [assistant]: 完美！现在让我保存经验并编写最终报告：
[2026-04-14 15:53:21] [assistant]: ## 快速了解总结

我已为你完整分析了 A2A-mcpUI 项目的 MCP-UI 核心架构，以下是关键信息汇总：

---

## 1. **MCP-UI 支持的类型** ✅

| 类型 | 定义位置 | 说明 |
|---|---|---|
| **rawHtml** | `mcp-ui-protocol-fields.md` (第 47 行) | 直接传入 HTML 字符串 |
| **externalUrl** | `mcp-ui-protocol-fields.md` (第 48 行) | 传入外部 URL，服务端自动注入 `<base>` 标签 |
| **encoding** | 同上 (第 53 行) | `text` 或 `blob` (Base64) 编码 |
| **MIME Type** | 同上 (第 297 行) | 固定为 `text/html;profile=mcp-app` |

**核心数据结构** （`packages/codeflicker-frontend/src/types.ts`）:

```typescript
interface McpUiResourcePart {
  kind: 'mcp_ui_resource';
  resourceUri: string;           // ui://stargate/employee-trend
  toolName?: string;
  toolResult?: { content: [...], data: [...] };
  uiMetadata?: { 'preferred-frame-size': { width, height } };
}
```

---

## 2. **Tool 与 UI 的绑定方式** 🔗

**通过 `_meta.ui.resourceUri` 字段关联：**

```python
# 后端（stargate-mcp-ui-server/main.py）
@mcp.tool()
async def query_employee_trend() -> dict:
    return {
        "_meta": { "ui": { "resourceUri": "ui://stargate/employee-trend" } },
        "content": [{"type": "text", "text": "..."}],
        "resource": {...}
    }

@mcp.resource("ui://stargate/employee-trend", mime_type="text/html;profile=mcp-app")
async def employee_trend_resource() -> str:
    return html_content  # 返回渲染的 HTML
```

**关键文件：**
- `/packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/main.py` (第 14-24 行)
- `/packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/tools.py` (第 1-88 行)

---

## 3. **前端渲染机制** 🎨

**三层渲染架构：**

1. **AppRenderer 组件** (`@mcp-ui/client`)
   - 位置: `packages/codeflicker-frontend/src/components/CardMessage.tsx` (第 46-71 行)
   - 作用: 自动从 resourceUri 拉取资源，管理 iframe 生命周期

2. **sandbox_proxy.html** (沙箱中间层)
   - 位置: `packages/codeflicker-frontend/public/sandbox_proxy.html`
   - 作用: 接收 `ui/notifications/sandbox-resource-ready` 消息，创建 iframe，代理 postMessage 通信

3. **iframe 内的卡片 HTML** (Module Federation 动态加载)
   - 加载 Remote Module: `http://localhost:3004/remoteEntry.js`
   - 通过 React 18 动态渲染员工趋势图表

**完整流程代码示例：**

```typescript
<AppRenderer
  ref={rendererRef}
  toolName={toolName}
  toolResult={toolResult}
  sandbox={{ url: sandboxUrl }}
  toolResourceUri={resourceUri}
  onReadResource={async ({ uri }) => {
    // 通过 /resource-proxy 获取资源
    const res = await fetch(`/resource-proxy?uri=${uri}&source=host`);
    return res.json();
  }}
  onSizeChanged={({ height }) => {
    setIframeHeight(height);  // 动态调整高度
  }}
/>
```

---

## 4. **resource-proxy 的作用** 🚀

**resource-proxy 是分布式资源管理中间层：**

```
前端 → CF-Agent(/resource-proxy) → SG-Agent(/mcp/resources/read) → MCP-Server
```

**实现位置和逻辑：**

**CF-Agent** (`packages/codeflicker-agent/src/codeflicker_agent/main.py` 第 207-235 行)：

```python
@app.get("/resource-proxy")
async def resource_proxy(uri: str, source: str = "host"):
    # 1. 验证 uri 前缀是否为 ui://stargate/
    if uri.startswith("ui://stargate/"):
        # 2. 记录 SSE 事件日志 (用于时序图追踪)
        inner_span = sse_logger.emit_request("CF-Agent", "SG-Agent", "mcp-resources/read")
        
        # 3. 转发请求到 SG-Agent
        resp = await client.get(
            "http://localhost:3001/mcp/resources/read",
            params={"uri": uri}
        )
        
        # 4. 返回资源内容
        return JSONResponse(data)
```

**SG-Agent** (`packages/stargate-agent/src/stargate_agent/main.py` 第 110-194 行)：

```python
@app.get("/mcp/resources/read")
async def mcp_resources_read(uri: str):
    # 处理三类资源：
    # 1. ui://stargate/employee-trend → 调用 MCP-Server 读取
    # 2. ui://stargate/card/... → 动态生成 Module Federation 加载脚本
    # 3. 其他 → 返回 404
    
    if uri.startswith("ui://stargate/card/"):
        # 从 card_cache 获取实例信息，生成包含 Module Federation 的 HTML
        return create_ui_resource({
            "uri": uri,
            "content": {"type": "rawHtml", "htmlString": html},
            "encoding": "text"
        })
```

**关键作用：**

| 功能 | 说明 |
|---|---|
| **CORS 隔离** | 前端 HTTP 请求跨域代理，避免直接跨域 |
| **事件追踪** | 通过 SSE 记录完整的 CF-Agent → SG-Agent → MCP-Server 链路 |
| **路由分发** | 根据 URI 前缀智能路由到不同的数据源 |
| **双向通信支持** | iframe 内组件通过 postMessage → `/tool-call` → SG-Agent 调用工具 |

---

## 关键文件速查表 📋

| 模块 | 文件路径 | 核心内容 |
|---|---|---|
| **类型定义** | `/mcp-ui-protocol-fields.md` | UIResource 完整字段说明 |
| | `packages/codeflicker-frontend/src/types.ts` | TS 类型定义 |
| **MCP Server** | `packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/main.py` | tool + resource 绑定实现 |
| | `packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/tools.py` | HTML 生成逻辑（Module Federation） |
| **SG-Agent** | `packages/stargate-agent/src/stargate_agent/main.py` | `/mcp/resources/read` 资源路由 |
| **CF-Agent** | `packages/codeflicker-agent/src/codeflicker_agent/main.py` | `/resource-proxy` 中间层代理 |
| **前端渲染** | `packages/codeflicker-frontend/src/components/CardMessage.tsx` | AppRenderer 使用 + resource-proxy 调用 |
| | `packages/codeflicker-frontend/public/sandbox_proxy.html` | iframe 沙箱代理逻辑 |
| **设计文档** | `docs/designs/2026-11-04-stargate-mcp-ui-server.md` | MCP-UI 架构决策 |

---

这套架构通过 MCP-UI + A2A 协议，实现了多 Agent 环境下的组件动态加载、跨域资源管理和完整的链路追踪！
