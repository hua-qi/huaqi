---
session_id: agent-a48cb48c
date: 2026-04-11
time_start: 2026-04-11T04:08:00.830Z
time_end: 2026-04-11T04:09:13.629Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: 
---

[2026-04-11 12:08:00] [user]: 探索以下内容并返回详细信息：

1. 读取 /Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/CardMessage.tsx - 当前的 MF 加载逻辑
2. 读取 /Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/App.tsx
3. 读取 /Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/types.ts
4. 读取 /Users/lianzimeng/working/A2A-mcpUI/packages/stargate-agent/src/stargate_agent/main.py - 重点看 mcp_resources_read 接口
5. 读取 /Users/lianzimeng/working/A2A-mcpUI/packages/employee-chart-card/src/EmployeeChart.tsx - 组件 props
6. 读取 /Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/public/sandbox_proxy.html
7. 读取 /Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/vite.config.ts
8. 检查 @mcp-ui/client 的 AppRenderer 导出：grep -n "AppRenderer\|exports\|export" /Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/node_modules/@mcp-ui/client/dist/index.d.ts 2>/dev/null | head -50

返回所有文件的完整内容以及 AppRenderer 的 TypeScript 类型签名。
[2026-04-11 12:08:04] [assistant]: I'll help you explore these files and gather the detailed information. Let me start by reading all the files you specified.
[2026-04-11 12:08:08] [assistant]: 让我尝试查找 @mcp-ui/client 的类型定义文件：
[2026-04-11 12:08:11] [assistant]: 现在让我读取 AppRenderer 的类型定义和索引文件：
[2026-04-11 12:09:13] [assistant]: 完美！现在让我整理所有信息并生成详细的报告。

## 详细探索报告

我已经完整读取了所有指定的文件。以下是详细信息：

---

### 1. CardMessage.tsx - MF 加载逻辑
**文件路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/CardMessage.tsx`

```typescript
import React, { useEffect, useState } from 'react';
import * as ReactDOM from 'react-dom';
import type { CardData, McpUiResourcePart } from '../types';

interface Props {
  resourceUri: string;
  uiMetadata?: McpUiResourcePart['uiMetadata'];
}

type Status = 'loading' | 'error' | 'ready';

function buildSharedScope() {
  return {
    react: {
      [React.version]: {
        get: () => Promise.resolve(() => React),
        loaded: true,
        eager: true,
        from: 'host',
      },
    },
    'react-dom': {
      [ReactDOM.version]: {
        get: () => Promise.resolve(() => ReactDOM),
        loaded: true,
        eager: true,
        from: 'host',
      },
    },
  };
}

async function loadMFComponent(remoteEntryUrl: string, containerName: string, componentName: string): Promise<React.ComponentType<any>> {
  return new Promise((resolve, reject) => {
    if ((window as any)[containerName]) {
      resolveMF(containerName, componentName, resolve, reject);
      return;
    }
    const script = document.createElement('script');
    script.src = remoteEntryUrl;
    script.onload = () => resolveMF(containerName, componentName, resolve, reject);
    script.onerror = () => reject(new Error(`Failed to load ${remoteEntryUrl}`));
    document.head.appendChild(script);
  });
}

async function resolveMF(
  containerName: string,
  componentName: string,
  resolve: (c: React.ComponentType<any>) => void,
  reject: (e: Error) => void
) {
  const container = (window as any)[containerName];
  if (!container) { reject(new Error(`Container '${containerName}' not found on window`)); return; }
  if (!container.__initialized) {
    await container.init(buildSharedScope());
    container.__initialized = true;
  }
  const factory = await container.get(`./${componentName}`);
  resolve(factory().default);
}

