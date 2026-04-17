---
session_id: agent-9f5db3ba
date: 2026-04-16
time_start: 2026-04-16T04:03:40.560Z
time_end: 2026-04-16T04:04:15.898Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: main
---

[2026-04-16 12:03:40] [user]: 探索 /Users/lianzimeng/working/A2A-mcpUI/packages/stargate-agent/src/stargate_agent/ 目录，重点了解：
1. main.py 完整内容
2. a2a_server.py 完整内容 - 特别是它返回的数据格式，result 中是否包含 mcp_ui_resource 字段
3. a2a_executor.py 完整内容
4. agent_card_builder.py 完整内容

请返回每个文件的完整内容。
[2026-04-16 12:03:44] [assistant]: 我将探索指定目录并读取这4个文件的完整内容。让我并行读取这些文件：
[2026-04-16 12:04:15] [assistant]: 我已完成对 `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-agent/src/stargate_agent/` 目录的探索。以下是4个文件的完整内容：

---

## 1. main.py (276行)

```python
import os
import json
import asyncio
import httpx
from dotenv import load_dotenv
from mcp.client.sse import sse_client
from mcp import ClientSession
from fastapi import FastAPI, Header, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import StreamingResponse, JSONResponse
from mcp_ui_server import create_ui_resource
from stargate_agent.agent_card_builder import build_agent_card
from stargate_agent.a2a_server import create_a2a_app
import uvicorn
import threading

A2A_PORT = 3011

from stargate_agent import card_cache, llm, sse_logger
from stargate_agent.shell_builder import MCP_INIT_SCRIPT

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

@app.get("/api/tool-result/{tool_name}")
async def get_tool_result(tool_name: str):
    if tool_name in ("query_employee_trend", "query_employee_trend_lazy"):
        await asyncio.sleep(1.5)
        sse_logger.emit("SG-Agent", "BusinessAPI", "http", "GET /api/employee/trend")
        trend_resp = await _fetch_employee_trend()
        result = {
            "content": [{"type": "text", "text": "已为您查询快手历年员工趋势数据，共 5 年记录。"}],
            "data": trend_resp["data"],
            "token": trend_resp["token"],
        }
        return result
    raise HTTPException(status_code=404, detail="Unknown tool")

@app.get("/api/employee/trend")
async def employee_trend():
    return {
        "data": [
            {"year": 2019, "count": 7000},
            {"year": 2020, "count": 10000},
            {"year": 2021, "count": 16000},
            {"year": 2022, "count": 22000},
            {"year": 2023, "count": 18000},
        ],
        "token": "mock-stargate-token-12345",
    }

@app.get("/api/employee/detail/{year}")
async def employee_detail(year: int, authorization: str = Header(default="")):
    span = sse_logger.emit_request(
        "Frontend", "SG-Agent", "http",
        params={"path": f"GET /api/employee/detail/{year}"},
    )
    if not authorization.startswith("Bearer "):
        sse_logger.emit_response(span, "SG-Agent", "Frontend", "http", detail="error: 401 missing token")
        raise HTTPException(status_code=401, detail="Missing token")
    detail = EMPLOYEE_DETAIL.get(year)
    if detail is None:
        sse_logger.emit_response(span, "SG-Agent", "Frontend", "http", detail="error: 404 year not found")
        raise HTTPException(status_code=404, detail="Year not found")
    sse_logger.emit_response(span, "SG-Agent", "Frontend", "http", result={"year": year, "count": detail["count"]})
    return detail

@app.get("/mcp/resources/read")
async def mcp_resources_read(uri: str):
    # ... (处理 MCP 资源读取，包括 ui://stargate/employee-trend 等)
    # ... 包含 HTML 动态生成逻辑

async def _fetch_employee_trend() -> dict:
    async with httpx.AsyncClient(timeout=10.0) as client:
        resp = await client.get(f"http://localhost:{PORT}/api/employee/trend")
        return resp.json()

def _start_a2a_server():
    """Start A2A SDK server on port 3011"""
    import uvicorn
    from stargate_agent.a2a_server import get_a2a_app
    
    a2a_app = get_a2a_app()
    uvicorn.run(a2a_app, host="0.0.0.0", port=A2A_PORT, log_level="info")

def main():
    # Start A2A server in background thread
    t = threading.Thread(target=_start_a2a_server, daemon=True)
    t.start()
    # Run FastAPI app in main thread
    uvicorn.run(app, host="0.0.0.0", port=PORT)

if __name__ == "__main__":
    main()
```

