---
session_id: agent-508bcd59
date: 2026-04-13
time_start: 2026-04-13T11:14:16.336Z
time_end: 2026-04-13T11:15:50.727Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: 
---

[2026-04-13 19:14:16] [user]: 请探索 /Users/lianzimeng/working/A2A-mcpUI 项目，重点查找：
1. packages/stargate-mcp-ui-server 的 main.py 或 tools.py，看 resource 返回的 HTML 内容（build_employee_trend_shell 函数的实现）
2. packages/cf-agent 或 packages/codeflicker-agent 中处理 mcp_ui_resource / resource-proxy 的代码，看它如何渲染 iframe
3. 前端中处理 resource-proxy 或 iframe 渲染的代码

返回每个关键文件的完整路径和核心代码内容。
[2026-04-13 19:14:19] [assistant]: 我来帮你探索这个项目，找出关键的资源处理和渲染代码。让我从项目结构开始。
[2026-04-13 19:14:32] [assistant]: 现在让我查找关键文件：
[2026-04-13 19:14:55] [assistant]: 现在让我查一下前端中处理 resource-proxy 的逻辑，以及涉及 iframe 和 AppRenderer 的代码：
[2026-04-13 19:15:00] [assistant]: 完美！现在让我查看一下 package.json 来确认依赖关系，并搜索是否有其他相关文件：
[2026-04-13 19:15:05] [assistant]: 现在让我查看 vite.config.ts 和前端的整体流程：
[2026-04-13 19:15:11] [assistant]: 让我查看 MCP-UI 协议的相关文档：
[2026-04-13 19:15:50] [assistant]: 完美！现在我有了完整的信息。让我生成一份综合报告：

## 项目探索完整报告

我已成功探索 A2A-mcpUI 项目的关键代码流程。以下是核心文件及实现详情：

---

## 1️⃣ **MCP-UI Server 的 HTML 生成（resources 返回内容）**

### 文件位置
- `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/tools.py` (88 行)
- `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/main.py` (52 行)

### 核心代码 - `build_html()` 函数
```python
def build_html(component_name: str = COMPONENT_NAME) -> str:
    return f"""<!DOCTYPE html>
<html><head><meta charset="UTF-8">
<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
</head>
<body style="margin:0">
<div id="root"></div>
<script src="{REMOTE_ENTRY_URL}"></script>
<script>
{MCP_INIT_SCRIPT}
(function() {{
  Promise.resolve().then(function() {{
    if (typeof {CONTAINER_NAME} === 'undefined') throw new Error('Container {CONTAINER_NAME} not found');
    var shareScope = Object.create(null);
    shareScope['default'] = {{}};
    shareScope['default']['react'] = {{
      '18.3.1': {{ get: function() {{ return function() {{ return React; }}; }}, loaded: 1, from: 'host' }}
    }};
    shareScope['default']['react-dom'] = {{
      '18.3.1': {{ get: function() {{ return function() {{ return ReactDOM; }}; }}, loaded: 1, from: 'host' }}
    }};
    if ({CONTAINER_NAME}.init) {{
      try {{ {CONTAINER_NAME}.init(shareScope['default']); }} catch(e) {{}}
    }}
    return {CONTAINER_NAME}.get('./{component_name}');
  }}).then(function(factory) {{
    var Comp = factory().default;
    ReactDOM.createRoot(document.getElementById('root'))
      .render(React.createElement(Comp, {{}}));
  }}).catch(function(e) {{
    document.body.innerHTML = '<p style="color:red;padding:16px">加载失败: ' + e.message + '</p>';
  }});
}})();
</script>
</body></html>"""
```

### Resource 返回方式
```python
@mcp.resource(RESOURCE_URI)
async def employee_trend_resource() -> str:
    r = get_ui_resource()
    return r["resource"]["text"]  # 返回 HTML 字符串

def get_ui_resource() -> dict:
    resource = create_ui_resource({
        "uri": RESOURCE_URI,
        "content": {"type": "rawHtml", "htmlString": build_html()},
        "encoding": "text",
    })
    return resource.model_dump(mode="json")
```