export const CardMessage: React.FC<Props> = ({ resourceUri, uiMetadata }) => {
  const [status, setStatus] = useState<Status>('loading');
  const [cardData, setCardData] = useState<CardData | null>(null);
  const [token, setToken] = useState('');
  const [Component, setComponent] = useState<React.ComponentType<any> | null>(null);

  useEffect(() => {
    (async () => {
      try {
        const proxyRes = await fetch(`/resource-proxy?uri=${encodeURIComponent(resourceUri)}`);
        const data: CardData = await proxyRes.json();

        const tokenRes = await fetch('http://localhost:3001/api/token/exchange', { method: 'POST' });
        const { token: stargateToken } = await tokenRes.json();

        const Comp = await loadMFComponent(data.remoteEntryUrl, data.containerName, data.componentName);

        setCardData(data);
        setToken(stargateToken);
        setComponent(() => Comp);
        setStatus('ready');
      } catch (e) {
        console.error(e);
        setStatus('error');
      }
    })();
  }, [resourceUri]);

  const { width = 560, height = 420 } = uiMetadata?.['preferred-frame-size'] ?? {};

  if (status === 'loading') return <div style={{ padding: 12, color: '#888' }}>加载卡片中...</div>;
  if (status === 'error') return <div style={{ padding: 12, color: 'red' }}>卡片加载失败</div>;
  if (!Component || !cardData) return null;

  return (
    <div style={{ border: '1px solid #e0e0e0', borderRadius: 8, overflow: 'hidden', width, height }}>
      <Component {...cardData.props} token={token} />
    </div>
  );
};
```

**关键点**:
- 使用 Module Federation 加载远程组件
- 共享 React 和 React-DOM 以减少重复加载
- 通过 `/resource-proxy` 获取卡片数据
- 调用 `http://localhost:3001/api/token/exchange` 交换 Stargate token
- 支持动态容器初始化

---

### 2. App.tsx
**文件路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/App.tsx`

```typescript
import React, { useState, useRef, useEffect } from 'react';
import type { ChatMessage as ChatMessageType, MessagePart } from './types';
import { ChatMessage } from './components/ChatMessage';
import { EventLog } from './components/EventLog';
import { useEventLog } from './hooks/useEventLog';

let msgIdCounter = 0;
const newId = () => String(++msgIdCounter);

export const App: React.FC = () => {
  const [messages, setMessages] = useState<ChatMessageType[]>([]);
  const [input, setInput] = useState('');
  const [loading, setLoading] = useState(false);
  const bottomRef = useRef<HTMLDivElement>(null);

  const eventEntries = useEventLog([
    'http://localhost:3002/events',
    'http://localhost:3001/events',
  ]);

  useEffect(() => {
    const handler = (e: MessageEvent) => {
      if (e.data?.type === 'agent_submit') {
        setInput(`分析以下员工趋势数据：${JSON.stringify(e.data.payload?.data)}`);
      }
    };
    window.addEventListener('message', handler);
    return () => window.removeEventListener('message', handler);
  }, []);

  const sendMessage = async () => {
    const text = input.trim();
    if (!text || loading) return;
    setInput('');
    setLoading(true);

    const userMsg: ChatMessageType = {
      id: newId(),
      role: 'user',
      parts: [{ kind: 'text', text }],
    };
    setMessages((prev) => [...prev, userMsg]);

    try {
      const res = await fetch('/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: text }),
      });
      const data = await res.json();
      const parts: MessagePart[] = (data.parts ?? []).map((p: any) => {
        if (p.kind === 'text') return { kind: 'text' as const, text: p.text };
        if (p.kind === 'mcp_ui_resource') return {
          kind: 'mcp_ui_resource' as const,
          resourceUri: p.resourceUri,
          uiMetadata: p.uiMetadata,
        };
        return { kind: 'text' as const, text: JSON.stringify(p) };
      });
      const agentMsg: ChatMessageType = { id: newId(), role: 'agent', parts };
      setMessages((prev) => [...prev, agentMsg]);
    } catch {
      setMessages((prev) => [...prev, {
        id: newId(), role: 'agent',
        parts: [{ kind: 'text', text: '请求失败，请检查服务是否启动。' }],
      }]);
    } finally {
      setLoading(false);
      setTimeout(() => bottomRef.current?.scrollIntoView({ behavior: 'smooth' }), 100);
    }
  };

  return (
    <div style={{ display: 'flex', height: '100vh', fontFamily: 'system-ui, sans-serif' }}>
      <div style={{ flex: 1, display: 'flex', flexDirection: 'column', borderRight: '1px solid #e0e0e0' }}>
        <div style={{ padding: '12px 16px', borderBottom: '1px solid #e0e0e0', fontWeight: 600 }}>
          CodeFlicker x MCP-UI Demo
        </div>
        <div style={{ flex: 1, overflowY: 'auto', padding: 16 }}>
          {messages.map((m) => <ChatMessage key={m.id} message={m} />)}
          {loading && <div style={{ color: '#888', padding: '8px 0' }}>思考中...</div>}
          <div ref={bottomRef} />
        </div>
        <div style={{ padding: 12, borderTop: '1px solid #e0e0e0', display: 'flex', gap: 8 }}>
          <input
            style={{ flex: 1, padding: '8px 12px', borderRadius: 8, border: '1px solid #ccc', fontSize: 14 }}
            value={input}
            onChange={(e) => setInput(e.target.value)}
            onKeyDown={(e) => e.key === 'Enter' && sendMessage()}
            placeholder="输入消息，例如：查询快手历年员工人数趋势"
            disabled={loading}
          />
          <button
            style={{ padding: '8px 16px', borderRadius: 8, background: '#0084ff', color: '#fff', border: 'none', cursor: 'pointer' }}
            onClick={sendMessage}
            disabled={loading}
          >
            发送
          </button>
        </div>
      </div>
      <div style={{ width: 420, overflow: 'hidden' }}>
        <EventLog entries={eventEntries} />
      </div>
    </div>
  );
};
```

**关键点**:
- 主聊天界面，支持用户输入和 Agent 回复
- 左侧是聊天区域，右侧是事件日志
- 监听 `agent_submit` postMessage 事件用于卡片交互反馈
- 支持 `mcp_ui_resource` 类型的消息部分

---

### 3. types.ts
**文件路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/types.ts`