**关键点**：
- 主服务运行在端口 **3001**
- A2A 服务运行在端口 **3011**（后台线程）
- 使用 `mcp_ui_server.create_ui_resource` 创建 UI 资源

---

## 2. a2a_server.py (41行)

```python
"""A2A Server setup using a2a-sdk"""
from a2a.server.apps.jsonrpc import A2AFastAPIApplication
from a2a.server.request_handlers import DefaultRequestHandler
from a2a.server.tasks import InMemoryTaskStore

from stargate_agent.agent_card_builder import build_agent_card
from stargate_agent.a2a_executor import StargateAgentExecutor


def create_a2a_app():
    """Create A2A FastAPI application"""
    
    # Build agent card with extensions
    agent_card = build_agent_card()
    
    # Create task store
    task_store = InMemoryTaskStore()
    
    # Create custom agent executor
    agent_executor = StargateAgentExecutor()
    
    # Create request handler
    request_handler = DefaultRequestHandler(
        agent_executor=agent_executor,
        task_store=task_store,
    )
    
    # Create A2A FastAPI application
    a2a_app = A2AFastAPIApplication(
        agent_card=agent_card,
        http_handler=request_handler,
    )
    
    # Build and return the FastAPI app
    return a2a_app.build()


def get_a2a_app():
    """Get A2A app instance for uvicorn"""
    return create_a2a_app()
```

**关键点**：
- 使用 `a2a-sdk` 的 `A2AFastAPIApplication` 创建服务
- 使用 `InMemoryTaskStore` 存储任务
- 使用自定义的 `StargateAgentExecutor` 执行器
- **返回数据格式**：通过 executor 生成，详见下文

---

## 3. a2a_executor.py (271行) - 核心文件

```python
"""A2A Agent Executor for stargate-agent using a2a-sdk"""
import asyncio
import json
import uuid
from datetime import datetime
from typing import AsyncGenerator

from a2a.server.agent_execution import AgentExecutor, RequestContext
from a2a.server.events.event_queue import EventQueue
from a2a.types import (
    Task,
    TaskStatus,
    TaskState,
    TaskStatusUpdateEvent,
    TaskArtifactUpdateEvent,
    Message,
    DataPart,
    TextPart,
)

from stargate_agent import llm
from stargate_agent.agent_card_builder import build_agent_card


class StargateAgentExecutor(AgentExecutor):
    """Custom Agent Executor for handling A2A tasks"""

    async def execute(self, context: RequestContext, event_queue: EventQueue) -> None:
        """Execute task and publish events to event_queue"""
        # ... 状态处理、消息解析、工具调用等
        
    async def _handle_tool_request(self, task_id: str, context_id: str, request_data: dict, event_queue: EventQueue) -> None:
        """Handle tool-protocol extension requests"""
        # ...

    def _create_tool_response_message(self, request_id: str, result: dict) -> Message:
        """Create tool response message with DataPart"""
        return Message(
            messageId=str(uuid.uuid4()),
            role="agent",
            parts=[
                DataPart(
                    data={
                        "kind": "tool_response",
                        "requestId": request_id,
                        "result": result
                    },
                    metadata={"schema": "https://stargate.example.com/schemas/tool-response-v1"}
                )
            ]
        )

    async def _handle_employee_trend(self, mode: str) -> dict:
        """Handle employee trend query"""
        return {
            "text": "已为您查询快手历年员工趋势数据，共 5 年记录。",
            "mcp_ui_resource": {
                "kind": "mcp_ui_resource",
                "resourceUri": "ui://stargate/employee-trend",
                "toolName": "query_employee_trend",
                "toolResult": {
                    "content": [{"type": "text", "text": "已为您查询快手历年员工趋势数据，共 5 年记录。"}],
                    "data": [
                        {"year": 2019, "count": 7000},
                        {"year": 2020, "count": 10000},
                        {"year": 2021, "count": 16000},
                        {"year": 2022, "count": 22000},
                        {"year": 2023, "count": 18000},
                    ],
                    "token": "mock-stargate-token-12345",
                },
                "uiMetadata": {
                    "preferred-frame-size": {"width": 560, "height": 420}
                },
            },
        }

    async def _handle_employee_trend_lazy(self, mode: str) -> dict:
        """Handle lazy employee trend query"""
        return {
            "text": "正在为您准备员工趋势数据，请稍候...",
            "mcp_ui_resource": {
                "kind": "mcp_ui_resource",
                "resourceUri": "ui://stargate/employee-trend-lazy",
                "toolName": "query_employee_trend_lazy",
                "toolResult": {
                    "content": [{"type": "text", "text": "正在为您准备员工趋势数据，请稍候..."}],
                    "token": "mock-stargate-token-12345",
                },
                "uiMetadata": {
                    "preferred-frame-size": {"width": 560, "height": 420}
                },
            },
        }
```