**关键点**：
- 使用 **Module Federation** (remoteEntry.js)
- 从 `http://localhost:3004/remoteEntry.js` 加载远程组件
- 共享 React 18.3.1 依赖
- 支持懒加载版本 (`build_lazy_html()`)

---

## 2️⃣ **Stargate Agent 处理 mcp_ui_resource 的代码**

### 文件位置
- `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-agent/src/stargate_agent/main.py` (359 行)
- `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-agent/src/stargate_agent/shell_builder.py` (57 行)

### A2A Response 构造（包含 mcp_ui_resource）
```python
response_data = {
    "text": "已为您查询快手历年员工趋势数据，共 5 年记录。",
    "mcp_ui_resource": {
        "kind": "mcp_ui_resource",
        "resourceUri": resource_uri,  # ui://stargate/employee-trend
        "toolName": "query_employee_trend",
        "toolResult": {
            "content": [{"type": "text", "text": "..."}],
            "data": trend_resp["data"],  # 图表数据
            "token": trend_resp["token"]
        },
        "uiMetadata": {
            "preferred-frame-size": {"width": 560, "height": 420}
        },
    },
}
```

### 动态 HTML 生成（用于 card 资源）
```python
@app.get("/mcp/resources/read")
async def mcp_resources_read(uri: str):
    if uri.startswith("ui://stargate/card/"):
        card_id = uri.removeprefix("ui://stargate/card/")
        inst = card_cache.get(card_id)
        
        # 构造 HTML shell
        html = f"""<!DOCTYPE html>
<html><head><meta charset="UTF-8">
<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
</head>
<body style="margin:0">
<div id="root"></div>
<script src="{inst.remote_entry_url}"></script>
<script>
{MCP_INIT_SCRIPT}
(function() {{
  // Module Federation 动态加载逻辑...
  return {inst.container_name}.get('./{inst.component_name}');
}})();
</script>
</body></html>"""
        
        resource = create_ui_resource({
            "uri": uri,
            "content": {"type": "rawHtml", "htmlString": html},
            "encoding": "text",
        })
        return JSONResponse({
            "contents": [{
                "uri": r["resource"]["uri"],
                "mimeType": "text/html;profile=mcp-app",
                "text": r["resource"]["text"]
            }]
        })
```

### MCP_INIT_SCRIPT（两端通信信号）
```python
MCP_INIT_SCRIPT = """
(function() {
  window.addEventListener('message', function(e) {
    var msg = e.data;
    if (msg && msg.method === 'ui/notifications/sandbox-resource-ready') {
      window.parent.postMessage({
        jsonrpc: '2.0',
        method: 'ui/notifications/initialized',
        params: {}
      }, '*');
    }
  });
})();
"""
```

---

## 3️⃣ **CodeFlicker Agent 处理 resource-proxy**

### 文件位置
- `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-agent/src/codeflicker_agent/main.py` (183 行)

### Resource-Proxy 端点
```python
@app.get("/resource-proxy")
async def resource_proxy(uri: str):
    sse_logger.emit("Frontend", "CF-Agent", "resource-proxy", uri)
    if uri.startswith("ui://stargate/"):
        sse_logger.emit("CF-Agent", "SG-Agent", "MCP resources/read", uri)
        async with httpx.AsyncClient() as client:
            resp = await client.get(
                f"{SG_AGENT_BASE_URL}/mcp/resources/read",
                params={"uri": uri},
            )
            return JSONResponse(resp.json())
    return JSONResponse({"error": "Unknown resource host"}, status_code=404)
```

### mcp_ui_resource 的转发
```python
def _call_sg_agent(user_text: str, mode: str) -> list:
    # ... 调用 StarGate Agent 获取响应
    response_msg = client.send_message(...)
    
    raw = response_msg.content.text
    data = json.loads(raw)
    
    parts = []
    if "text" in data:
        parts.append({"kind": "text", "text": data["text"]})
    if "mcp_ui_resource" in data:
        parts.append(data["mcp_ui_resource"])  # ✓ 直接转发
    return parts
```