```typescript
export interface TextPart {
  kind: 'text';
  text: string;
}

export interface McpUiResourcePart {
  kind: 'mcp_ui_resource';
  resourceUri: string;
  uiMetadata?: { 'preferred-frame-size'?: { width: number; height: number } };
}

export type MessagePart = TextPart | McpUiResourcePart;

export interface ChatMessage {
  id: string;
  role: 'user' | 'agent';
  parts: MessagePart[];
}

export interface CardData {
  componentName: string;
  containerName: string;
  remoteEntryUrl: string;
  props: Record<string, unknown>;
}

export interface EventLogEntry {
  time: string;
  source: string;
  target: string;
  type: string;
  detail: string;
}
```

---

### 4. stargate_agent/main.py - mcp_resources_read 接口
**文件路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-agent/src/stargate_agent/main.py`

**关键部分 - mcp_resources_read 接口**:
```python
@app.get("/mcp/resources/read")
async def mcp_resources_read(uri: str):
    if not uri.startswith("ui://stargate/card/"):
        raise HTTPException(status_code=404, detail="Unknown resource URI")
    card_id = uri.removeprefix("ui://stargate/card/")
    inst = card_cache.get(card_id)
    if inst is None:
        raise HTTPException(status_code=404, detail="Card instance not found or expired")
    sse_logger.emit("SG-Agent", "CF-Agent", "mcp-resources/read", uri)
    return {
        "componentName": inst.component_name,
        "containerName": inst.container_name,
        "remoteEntryUrl": inst.remote_entry_url,
        "props": inst.props,
    }
```

**完整文件内容**:
```python
import os
import json
import httpx
from dotenv import load_dotenv
from fastapi import FastAPI, Header, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import StreamingResponse
from python_a2a import A2AServer, Message, TextContent, MessageRole, AgentCard

from stargate_agent import card_cache, llm, sse_logger

load_dotenv(dotenv_path=os.path.join(os.path.dirname(__file__), "../../../../.env"))

RESOURCE_CENTER_URL = os.environ.get("RESOURCE_CENTER_URL", "http://localhost:3003")
PORT = int(os.environ.get("SG_AGENT_PORT", 3001))
A2A_PORT = 3011

app = FastAPI(title="stargate-agent")
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

EMPLOYEE_DETAIL = {
    2019: {"year": 2019, "count": 7000, "note": "快速扩张期"},
    2020: {"year": 2020, "count": 10000, "note": "疫情期逆势增长"},
    2021: {"year": 2021, "count": 16000, "note": "业务多元化"},
    2022: {"year": 2022, "count": 22000, "note": "峰值"},
    2023: {"year": 2023, "count": 18000, "note": "降本增效"},
}

@app.get("/health")
async def health():
    return {"ok": True}