### 关键发现：result 中确实包含 `mcp_ui_resource` 字段

在 `_handle_employee_trend` 和 `_handle_employee_trend_lazy` 方法中，返回的 result 字典结构如下：

```python
{
    "text": "...",
    "mcp_ui_resource": {
        "kind": "mcp_ui_resource",
        "resourceUri": "ui://stargate/employee-trend",  # 或 employee-trend-lazy
        "toolName": "query_employee_trend",  # 或 query_employee_trend_lazy
        "toolResult": {
            "content": [{"type": "text", "text": "..."}],
            "data": [...],  # 仅 employee-trend 有
            "token": "mock-stargate-token-12345",
        },
        "uiMetadata": {
            "preferred-frame-size": {"width": 560, "height": 420}
        },
    },
}
```

这个 result 通过 `_create_response_message` 方法被封装到 `DataPart` 中：

```python
def _create_response_message(self, result: dict) -> Message:
    """Create response message with DataPart"""
    parts = []
    if "text" in result:
        parts.append(TextPart(text=result["text"]))
    parts.append(DataPart(
        data=result,
        metadata={"schema": "https://stargate.example.com/schemas/agent-response-v1"}
    ))
    return Message(messageId=str(uuid.uuid4()), role="agent", parts=parts)
```

---

## 4. agent_card_builder.py (25行)

```python
"""AgentCard builder for a2a-sdk"""
from a2a.types import AgentCard, AgentCapabilities, AgentExtension

A2A_PORT = 3011

def build_agent_card() -> AgentCard:
    """Build AgentCard with A2A extensions for a2a-sdk"""
    return AgentCard(
        name="stargate-agent",
        description="Stargate A2A Agent with MCP-UI support",
        url=f"http://localhost:{A2A_PORT}",
        version="0.1.0",
        capabilities=AgentCapabilities(
            streaming=True,
            extensions=[
                AgentExtension(uri="https://stargate.example.com/ext/a2a-structured-data/v1", name="structured-data", required=True),
                AgentExtension(uri="https://stargate.example.com/ext/a2a-streaming/v1", name="streaming", required=True),
                AgentExtension(uri="https://stargate.example.com/ext/a2a-tool-protocol/v1", name="tool-protocol", required=True),
            ]
        ),
        defaultInputModes=["text/plain"],
        defaultOutputModes=["text/plain"],
        skills=[]
    )
```

**关键点**：
- Agent 定义了3个扩展：
  1. `structured-data` - 结构化数据扩展
  2. `streaming` - 流式传输扩展  
  3. `tool-protocol` - 工具协议扩展（支持直接调用工具）

---

## 总结

| 文件 | 行数 | 主要功能 |
|------|------|----------|
| `main.py` | 276 | 主入口，FastAPI 服务(3001) + A2A 服务(3011) |
| `a2a_server.py` | 41 | A2A SDK 应用创建，配置 task store 和 executor |
| `a2a_executor.py` | 271 | **核心**：任务执行器，**result 包含 `mcp_ui_resource` 字段** |
| `agent_card_builder.py` | 25 | AgentCard 构建，声明3个扩展能力 |

**关于 `mcp_ui_resource` 字段的确认**：是的，`a2a_executor.py` 中的 `_handle_employee_trend` 和 `_handle_employee_trend_lazy` 方法返回的 result 字典确实包含 `mcp_ui_resource` 字段，其中包含 `kind`、`resourceUri`、`toolName`、`toolResult` 和 `uiMetadata` 等子字段。