---

## 4️⃣ **前端处理 iframe 和 AppRenderer 的代码**

### 文件位置
- `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/CardMessage.tsx` (75 行)
- `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/ChatMessage.tsx` (46 行)
- `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/public/sandbox_proxy.html` (62 行)

### CardMessage - AppRenderer 配置
```typescript
export const CardMessage: React.FC<Props> = ({
  resourceUri,
  toolName,
  toolResult,
  uiMetadata,
  onMessage,
  onLayout,
}) => {
  const { width = 560 } = uiMetadata?.['preferred-frame-size'] ?? {};
  const sandboxUrl = new URL('/sandbox_proxy.html', window.location.href);
  const [iframeHeight, setIframeHeight] = useState<number | undefined>(undefined);

  return (
    <div style={{
      border: iframeHeight ? '1px solid #e0e0e0' : 'none',
      borderRadius: 8,
      overflow: 'hidden',
      width,
      height: iframeHeight ?? 0,
      transition: 'height 0.2s ease',
    }}>
      <AppRenderer
        ref={rendererRef}
        toolName={toolName as string}
        toolResult={toolResult as any}
        sandbox={{ url: sandboxUrl }}  // ✓ 沙箱代理 URL
        toolResourceUri={resourceUri}   // ✓ MCP 资源 URI
        
        onReadResource={async ({ uri }) => {
          const res = await fetch(`/resource-proxy?uri=${encodeURIComponent(uri)}`);
          return res.json();
        }}
        
        onCallTool={handleCallTool}
        
        onSizeChanged={({ height }) => {
          if (height !== undefined) {
            setIframeHeight(height);
            onLayout?.();
          }
        }}
        
        onMessage={async (params) => {
          const textBlock = params.content.find((c: any) => c.type === 'text');
          if (textBlock && onMessage) {
            onMessage((textBlock as any).text);
          }
          return {};
        }}
      />
    </div>
  );
};
```

### Sandbox Proxy HTML - 双向通信中继
```html
<!DOCTYPE html>
<html>
<body style="margin:0;padding:0;width:100%;height:100%;">
  <script>
    var appIframe = null;

    window.addEventListener('message', function (e) {
      var msg = e.data;
      if (!msg || msg.jsonrpc !== '2.0') return;

      // ✓ 接收 Host 的资源就绪信号
      if (msg.method === 'ui/notifications/sandbox-resource-ready') {
        var html = msg.params && msg.params.html;
        if (!html) return;

        // 创建 iframe 运行 HTML
        appIframe = document.createElement('iframe');
        appIframe.style.width = '100%';
        appIframe.style.height = '100%';
        appIframe.style.border = 'none';
        appIframe.setAttribute('sandbox', 'allow-scripts allow-same-origin allow-forms allow-popups');

        var blob = new Blob([html], { type: 'text/html' });
        appIframe.src = URL.createObjectURL(blob);
        document.body.appendChild(appIframe);

        appIframe.addEventListener('load', function () {
          appIframe.contentWindow.postMessage(msg, '*');
        });
        return;
      }

      // ✓ 中继 Host → Guest 消息
      if (e.source === window.parent && appIframe && appIframe.contentWindow) {
        appIframe.contentWindow.postMessage(msg, '*');
        return;
      }

      // ✓ 中继 Guest → Host 消息
      if (appIframe && e.source === appIframe.contentWindow) {
        window.parent.postMessage(msg, '*');
        return;
      }
    });

    // 通知 Host 自己已准备就绪
    window.parent.postMessage({
      jsonrpc: '2.0',
      method: 'ui/notifications/sandbox-proxy-ready',
      params: {}
    }, '*');
  </script>
</body>
</html>
```

### ChatMessage - 消息部分分类渲染
```typescript
export const ChatMessage: React.FC<Props> = ({ message, onCardMessage, onLayout }) => (
  <div style={{ marginBottom: 12, display: 'flex' }}>
    {message.parts.map((part, i) => {
      if (part.kind === 'text') {
        return <div key={i}>{part.text}</div>;
      }
      if (part.kind === 'mcp_ui_resource') {
        return (
          <CardMessage
            key={i}
            resourceUri={part.resourceUri}
            toolName={part.toolName}
            toolResult={part.toolResult}
            uiMetadata={part.uiMetadata}
            onMessage={onCardMessage}
            onLayout={onLayout}
          />
        );
      }
      return null;
    })}
  </div>
);
```