@app.get("/events")
async def events():
    return StreamingResponse(sse_logger.subscribe(), media_type="text/event-stream")

@app.get("/api/card-instance/{card_id}")
async def get_card_instance(card_id: str):
    inst = card_cache.get(card_id)
    if inst is None:
        raise HTTPException(status_code=404, detail="Card instance not found or expired")
    return {
        "componentName": inst.component_name,
        "containerName": inst.container_name,
        "remoteEntryUrl": inst.remote_entry_url,
        "props": inst.props,
    }

@app.post("/api/token/exchange")
async def token_exchange():
    return {"token": "mock-stargate-token-12345"}

@app.get("/api/employee/detail/{year}")
async def employee_detail(year: int, authorization: str = Header(default="")):
    if not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Missing token")
    detail = EMPLOYEE_DETAIL.get(year)
    if detail is None:
        raise HTTPException(status_code=404, detail="Year not found")
    return detail

@app.get("/mcp/resources/read")
async def mcp_resources_read(uri: str):
    if not uri.startswith("ui://stargate/card/"):
        raise HTTPException(status_code=404, detail="Unknown resource URI")
    card_id = uri.removeprefix("ui://stargate/card/")
    inst = card_cache.get(card_id)
    if inst is None:
        raise HTTPException(status_code=404, detail="Card instance not found or expired")
    sse_logger.emit("SG-Agent", "CF-Agent", "mcp-resources/read", uri)
    return {
        "componentName": inst.component_name,
        "containerName": inst.container_name,
        "remoteEntryUrl": inst.remote_entry_url,
        "props": inst.props,
    }


import asyncio
import concurrent.futures

_executor = concurrent.futures.ThreadPoolExecutor(max_workers=4)

def _run_async(coro):
    try:
        loop = asyncio.get_event_loop()
        if loop.is_running():
            future = _executor.submit(asyncio.run, coro)
            return future.result(timeout=30)
        return loop.run_until_complete(coro)
    except RuntimeError:
        return asyncio.run(coro)


async def _fetch_component_info():
    async with httpx.AsyncClient() as client:
        resp = await client.get(f"{RESOURCE_CENTER_URL}/api/components/EmployeeChart")
        return resp.json()


class StargateA2AServer(A2AServer):
    def handle_message(self, message: Message) -> Message:
        user_text = ""
        if hasattr(message.content, "text"):
            user_text = message.content.text

        sse_logger.emit("SG-Agent", "LLM", "llm-call", "tool selection")
        tool_name, tool_args = _run_async(llm.select_tool(user_text))

        if tool_name == "query_employee_trend":
            sse_logger.emit("SG-Agent", "ResourceCenter", "http", "GET /api/components/EmployeeChart")
            component_info = _run_async(_fetch_component_info())

            trend_data = [
                {"year": 2019, "count": 7000},
                {"year": 2020, "count": 10000},
                {"year": 2021, "count": 16000},
                {"year": 2022, "count": 22000},
                {"year": 2023, "count": 18000},
            ]
            card_id = card_cache.put(
                component_name=component_info["componentName"],
                container_name=component_info.get("containerName", component_info["componentName"]),
                remote_entry_url=component_info["remoteEntryUrl"],
                props={"data": trend_data},
            )
            sse_logger.emit("SG-Agent", "CF-Agent", "A2A Response", f"mcp_ui_resource card/{card_id}")

            response_data = {
                "text": f"已为您查询快手历年员工趋势数据，共 {len(trend_data)} 年记录。",
                "mcp_ui_resource": {
                    "kind": "mcp_ui_resource",
                    "resourceUri": f"ui://stargate/card/{card_id}",
                    "uiMetadata": {
                        "preferred-frame-size": {"width": 560, "height": 420}
                    },
                },
            }
            return Message(
                content=TextContent(text=json.dumps(response_data, ensure_ascii=False)),
                role=MessageRole.AGENT,
            )
        else:
            return Message(
                content=TextContent(text=json.dumps({"text": "抱歉，我目前只支持查询员工趋势数据。"}, ensure_ascii=False)),
                role=MessageRole.AGENT,
            )


