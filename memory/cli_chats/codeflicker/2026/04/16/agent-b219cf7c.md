---
session_id: agent-b219cf7c
date: 2026-04-16
time_start: 2026-04-16T04:02:50.016Z
time_end: 2026-04-16T04:03:22.902Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: main
---

[2026-04-16 12:02:50] [user]: 探索 /Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-agent/src/codeflicker_agent/ 目录下所有文件，重点了解：
1. main.py 的完整内容 - 特别是 /chat-stream、/a2a-tool-call、/resource-proxy 端点的实现
2. 是否有 /events 端点
3. a2a_stream_client.py 的内容
4. tool_protocol.py 的内容
5. 后端返回的 result 数据格式是什么

请返回每个关键文件的完整内容。
[2026-04-16 12:02:54] [assistant]: 我来帮你探索这个目录。让我先列出所有文件，然后并行读取关键文件。
[2026-04-16 12:02:58] [assistant]: 现在让我读取其他相关文件：
[2026-04-16 12:03:22] [assistant]: 现在我已经读取了所有相关文件。让我整理一份完整的报告。

---

## 目录结构

```
codeflicker_agent/
├── __init__.py
├── a2a_parts.py
├── a2a_stream_client.py
├── extension_negotiation.py
├── llm.py
├── main.py
├── sse_logger.py
└── tool_protocol.py
```

---

## 1. main.py 完整内容

```python
"""CodeFlicker Agent - CF-Agent with a2a-sdk"""
import os
import json
from typing import AsyncGenerator
from dotenv import load_dotenv
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import StreamingResponse, JSONResponse

from codeflicker_agent import llm
from codeflicker_agent.extension_negotiation import validate_extensions, ExtensionNegotiationError
from codeflicker_agent.a2a_stream_client import call_sg_agent_streaming
from codeflicker_agent.tool_protocol import (
    make_tool_request_message,
    parse_tool_response_message,
    new_request_id,
)

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

current_mode: str = "endpoint"


@app.get("/health")
async def health():
    return {"ok": True}


@app.post("/mode")
async def set_mode(request: Request):
    global current_mode
    body = await request.json()
    mode = body.get("mode", "endpoint")
    if mode not in ("endpoint", "mcp"):
        return JSONResponse({"error": "mode must be 'endpoint' or 'mcp'"}, status_code=400)
    current_mode = mode
    return {"mode": current_mode}


@app.get("/mode")
async def get_mode():
    return {"mode": current_mode}


async def validate_sg_extensions() -> bool:
    """Validate SG-Agent has required extensions"""
    try:
        import httpx
        from a2a.client.card_resolver import A2ACardResolver
        async with httpx.AsyncClient() as client:
            resolver = A2ACardResolver(client, SG_AGENT_A2A_URL)
            card = await resolver.get_agent_card()
            validate_extensions(card)
        return True
    except ExtensionNegotiationError as e:
        print(f"Extension validation failed: {e}")
        return False
    except Exception as e:
        print(f"Failed to resolve agent card: {e}")
        return False


@app.post("/chat-stream")
async def chat_stream(request: Request):
    """SSE streaming endpoint for chat messages"""
    body = await request.json()
    user_message: str = body.get("message", "")
    mode: str = body.get("mode", current_mode)

    async def generate() -> AsyncGenerator[str, None]:
        try:
            # Validate extensions first
            if not await validate_sg_extensions():
                error_data = {"type": "error", "code": -32001, "message": "Extension negotiation failed"}
                yield f"data: {json.dumps(error_data)}\n\n"
                return

            # Detect intent
            intent = await llm.detect_intent(user_message)

            if intent != "query_data":
                result = {"parts": [{"kind": "text", "text": "您好！我是 CodeFlicker 助手，可以帮您查询快手员工趋势等数据。"}]}
                yield f"data: {json.dumps({'type': 'complete', 'result': result})}\n\n"
                return

            # Stream from SG-Agent
            async for event in call_sg_agent_streaming(SG_AGENT_A2A_URL, user_message, mode):
                yield f"data: {json.dumps(event)}\n\n"

        except Exception as e:
            error_data = {"type": "error", "code": -32000, "message": str(e)}
            yield f"data: {json.dumps(error_data)}\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")


@app.post("/a2a-tool-call")
async def a2a_tool_call(request: Request):
    """Handle tool calls via A2A protocol"""
    body = await request.json()
    tool_name = body.get("toolName", "")
    arguments = body.get("arguments", {})
    
    try:
        # Validate extensions
        if not await validate_sg_extensions():
            return JSONResponse(
                {"error": {"code": -32001, "message": "Extension negotiation failed"}},
                status_code=400
            )

        request_id = new_request_id()
        
        # Create tool request message
        message = make_tool_request_message(request_id, tool_name, arguments)
        
        # Send via A2A streaming
        result = None
        async for event in call_sg_agent_streaming(SG_AGENT_A2A_URL, "", "endpoint"):
            if event.get("type") == "complete":
                # Parse tool response from result
                result_data = event.get("result", {})
                # For now, return the full result
                result = result_data
                break
            elif event.get("type") == "error":
                return JSONResponse(
                    {"error": {"code": event.get("code", -32000), "message": event.get("message", "")}},
                    status_code=500
                )
        
        if result is None:
            return JSONResponse(
                {"error": {"code": -32000, "message": "No response from agent"}},
                status_code=500
            )
        
        return JSONResponse({"toolResult": result.get("toolResult", result)})
        
    except Exception as e:
        return JSONResponse(
            {"error": {"code": -32000, "message": str(e)}},
            status_code=500
        )


@app.get("/resource-proxy")
async def resource_proxy(uri: str, source: str = "host"):
    """Proxy resource requests to SG-Agent"""
    import httpx
    
    if uri.startswith("ui://stargate/"):
        async with httpx.AsyncClient() as client:
            resp = await client.get(
                f"{SG_AGENT_BASE_URL}/mcp/resources/read",
                params={"uri": uri},
            )
            if resp.status_code == 200:
                return JSONResponse(resp.json())
            return JSONResponse({"error": "Resource not found"}, status_code=resp.status_code)
    
    return JSONResponse({"error": "Unknown resource host"}, status_code=404)


def main():
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=PORT)


if __name__ == "__main__":
    main()
```

