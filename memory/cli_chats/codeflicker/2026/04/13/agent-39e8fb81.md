---
session_id: agent-39e8fb81
date: 2026-04-13
time_start: 2026-04-13T14:50:09.535Z
time_end: 2026-04-13T14:55:50.084Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: 
---

[2026-04-13 22:50:09] [user]: 在 /Users/lianzimeng/working/A2A-mcpUI 项目中，快速梳理以下几点并返回简洁结论：
1. CF-Agent 调用 SG-Agent 时，目前如何传递 mcp_ui_resource（python_a2a 库的用法、请求/响应的具体数据结构）
2. SG-Agent 的 AgentCard 当前声明了哪些字段
3. python_a2a 库是否支持多 part、data part、自定义 header——查看 .venv 下的 python_a2a 源码或 pyproject.toml 确认版本和能力
只需返回事实，不要写代码。
[2026-04-13 22:50:29] [assistant]: 我来帮你快速梳理这个项目的几个关键问题。让我先搜索项目结构和相关文件。
[2026-04-13 22:50:46] [assistant]: 让我更精准地搜索项目结构：
[2026-04-13 22:51:04] [assistant]: 现在让我并行搜索这些关键信息：
[2026-04-13 22:51:20] [assistant]: 现在让我搜索 SG-Agent 的相关文件：
[2026-04-13 22:51:39] [assistant]: 现在让我查看 python_a2a 库的源码和 AgentCard 的定义：
[2026-04-13 22:52:05] [assistant]: 现在让我查看 python_a2a 的源码：
[2026-04-13 22:52:50] [assistant]: 现在让我查看 Message 和 TextContent 的详细定义，以及 python_a2a 的多 part 支持：
[2026-04-13 22:53:11] [assistant]: 现在让我查看 server 的实现细节，特别是关于 multipart 的支持：
[2026-04-13 22:53:27] [assistant]: 让我查看 http.py 的详细实现，看 send_message 的方法：
[2026-04-13 22:54:02] [assistant]: 现在让我查看项目中 CF-Agent 如何实际调用 SG-Agent 的具体逻辑：
[2026-04-13 22:54:17] [assistant]: 现在查看版本信息和 python-a2a 的 METADATA：
[2026-04-13 22:54:35] [assistant]: 现在让我查看 stargate-agent 中 AgentCard 的具体声明：
[2026-04-13 22:55:07] [assistant]: 现在让我查看 HTTP client 如何处理自定义 header 的详细逻辑：
[2026-04-13 22:55:24] [assistant]: 完美。现在让我最后检查一下关于 multipart 和 data part 的支持：
[2026-04-13 22:55:50] [assistant]: 完美，现在让我总结所有信息。我已经有足够的信息来回答所有三个问题。

## 简洁结论

### 1. **CF-Agent 调用 SG-Agent 时，mcp_ui_resource 的传递方式**

**请求数据结构：**
- CF-Agent 通过 `A2AClient.send_message()` 调用 SG-Agent
- 请求 payload 为 JSON：`{"text": user_message, "mode": "endpoint" | "mcp"}`
- 使用 `TextContent` 和 `MessageRole.USER` 包装发送

**响应数据结构：**
- SG-Agent 响应为 JSON 字符串，包含两个关键字段：
  ```json
  {
    "text": "描述文本",
    "mcp_ui_resource": {
      "kind": "mcp_ui_resource",
      "resourceUri": "ui://stargate/card/{card_id}",
      "toolName": "query_employee_trend",
      "toolResult": {...},
      "uiMetadata": {"preferred-frame-size": {...}}
    }
  }
  ```
- CF-Agent 解析后将响应分解为 `parts` 数组，每个 part 有 `kind` 字段区分（text 或 mcp_ui_resource）

**python_a2a 库的用法：**
- A2AClient 初始化时可传入自定义 headers：`A2AClient(endpoint_url, headers={"Authorization": "Bearer token"})`
- send_message() 接收 Message 对象（包含 TextContent、MessageRole）
- 响应为 Message 对象，content 为 TextContent.text，其中通常是 JSON 字符串

---

### 2. **SG-Agent 的 AgentCard 当前声明字段**

根据 `stargate-agent/main.py` 第 380-385 行：

```python
agent_card = AgentCard(
    name="stargate-agent",
    description="Stargate A2A Agent with MCP-UI support",
    url=f"http://localhost:{A2A_PORT}",
    version="0.1.0",
)
```

**当前只声明了 4 个字段：**
1. `name`: 代理名称
2. `description`: 代理描述
3. `url`: 代理 A2A 服务地址
4. `version`: 代理版本

**AgentCard 的完整可用字段（python_a2a 0.5.10）：**
- 必需：name, description, url, version
- 可选：protocol_version (默认"0.3.0"), preferred_transport (默认"JSONRPC"), authentication, capabilities, default_input_modes, default_output_modes, skills, provider, documentation_url

---

### 3. **python_a2a 库（版本 0.5.10）的能力支持**

**版本信息：**
- 当前版本：**0.5.10**（pyproject.toml 中 `python-a2a>=0.5.0`）
- 依赖：requests, flask, aiohttp, openai, anthropic, boto3, botocore, httpx, fastapi, uvicorn, pydantic, langchain

**功能支持情况：**

| 功能 | 支持 | 说明 |
|------|------|------|
| **多 part** | ✅ 是 | 支持 Google A2A 格式的 parts 数组（text part 和 data part） |
| **data part** | ✅ 是 | 支持 data part，包含 function_call, function_response, error 等结构 |
| **自定义 header** | ✅ 是 | A2AClient 初始化时接受 `headers` 参数，支持任意 HTTP 头 |

**具体实现：**
- Message 支持 `from_google_a2a()` 方法，可解析 Google A2A 格式的 parts 数组
- 支持四种 content type：text, function_call, function_response, error
- HTTP 客户端自动管理 Content-Type，初始化时可注入自定义 headers
- Message 和 Metadata 都支持自定义字段扩展

**关键限制：**
- 当前实现中，Message 对象内部仍为单个 content（非数组），parts 支持是针对序列化/反序列化层面的格式转换
- 若需多个异构 part 同时返回，需在 JSON 字符串中手工组织（如 CF-Agent 中的做法）