def _start_a2a_flask():
    from flask import Flask
    flask_app = Flask(__name__)
    agent_card = AgentCard(
        name="stargate-agent",
        description="Stargate A2A Agent with MCP-UI support",
        url=f"http://localhost:{A2A_PORT}",
        version="0.1.0",
    )
    a2a_server = StargateA2AServer(agent_card=agent_card)
    a2a_server.setup_routes(flask_app)
    flask_app.run(host="0.0.0.0", port=A2A_PORT, debug=False, use_reloader=False)


def main():
    import uvicorn
    import threading

    t = threading.Thread(target=_start_a2a_flask, daemon=True)
    t.start()

    uvicorn.run(app, host="0.0.0.0", port=PORT)

if __name__ == "__main__":
    main()
```

---

### 5. EmployeeChart.tsx - 组件 props
**文件路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/employee-chart-card/src/EmployeeChart.tsx`

```typescript
import React, { useEffect, useRef } from 'react';
import * as echarts from 'echarts';

export interface EmployeeChartProps {
  data: { year: number; count: number }[];
  token: string;
  sgAgentBaseUrl?: string;
}

const EmployeeChart: React.FC<EmployeeChartProps> = ({
  data,
  token,
  sgAgentBaseUrl = 'http://localhost:3001',
}) => {
  const chartRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!chartRef.current) return;
    const chart = echarts.init(chartRef.current);
    chart.setOption({
      title: { text: '快手员工趋势' },
      xAxis: { type: 'category', data: data.map((d) => String(d.year)) },
      yAxis: { type: 'value' },
      series: [{ type: 'line', data: data.map((d) => d.count), smooth: true }],
    });
    return () => chart.dispose();
  }, [data]);

  const handleAnalyze = () => {
    window.parent.postMessage({ type: 'agent_submit', payload: { action: 'analyze_trend', data } }, '*');
  };

  const handleHoverYear = async (year: number) => {
    const res = await fetch(`${sgAgentBaseUrl}/api/employee/detail/${year}`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    const detail = await res.json();
    alert(`${year} 年详情：${JSON.stringify(detail)}`);
  };

  return (
    <div style={{ padding: 16 }}>
      <div ref={chartRef} style={{ width: 500, height: 300 }} />
      <div style={{ marginTop: 8, display: 'flex', gap: 8 }}>
        <button onClick={handleAnalyze}>分析趋势</button>
        {data.map((d) => (
          <button key={d.year} onClick={() => handleHoverYear(d.year)}>
            {d.year} 详情
          </button>
        ))}
      </div>
    </div>
  );
};

export default EmployeeChart;
```