---

## 2. 关于 /events 端点

**结论：没有 /events 端点**

但是存在 `sse_logger.py` 模块，它提供了 `subscribe()` 函数用于 SSE 事件订阅。这个功能目前**没有在 main.py 中暴露为 HTTP 端点**。如果要添加，需要：

```python
@app.get("/events")
async def events():
    from codeflicker_agent.sse_logger import subscribe
    return StreamingResponse(subscribe(), media_type="text/event-stream")
```

---

## 3. a2a_stream_client.py 完整内容

```python
"""A2A Streaming Client for connecting to SG-Agent (simplified HTTP version)"""
import json
import uuid
from typing import AsyncGenerator, Optional
import httpx

from codeflicker_agent.a2a_parts import make_agent_request_message


class A2AStreamClient:
    """Client for streaming A2A communication with SG-Agent"""

    def __init__(self, endpoint_url: str):
        self.endpoint_url = endpoint_url

    async def send_message_stream(
        self,
        text: str,
        mode: str = "endpoint"
    ) -> AsyncGenerator[dict, None]:
        """
        Send message and receive streaming response via HTTP SSE.
        Yields events: status updates and final result.
        """
        # Build A2A JSON-RPC request
        message = make_agent_request_message(text, mode)
        
        # Convert to dict
        message_dict = {
            "messageId": message.message_id,
            "role": message.role,
            "parts": []
        }
        for part in message.parts:
            content = part.root if hasattr(part, 'root') else part
            if hasattr(content, 'kind') and content.kind == 'data':
                message_dict["parts"].append({
                    "kind": "data",
                    "data": content.data,
                    "metadata": content.metadata
                })
        
        request_payload = {
            "jsonrpc": "2.0",
            "id": str(uuid.uuid4()),
            "method": "message/stream",
            "params": {
                "message": message_dict,
                "metadata": {}
            }
        }
        
        try:
            async with httpx.AsyncClient() as client:
                async with client.stream(
                    "POST",
                    f"{self.endpoint_url}",
                    json=request_payload,
                    headers={"Content-Type": "application/json"},
                    timeout=60.0
                ) as response:
                    response.raise_for_status()
                    
                    async for line in response.aiter_lines():
                        if line.startswith("data: "):
                            try:
                                event_data = json.loads(line[6:])
                                yield self._convert_event(event_data)
                            except json.JSONDecodeError:
                                continue
                        elif line.strip() == "":
                            continue
                    
        except Exception as e:
            yield {
                "type": "error",
                "code": -32000,
                "message": f"Streaming error: {str(e)}"
            }

    def _convert_event(self, event: dict) -> dict:
        """Convert A2A JSON-RPC response to dict format for frontend"""
        # Check for error
        if "error" in event:
            return {
                "type": "error",
                "code": event["error"].get("code", -32000),
                "message": event["error"].get("message", "Unknown error"),
            }
        
        result = event.get("result", {})
        
        # Check event type based on kind field
        kind = result.get("kind", "")
        
        if kind == "status-update":
            status = result.get("status", {})
            message = status.get("message", {})
            parts = message.get("parts", [])
            text = ""
            for part in parts:
                if part.get("kind") == "text":
                    text = part.get("text", "")
                    break
            
            return {
                "type": "status",
                "state": status.get("state", "unknown"),
                "message": text,
                "final": result.get("final", False),
            }
        elif kind == "message":
            # Final response message
            parts = result.get("parts", [])
            data = None
            for part in parts:
                if part.get("kind") == "data":
                    data = part.get("data", {})
                    break
            
            return {
                "type": "complete",
                "result": data or {"text": "No data"},
            }
        elif kind == "task":
            return {
                "type": "task",
                "task_id": result.get("id", ""),
                "state": result.get("status", {}).get("state", "unknown"),
            }
        else:
            return {
                "type": "unknown",
                "event_type": kind or "unknown",
            }


async def call_sg_agent_streaming(
    endpoint_url: str,
    text: str,
    mode: str = "endpoint"
) -> AsyncGenerator[dict, None]:
    """Convenience function for streaming calls"""
    client = A2AStreamClient(endpoint_url)
    async for event in client.send_message_stream(text, mode):
        yield event
```