### 类型定义 (types.ts)
```typescript
export interface McpUiResourcePart {
  kind: 'mcp_ui_resource';
  resourceUri: string;          // ui://stargate/employee-trend
  toolName?: string;            // query_employee_trend
  toolResult?: {
    content?: Array<{ type: string; text: string }>;
    data?: Array<{ year: number; count: number }>;  // 图表数据
    token?: string;             // 认证令牌
  };
  uiMetadata?: { 'preferred-frame-size'?: { width: number; height: number } };
}
```

---

## 📊 完整数据流链路

```
┌─────────────┐
│   前端      │
└──────┬──────┘
       │
       │ 1. 发送 /chat 请求
       ↓
┌─────────────────────┐
│ CodeFlicker Agent   │
│ (CF-Agent:3002)     │
└──────┬──────────────┘
       │
       │ 2. 调用 SG-Agent A2A
       ↓
┌──────────────────────┐
│  StarGate Agent      │
│  (SG-Agent:3001)     │
└──────┬───────────────┘
       │
       │ 3. 调用 MCP Tool
       ↓
┌──────────────────────────────────┐
│ MCP-UI Server                    │
│ (StarGate MCP:3005)              │
│                                  │
│ query_employee_trend tool:       │
│ └─ 返回 {                        │
│    _meta: { ui: {                │
│      resourceUri: "ui://..." }}, │
│    resource: { text: "..." }     │
│ }                                │
└──────┬───────────────────────────┘
       │
       │ 4. SG-Agent 构造响应
       │    mcp_ui_resource {
       │      resourceUri,
       │      toolResult
       │    }
       ↓
┌──────────────────┐
│ CF-Agent 转发    │
└──────┬───────────┘
       │
       │ 5. 前端接收 mcp_ui_resource
       ↓
┌──────────────────────┐
│ CardMessage render   │
│ AppRenderer          │
└──────┬───────────────┘
       │
       │ 6. AppRenderer 读取 resource
       │    onReadResource → /resource-proxy
       ↓
┌──────────────────────┐
│ CF-Agent resource-   │
│ proxy 端点           │
└──────┬───────────────┘
       │
       │ 7. 转发到 SG-Agent
       │    /mcp/resources/read
       ↓
┌──────────────────────┐
│ SG-Agent 返回 HTML   │
│ (build_html 生成)    │
└──────┬───────────────┘
       │
       │ 8. AppRenderer 在 sandbox
       │    中渲染 HTML
       ↓
┌──────────────────────┐
│ sandbox_proxy.html   │
│ (双向消息中继)       │
└──────┬───────────────┘
       │
       │ 9. 创建 iframe，运行
       │    Module Federation
       │    加载远程组件
       ↓
┌──────────────────────┐
│ EmployeeChart 组件   │
│ (remoteEntry.js)     │
│ 共享 React 18.3.1    │
└──────────────────────┘
```

---

## 🔑 关键总结

| 组件 | 职责 | 核心文件 |
|------|------|---------|
| **MCP-UI Server** | 生成 HTML shell + Module Federation 加载逻辑 | `tools.py` (build_html) |
| **StarGate Agent** | 构造 mcp_ui_resource + 动态生成 iframe HTML | `main.py` (mcp_ui_resource 构造) |
| **CodeFlicker Agent** | resource-proxy 代理 + 转发 mcp_ui_resource | `main.py` (/resource-proxy 端点) |
| **前端 AppRenderer** | 自动拉取资源、管理 iframe 生命周期 | `CardMessage.tsx` |
| **Sandbox Proxy** | JSON-RPC 双向通信中继 + iframe 隔离 | `sandbox_proxy.html` |

所有文件都是 **read-only** 模式探索完成 ✓