**Props 说明**:
- `data`: 员工年份和数量数据数组
- `token`: Stargate 认证 token
- `sgAgentBaseUrl`: Stargate Agent 基础 URL (默认 http://localhost:3001)

---

### 6. sandbox_proxy.html
**文件路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/public/sandbox_proxy.html`

```html
<!DOCTYPE html>
<html>
  <head><meta charset="UTF-8" /></head>
  <body><script>
    window.addEventListener('message', (e) => {
      if (e.data && e.data.type === 'mcp-ui-proxy-init') {
        window.parent.postMessage({ type: 'mcp-ui-proxy-ready' }, '*');
      }
    });
  </script></body>
</html>
```

**说明**:
- 简单的沙箱代理页面
- 用于初始化 MCP-UI 沙箱通信
- 接收 `mcp-ui-proxy-init` 消息，回复 `mcp-ui-proxy-ready`

---

### 7. vite.config.ts
**文件路径**: `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/vite.config.ts`

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/chat': 'http://localhost:3002',
      '/resource-proxy': 'http://localhost:3002',
    },
  });
```

**关键配置**:
- 前端开发服务器运行在 3000 端口
- `/chat` 和 `/resource-proxy` 请求代理到 3002 (cf-agent 后端)

---

### 8. @mcp-ui/client TypeScript 类型定义

#### AppRenderer.d.ts (完整)
```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { CallToolRequest, CallToolResult, JSONRPCRequest, ListPromptsRequest, ListPromptsResult, ListResourcesRequest, ListResourcesResult, ListResourceTemplatesRequest, ListResourceTemplatesResult, LoggingMessageNotification, ReadResourceRequest, ReadResourceResult } from '@modelcontextprotocol/sdk/types.js';
import { AppBridge, McpUiMessageRequest, McpUiMessageResult, McpUiOpenLinkRequest, McpUiOpenLinkResult, McpUiSizeChangedNotification, McpUiToolInputPartialNotification, McpUiHostContext } from '@modelcontextprotocol/ext-apps/app-bridge';
import { SandboxConfig } from './AppFrame';

/**
 * Extra metadata passed to request handlers (from AppBridge).
 */
export type RequestHandlerExtra = Parameters<Parameters<AppBridge['setRequestHandler']>[1]>[1];

/**
 * Handle to access AppRenderer methods for sending notifications to the Guest UI.
 * Obtained via ref on AppRenderer.
 */
export interface AppRendererHandle {
    /** Notify the Guest UI that the server's tool list has changed */
    sendToolListChanged: () => void;
    /** Notify the Guest UI that the server's resource list has changed */
    sendResourceListChanged: () => void;
    /** Notify the Guest UI that the server's prompt list has changed */
    sendPromptListChanged: () => void;
    /** Notify the Guest UI that the resource is being torn down / cleaned up */
    teardownResource: () => void;
}

/**
 * Props for the AppRenderer component.
 */
export interface AppRendererProps {
    /** MCP client connected to the server providing the tool. Omit to disable automatic MCP forwarding and use custom handlers instead. */
    client?: Client;
    /** Name of the MCP tool to render UI for */
    toolName: string;
    /** Sandbox configuration */
    sandbox: SandboxConfig;
    /** Optional pre-fetched resource URI. If not provided, will be fetched via getToolUiResourceUri() */
    toolResourceUri?: string;
    /** Optional pre-fetched HTML. If provided, skips all resource fetching */
    html?: string;
    /** Optional input arguments to pass to the tool UI once it's ready */
    toolInput?: Record<string, unknown>;
    /** Optional result from tool execution to pass to the tool UI once it's ready */
    toolResult?: CallToolResult;
    /** Partial/streaming tool input to send to the guest UI */
    toolInputPartial?: McpUiToolInputPartialNotification['params'];
    /** Set to true to notify the guest UI that the tool execution was cancelled */
    toolCancelled?: boolean;
    /** Host context (theme, viewport, locale, etc.) to pass to the guest UI */
    hostContext?: McpUiHostContext;
    /** Handler for open-link requests from the guest UI */
    onOpenLink?: (params: McpUiOpenLinkRequest['params'], extra: RequestHandlerExtra) => Promise<McpUiOpenLinkResult>;
    /** Handler for message requests from the guest UI */
    onMessage?: (params: McpUiMessageRequest['params'], extra: RequestHandlerExtra) => Promise<McpUiMessageResult>;
    /** Handler for logging messages from the guest UI */
    onLoggingMessage?: (params: LoggingMessageNotification['params']) => void;
    /** Handler for size change notifications from the guest UI */
    onSizeChanged?: (params: McpUiSizeChangedNotification['params']) => void;
    /** Callback invoked when an error occurs during setup or message handling */
    onError?: (error: Error) => void;
    /**
     * Handler for tools/call requests from the guest UI.
     * If provided, overrides the automatic forwarding to the MCP client.
     */
    onCallTool?: (params: CallToolRequest['params'], extra: RequestHandlerExtra) => Promise<CallToolResult>;
    /**
     * Handler for resources/list requests from the guest UI.
     * If provided, overrides the automatic forwarding to the MCP client.
     */
    onListResources?: (params: ListResourcesRequest['params'], extra: RequestHandlerExtra) => Promise<ListResourcesResult>;
    /**
     * Handler for resources/templates/list requests from the guest UI.
     * If provided, overrides the automatic forwarding to the MCP client.
     */
    onListResourceTemplates?: (params: ListResourceTemplatesRequest['params'], extra: RequestHandlerExtra) => Promise<ListResourceTemplatesResult>;
    /**
     * Handler for resources/read requests from the guest UI.
     * If provided, overrides the automatic forwarding to the MCP client.
     */
    onReadResource?: (params: ReadResourceRequest['params'], extra: RequestHandlerExtra) => Promise<ReadResourceResult>;
    /**
     * Handler for prompts/list requests from the guest UI.
     * If provided, overrides the automatic forwarding to the MCP client.
     */
    onListPrompts?: (params: ListPromptsRequest['params'], extra: RequestHandlerExtra) => Promise<ListPromptsResult>;
    /**
     * Handler for JSON-RPC requests from the guest UI that don't match any
     * built-in handler (e.g., experimental methods like "x/clipboard/write",
     * or standard MCP methods not yet in the Apps spec like "sampling/createMessage").
     *
     * This is wired to AppBridge's `fallbackRequestHandler` from the MCP SDK Protocol class.
     * It receives the full JSON-RPC request and should return a result object or throw
     * a McpError for unsupported methods.
     *
     * @example
     * ```tsx
     * <AppRenderer
     *   onFallbackRequest={async (request, extra) => {
     *     switch (request.method) {
     *       case 'x/clipboard/write':
     *         await navigator.clipboard.writeText(request.params?.text);
     *         return { success: true };
     *       case 'sampling/createMessage':
     *         return mcpClient.createMessage(request.params);
     *       default:
     *         throw new McpError(ErrorCode.MethodNotFound, `Unknown method: ${request.method}`);
     *     }
     *   }}
     * />
     * ```
     */
    onFallbackRequest?: (request: JSONRPCRequest, extra: RequestHandlerExtra) => Promise<Record<string, unknown>>;
}

/**
 * React component that renders an MCP tool's custom UI in a sandboxed iframe.
 *
 * This component manages the complete lifecycle of an MCP Apps tool:
 * 1. Creates AppBridge for MCP communication
 * 2. Fetches the tool's UI resource (HTML) if not provided
 * 3. Delegates rendering to AppFrame
 * 4. Handles UI actions (intents, link opening, prompts, notifications)
 *
 * For lower-level control or when you already have the HTML content,
 * use the AppFrame component directly.
 *
 * @example Basic usage
 * ```tsx
 * <AppRenderer
 *   sandbox={{ url: new URL('http://localhost:8765/sandbox_proxy.html') }}
 *   client={mcpClient}
 *   toolName="create-chart"
 *   toolInput={{ data: [1, 2, 3], type: 'bar' }}
 *   onOpenLink={async ({ url }) => window.open(url)}
 *   onError={(error) => console.error('UI Error:', error)}
 * />
 * ```
 *
 * @example With pre-fetched HTML (skips resource fetching)
 * ```tsx
 * <AppRenderer
 *   sandbox={{ url: sandboxUrl }}
 *   client={mcpClient}
 *   toolName="my-tool"
 *   html={preloadedHtml}
 *   toolInput={args}
 * />
 * ```
 *
 * @example Using ref to access AppBridge methods
 * ```tsx
 * const appRef = useRef<AppRendererHandle>(null);
 *
 * // Notify guest UI when tools change
 * useEffect(() => {
 *   appRef.current?.sendToolListChanged();
 * }, [toolsVersion]);
 *
 * <AppRenderer ref={appRef} ... />
 * ```
 *
 * @example With custom MCP request handlers (no client)
 * ```tsx
 * <AppRenderer
 *   // client omitted - use toolResourceUri + onReadResource to fetch HTML
 *   sandbox={{ url: sandboxUrl }}
 *   toolName="my-tool"
 *   toolResourceUri="ui://my-server/my-tool"
 *   onReadResource={async ({ uri }) => {
 *     // Proxy to your MCP client (e.g., in a different context)
 *     return myMcpProxy.readResource({ uri });
 *   }}
 *   onCallTool={async (params) => {
 *     // Custom tool call handling with caching/filtering
 *     return myCustomToolCall(params);
 *   }}
 *   onListResources={async () => {
 *     // Aggregate resources from multiple servers
 *     return { resources: [...server1Resources, ...server2Resources] };
 *   }}
 * />
 * ```
 */
export declare const AppRenderer: import('react').ForwardRefExoticComponent<AppRendererProps & import('react').RefAttributes<AppRendererHandle>>;
```

#### index.d.ts (导出列表)
```typescript
export { getUIResourceMetadata, getResourceMetadata } from './utils/metadataUtils';
export { isUIResource } from './utils/isUIResource';
export { type ClientCapabilitiesWithExtensions, UI_EXTENSION_NAME, UI_EXTENSION_CONFIG, UI_EXTENSION_CAPABILITIES, } from './capabilities';
export { AppRenderer, type AppRendererProps, type AppRendererHandle, type RequestHandlerExtra, } from './components/AppRenderer';
export { AppFrame, type AppFrameProps, type SandboxConfig, type AppInfo, } from './components/AppFrame';
export { AppBridge, PostMessageTransport, type McpUiHostContext, } from '@modelcontextprotocol/ext-apps/app-bridge';
export type { JSONRPCRequest } from '@modelcontextprotocol/sdk/types.js';
export type { UIResourceMetadata } from './types';
```

#### types.d.ts
```typescript
export declare const UIMetadataKey: {
    readonly PREFERRED_FRAME_SIZE: "preferred-frame-size";
    readonly INITIAL_RENDER_DATA: "initial-render-data";
};
export declare const UI_METADATA_PREFIX = "mcpui.dev/ui-";
export type UIResourceMetadata = {
    [UIMetadataKey.PREFERRED_FRAME_SIZE]?: [string, string];
    [UIMetadataKey.INITIAL_RENDER_DATA]?: Record<string, unknown>;
};
```

#### AppFrame.d.ts
```typescript
import { CallToolResult, Implementation } from '@modelcontextprotocol/sdk/types.js';
import { AppBridge, McpUiSizeChangedNotification, McpUiResourceCsp, McpUiAppCapabilities } from '@modelcontextprotocol/ext-apps/app-bridge';

export interface AppInfo {
    /** Guest app's name and version */
    appVersion?: Implementation;
    /** Guest app's declared capabilities */
    appCapabilities?: McpUiAppCapabilities;
}

export interface SandboxConfig {
    /** URL to the sandbox proxy HTML */
    url: URL;
    /** Override iframe sandbox attribute (default: "allow-scripts allow-same-origin allow-forms") */
    permissions?: string;
    /**
     * CSP metadata for the sandbox.
     *
     * This CSP is passed to the sandbox proxy in two ways:
     * 1. Via URL query parameter (`?csp=<json>`) - allows servers that support it to set
     *    CSP via HTTP headers (tamper-proof, recommended)
     * 2. Via postMessage after sandbox loads - fallback for proxies that don't parse query params
     *
     * For maximum security, use a proxy server that reads the `csp` query parameter and sets
     * Content-Security-Policy HTTP headers accordingly.
     *
     * @see https://github.com/modelcontextprotocol/ext-apps/pull/234
     */
    csp?: McpUiResourceCsp;
}

export interface AppFrameProps {
    /** Pre-fetched HTML content to render in the sandbox */
    html: string;
    /** Sandbox configuration */
    sandbox: SandboxConfig;
    /** Pre-configured AppBridge for MCP communication (required) */
    appBridge: AppBridge;
    /** Callback when guest reports size change */
    onSizeChanged?: (params: McpUiSizeChangedNotification['params']) => void;
    /** Callback when app initialization completes, with app info */
    onInitialized?: (appInfo: AppInfo) => void;
    /** Tool input arguments to send when app initializes */
    toolInput?: Record<string, unknown>;
    /** Tool result to send when app initializes */
    toolResult?: CallToolResult;
    /** Callback when an error occurs */
    onError?: (error: Error) => void;
}

export declare const AppFrame: (props: AppFrameProps) => import("react/jsx-runtime").JSX.Element;
```

---

## 系统架构总结

**核心流程**:

1. **Frontend (3000)** -> 用户输入消息
2. **CF-Agent (3002)** -> 接收消息，调用 LLM + 资源中心
3. **Stargate-Agent (3001)** -> 处理 MCP 协议，生成卡片 ID，缓存卡片信息
4. **Frontend** -> 通过 `/resource-proxy` 获取卡片数据
5. **CardMessage** -> 通过 Module Federation 动态加载远程组件
6. **EmployeeChart** -> 渲染可视化，支持交互和返回数据

**关键接口**:
- `GET /mcp/resources/read?uri=ui://stargate/card/{cardId}` - 读取卡片资源
- `POST /api/token/exchange` - 交换认证 token
- `GET /api/employee/detail/{year}` - 获取员工年度详情
- `/chat` - 聊天接口
- `/resource-proxy` - 资源代理接口

所有文件和类型定义已完整提供。