---

## 4. tool_protocol.py 完整内容

```python
"""Tool Protocol for A2A-based tool calling"""
import uuid
from typing import Optional
from a2a.types import Message, DataPart, TextPart


TOOL_REQUEST_SCHEMA = "https://stargate.example.com/schemas/tool-request-v1"
TOOL_RESPONSE_SCHEMA = "https://stargate.example.com/schemas/tool-response-v1"


def make_tool_request_message(
    request_id: str,
    tool_name: str,
    arguments: dict
) -> Message:
    """Create A2A message with ToolRequestPart"""
    return Message(
        messageId=str(uuid.uuid4()),
        role="user",
        parts=[
            DataPart(
                data={
                    "kind": "tool_request",
                    "id": request_id,
                    "toolName": tool_name,
                    "arguments": arguments
                },
                metadata={"schema": TOOL_REQUEST_SCHEMA}
            )
        ]
    )


def parse_tool_response_message(message: Message, expected_request_id: str) -> dict:
    """Parse A2A message to extract tool response"""
    for part in message.parts:
        content = part.root if hasattr(part, 'root') else part
        
        if isinstance(content, DataPart):
            data = content.data
            if data.get("kind") == "tool_response":
                if data.get("requestId") != expected_request_id:
                    raise ValueError(
                        f"requestId mismatch: expected {expected_request_id!r}, "
                        f"got {data.get('requestId')!r}"
                    )
                return data.get("result", {})
    
    raise ValueError("No tool_response found in message")


def new_request_id() -> str:
    """Generate new unique request ID"""
    return f"req-{uuid.uuid4().hex[:12]}"
```

---

## 5. 后端返回的 result 数据格式

根据代码分析，后端返回的数据格式有以下几种类型：

### 5.1 完整响应
```json
{
  "type": "complete",
  "result": {
    "parts": [
      {"kind": "text", "text": "响应文本"}
    ]
  }
}
```

### 5.2 状态更新
```json
{
  "type": "status",
  "state": "working" | "input-required" | "completed" | "unknown",
  "message": "状态描述文本",
  "final": false
}
```

### 5.3 任务创建
```json
{
  "type": "task",
  "task_id": "task-uuid",
  "state": "running"
}
```

### 5.4 错误响应
```json
{
  "type": "error",
  "code": -32000,
  "message": "错误信息"
}
```

### 5.5 A2A 协议原生格式（a2a_stream_client.py 中转换前）

**status-update 类型:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "kind": "status-update",
    "status": {
      "state": "working",
      "message": {
        "parts": [{"kind": "text", "text": "正在处理..."}]
      }
    },
    "final": false
  }
}
```

**message 类型:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "kind": "message",
    "parts": [
      {"kind": "data", "data": {...}}
    ]
  }
}
```

---

## 总结

| 项目 | 状态 |
|------|------|
| `/chat-stream` 端点 | 已实现 - SSE 流式聊天 |
| `/a2a-tool-call` 端点 | 已实现 - A2A 工具调用 |
| `/resource-proxy` 端点 | 已实现 - 资源代理 |
| `/events` 端点 | **未实现** - 需添加 |
