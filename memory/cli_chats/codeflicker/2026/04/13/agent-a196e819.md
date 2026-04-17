---
session_id: agent-a196e819
date: 2026-04-13
time_start: 2026-04-13T03:44:58.026Z
time_end: 2026-04-13T03:45:58.661Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: 
---

[2026-04-13 11:44:58] [user]: 快速梳理 /Users/lianzimeng/working/A2A-mcpUI 项目，返回：
1. packages/ 下有哪些服务，各自端口
2. codeflicker-agent/src/codeflicker_agent/main.py 完整内容
3. codeflicker-frontend/src/components/CardMessage.tsx 完整内容
4. codeflicker-frontend/src/App.tsx 完整内容
只需返回原始代码，不需要分析。
[2026-04-13 11:45:02] [assistant]: 我来帮你快速梳理这个项目。让我并行获取所有需要的信息。
[2026-04-13 11:45:07] [assistant]: 看起来这些文件不在我给的路径中。让我再次查看目录结构并重新定位文件：
[2026-04-13 11:45:14] [assistant]: 现在让我读取各个 package.json 文件来获取端口信息，以及查找正确的代码文件：
[2026-04-13 11:45:22] [assistant]: 现在让我读取这些文件：
[2026-04-13 11:45:27] [assistant]: 让我获取更多端口信息：
[2026-04-13 11:45:32] [assistant]: 完美！现在让我获取 stargate-mcp-ui-server 的端口配置：
[2026-04-13 11:45:37] [assistant]: 让我查看 resource-center-mock 的完整源码：
[2026-04-13 11:45:58] [assistant]: 完美！现在我已经获得了所有需要的信息。让我整理一下返回给你：

## 项目梳理结果

### 1. packages/ 下的服务及端口

| 服务名 | 端口 | 说明 |
|------|------|------|
| **codeflicker-frontend** | 3000 | 前端应用 (Vite) |
| **codeflicker-agent** | 3002 | CodeFlicker代理 (FastAPI) |
| **resource-center-mock** | 3003 | 资源中心模拟服务 (Express) |
| **employee-chart-card** | 3004 | 员工图表卡片 (Webpack Module Federation) |
| **stargate-mcp-ui-server** | 3005 | MCP UI服务器 (FastMCP) |
| **stargate-agent** | 3001 | Stargate代理主服务 (FastAPI) |
| **stargate-agent (A2A)** | 3011 | Stargate代理A2A服务 (Flask) |

---

### 2. codeflicker-agent/src/codeflicker_agent/main.py 完整内容

```python
import os
import json
import httpx
from dotenv import load_dotenv
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import StreamingResponse, JSONResponse
from python_a2a import A2AClient, Message, TextContent, MessageRole

from codeflicker_agent import llm, sse_logger

load_dotenv(dotenv_path=os.path.join(os.path.dirname(__file__), "../../../../.env"))

SG_AGENT_A2A_URL = os.environ.get("SG_AGENT_A2A_URL", "http://localhost:3011")
SG_AGENT_BASE_URL = os.environ.get("SG_AGENT_BASE_URL", "http://localhost:3001")
PORT = int(os.environ.get("CF_AGENT_PORT", 3002))

app = FastAPI(title="codeflicker-agent")
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/health")
async def health():
    return {"ok": True}

@app.get("/events")
async def events():
    return StreamingResponse(sse_logger.subscribe(), media_type="text/event-stream")

@app.post("/chat")
async def chat(request: Request):
    body = await request.json()
    user_message: str = body.get("message", "")

    sse_logger.emit("Frontend", "CF-Agent", "chat", user_message[:50])
    sse_logger.emit("CF-Agent", "LLM", "llm-call", "intent detection")

    intent = await llm.detect_intent(user_message)

    if intent == "query_data":
        sse_logger.emit("CF-Agent", "SG-Agent", "A2A Task", user_message[:50])
        client = A2AClient(endpoint_url=SG_AGENT_A2A_URL)
        response_msg = client.send_message(
            Message(
                content=TextContent(text=user_message),
                role=MessageRole.USER,
            )
        )

        parts = []
        if hasattr(response_msg, "content") and hasattr(response_msg.content, "text"):
            raw = response_msg.content.text
            try:
                data = json.loads(raw)
                if isinstance(data, dict):
                    if "text" in data:
                        parts.append({"kind": "text", "text": data["text"]})
                    if "mcp_ui_resource" in data:
                        parts.append(data["mcp_ui_resource"])
                else:
                    parts.append({"kind": "text", "text": raw})
            except json.JSONDecodeError:
                parts.append({"kind": "text", "text": raw})

        sse_logger.emit("SG-Agent", "CF-Agent", "A2A Response", f"{len(parts)} parts")
        return JSONResponse({"parts": parts})
    else:
        return JSONResponse({
            "parts": [{"kind": "text", "text": "您好！我是 CodeFlicker 助手，可以帮您查询快手员工趋势等数据。"}]
        })

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

def main():
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=PORT)

if __name__ == "__main__":
    main()
```

---

### 3. codeflicker-frontend/src/components/CardMessage.tsx 完整内容

```typescript
import React, { useState } from 'react';
import { AppRenderer } from '@mcp-ui/client';
import type { McpUiResourcePart } from '../types';

interface Props {
  resourceUri: string;
  uiMetadata?: McpUiResourcePart['uiMetadata'];
  onMessage?: (text: string) => void;
  onLayout?: () => void;
}

export const CardMessage: React.FC<Props> = ({ resourceUri, uiMetadata, onMessage, onLayout }) => {
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
        toolName="query_employee_trend"
        sandbox={{ url: sandboxUrl }}
        toolResourceUri={resourceUri}
        onReadResource={async ({ uri }) => {
          const res = await fetch(`/resource-proxy?uri=${encodeURIComponent(uri)}`);
          return res.json();
        }}
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
        onError={(e) => console.error('[AppRenderer]', e)}
      />
    </div>
  );
};
```

---

### 4. codeflicker-frontend/src/App.tsx 完整内容

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
    bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  const sendMessage = async (text?: string) => {
    const finalText = (text ?? input).trim();
    if (!finalText || loading) return;
    setInput('');
    setLoading(true);

    const userMsg: ChatMessageType = {
      id: newId(),
      role: 'user',
      parts: [{ kind: 'text', text: finalText }],
    };
    setMessages((prev) => [...prev, userMsg]);

    try {
      const res = await fetch('/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: finalText }),
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
          {messages.map((m) => (
            <ChatMessage
              key={m.id}
              message={m}
              onCardMessage={(text) => sendMessage(text)}
              onLayout={() => bottomRef.current?.scrollIntoView({ behavior: 'smooth' })}
            />
          ))}
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
            onClick={() => sendMessage()}
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
