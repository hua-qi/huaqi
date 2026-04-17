---
session_id: agent-3ef770c5
date: 2026-04-13
time_start: 2026-04-13T12:33:50.156Z
time_end: 2026-04-13T12:37:36.020Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: 
---

[2026-04-13 20:33:50] [user]: 请全面探索 /Users/lianzimeng/working/A2A-mcpUI 目录下的所有项目代码结构。我需要了解：

1. 顶层目录结构
2. 每个子项目/服务的目录结构
3. 各服务的核心文件内容（特别是：入口文件、路由定义、核心逻辑、数据模型/类型定义）
4. 各服务之间如何通信（API调用、消息格式）
5. MCP-UI 和 A2A 之间的交互关系
6. 关键的数据结构和协议定义

请详细阅读所有关键代码文件，包括：
- package.json（了解依赖和脚本）
- 主要的 TypeScript/JavaScript 源文件
- 配置文件
- README 文件

请返回非常详细的分析报告，包含：
- 完整的项目结构树
- 每个服务的核心功能描述
- 关键代码片段（函数签名、接口定义、类型定义）
- 服务间通信的数据格式
- A2A 协议的关键数据结构
- MCP 协议的关键数据结构
[2026-04-13 20:33:54] [assistant]: 我来帮您全面探索这个项目的代码结构。让我首先检查 git 仓库信息，然后进行深入的代码分析。
[2026-04-13 20:35:15] [assistant]: 项目不是 git 仓库。让我开始探索目录结构：
[2026-04-13 20:35:20] [assistant]: 现在让我读取关键的配置和协议文件：
[2026-04-13 20:35:26] [assistant]: 现在让我探索各个子项目的核心代码：
[2026-04-13 20:35:31] [assistant]: 现在让我读取各个 Agent 的主要代码文件：
[2026-04-13 20:35:37] [assistant]: 现在让我读取前端源代码和其他关键文件：
[2026-04-13 20:35:42] [assistant]: 现在让我读取更多核心文件，特别是前端组件和设计文档：
[2026-04-13 20:35:48] [assistant]: 现在让我查看更多设计文档和其他关键文件：
[2026-04-13 20:35:53] [assistant]: 现在让我查看 pyproject.toml 文件来了解依赖关系：
[2026-04-13 20:35:58] [assistant]: 让我读取 sandbox_proxy.html 和其他重要文件：
[2026-04-13 20:36:02] [assistant]: 让我读取一些关键的设计文档：
[2026-04-13 20:36:10] [assistant]: 现在让我检查一些其他 TypeScript 文件和查看项目结构完整性：
[2026-04-13 20:37:36] [assistant]: 现在我有足够的信息来生成详细报告。让我生成总结：

## 详细分析报告：A2A-mcpUI 项目全面代码结构探索

### 一、项目概述

**项目名称**：A2A-mcpUI  
**性质**：Multi-Agent Demo 项目，展示在 A2A 协议和 MCP-UI 框架下多个 Agent 如何协作和渲染富交互UI  
**技术栈**：
- **后端**：Python (FastAPI, Flask) + A2A、MCP、LangGraph 生态
- **前端**：React 18 + TypeScript + Vite
- **通信**：A2A 协议（Agent间通信）+ MCP 协议（模型与工具/数据连接）+ MCP-UI 协议（UI 层扩展）

---

### 二、顶层项目结构

```
A2A-mcpUI (根)
├── package.json                      # 工作区根配置
├── pnpm-workspace.yaml              # Pnpm monorepo
├── a2a-protocol.md                  # A2A 协议总结文档
├── mcp-ui-protocol.md               # MCP-UI 协议文档
├── a2a-protocol-fields.md           # A2A 协议详细字段定义
├── mcp-ui-protocol-fields.md        # MCP-UI 协议详细字段定义
├── design.md                        # 架构设计方案
├── .env.example                     # 环境变量模板
│
├── packages/
│   │
│   ├── codeflicker-agent/           # Agent 1：对话入口
│   │   ├── package.json
│   │   ├── pyproject.toml
│   │   └── src/codeflicker_agent/
│   │       ├── main.py              # FastAPI 服务，端口 3002
│   │       ├── llm.py               # OpenAI API 集成
│   │       └── sse_logger.py        # 事件日志 SSE 推送
│   │
│   ├── stargate-agent/              # Agent 2：业务数据查询
│   │   ├── package.json
│   │   ├── pyproject.toml
│   │   └── src/stargate_agent/
│   │       ├── main.py              # FastAPI 服务，端口 3001；A2A Flask 服务，端口 3011
│   │       ├── llm.py               # Tool 选择逻辑
│   │       ├── card_cache.py        # Card Instance 缓存
│   │       ├── shell_builder.py     # HTML 壳子生成
│   │       └── sse_logger.py        # 事件日志
│   │
│   ├── stargate-mcp-ui-server/      # Agent 3：MCP-UI 协议实现
│   │   ├── package.json
│   │   ├── pyproject.toml
│   │   └── src/stargate_mcp_ui_server/
│   │       ├── main.py              # FastMCP 服务，端口 3005（SSE）
│   │       └── tools.py             # Tool + Resource 定义
│   │
│   ├── codeflicker-frontend/        # 前端应用
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── vite.config.ts
│   │   ├── index.html
│   │   ├── public/
│   │   │   └── sandbox_proxy.html   # iframe 沙箱代理
│   │   └── src/
│   │       ├── main.tsx             # 入口
│   │       ├── App.tsx              # 主应用组件
│   │       ├── types.ts             # 类型定义
│   │       ├── hooks/
│   │       │   └── useEventLog.ts   # SSE 事件订阅 Hook
│   │       └── components/
│   │           ├── ChatMessage.tsx  # 聊天消息组件
│   │           ├── CardMessage.tsx  # MCP-UI 卡片渲染器
│   │           ├── SequenceDiagram.tsx # 时序图可视化
│   │           └── ResizableDivider.tsx # 可调整分隔条
│   │
│   ├── resource-center-mock/        # 组件注册中心
│   │   ├── package.json
│   │   └── src/
│   │       └── index.ts             # Express 服务，端口 3003
│   │
│   ├── employee-chart-card/         # 业务卡片组件
│   │   ├── package.json
│   │   ├── webpack.config.js
│   │   ├── tsconfig.json
│   │   └── src/
│   │       ├── EmployeeChart.tsx    # 员工趋势图表（完整数据）
│   │       ├── EmployeeChartLazy.tsx # 员工趋势图表（懒加载）
│   │       └── index.tsx            # Module Federation 入口
│   │
│   └── [虚拟环境 .venv/]           # Python 依赖
│
└── docs/
    ├── architecture-decisions.md    # 架构决策记录
    ├── designs/                     # 设计文档
    └── plans/                       # 计划文档
```

---

### 三、各服务深入分析

#### 1. **CodeFlicker Agent**（端口 3002）
**职责**：对话入口、意图识别、A2A 客户端

**核心文件**：`packages/codeflicker-agent/src/codeflicker_agent/main.py`

**关键接口**：
| 端点 | 方法 | 职责 |
|------|------|------|
| `/health` | GET | 健康检查 |
| `/mode` | GET/POST | 切换工作模式（endpoint/mcp） |
| `/chat` | POST | 主对话端点 |
| `/tool-call` | POST | 工具调用代理 |
| `/tool-result` | POST | 工具结果获取 |
| `/resource-proxy` | GET | 资源跨域代理 |
| `/events` | GET | SSE 事件流 |

**核心流程**：
```python
# 1. 接收用户消息
POST /chat { "message": "查询快手员工趋势" }

# 2. LLM 意图识别（OpenAI）
async def detect_intent(user_message) -> "query_data" | "general_chat"

# 3. 调用 Stargate Agent（A2A 协议）
response_msg = A2AClient(endpoint_url=SG_AGENT_A2A_URL)
    .send_message(Message(content=TextContent(text=user_message)))

# 4. 解析响应并返回
# 响应格式：{ "text": "...", "mcp_ui_resource": { kind, resourceUri, toolResult, ... } }

# 5. 根据 mode 参数动态切换：
# - mode=endpoint: 调业务 API 获取数据，由 Card Instance 包装
# - mode=mcp: 调 MCP Server 获取数据
```

**类型定义** (`types.ts`)：
```typescript
interface TextPart {
  kind: 'text';
  text: string;
}

interface McpUiResourcePart {
  kind: 'mcp_ui_resource';
  resourceUri: string;           // ui://stargate/card/{id}
  toolName?: string;
  toolResult?: {
    content?: Array<{ type: string; text: string }>;
    data?: Array<{ year: number; count: number }>;
    token?: string;
  };
  uiMetadata?: {
    'preferred-frame-size'?: { width: number; height: number };
  };
}

type MessagePart = TextPart | McpUiResourcePart;
```

---

#### 2. **Stargate Agent**（端口 3001 / A2A 3011）
**职责**：业务数据查询、Card Instance 管理、A2A 服务端

**核心文件**：
- `packages/stargate-agent/src/stargate_agent/main.py` - 主服务
- `packages/stargate-agent/src/stargate_agent/llm.py` - Tool 选择
- `packages/stargate-agent/src/stargate_agent/card_cache.py` - Card 缓存

**关键接口**：
| 端点 | 方法 | 职责 |
|------|------|------|
| `/health` | GET | 健康检查 |
| `/api/card-instance/{card_id}` | GET | 获取 Card Instance 信息 |
| `/api/token/exchange` | POST | 换取 Stargate Token |
| `/api/tool-result/{tool_name}` | GET | 获取工具结果 |
| `/api/employee/trend` | GET | 员工趋势数据 |
| `/api/employee/detail/{year}` | GET | 年份详情 |
| `/mcp/resources/read` | GET | MCP 资源读取 |
| `/.well-known/agent-card` | GET | A2A Agent 身份卡 |
| `/sendMessage` | POST | A2A 消息处理（Flask 路由） |
| `/events` | GET | SSE 事件流 |

**Card Instance 数据结构**：
```python
@dataclass
class CardInstance:
    component_name: str        # "EmployeeChart"
    container_name: str        # "employeeChartCard"
    remote_entry_url: str      # "http://localhost:3004/remoteEntry.js"
    props: dict[str, Any]      # { "data": [...] }
    created_at: float          # TTL = 3600s
```

**Tool 选择逻辑**：
```python
TOOLS = [
    {
        "name": "query_employee_trend",
        "description": "查询快手历年员工人数趋势数据",
    },
    {
        "name": "query_employee_trend_lazy",
        "description": "懒加载方式查询快手历年员工人数趋势",
    },
]

async def select_tool(user_message) -> (tool_name, tool_args)
    # OpenAI ChatCompletion with tool_choice="auto"
```

**两种数据流路径**：

**Endpoint 路径**（mode=endpoint）：
```
1. SG-Agent 调 resource-center-mock GET /api/components/EmployeeChart
   ↓ 返回：{ componentName, containerName, remoteEntryUrl }

2. SG-Agent 调自身 GET /api/employee/trend
   ↓ 返回：{ data, token }

3. SG-Agent 构造 Card Instance，缓存并返回 cardInstanceId

4. CF-Agent 收到 A2A 响应，包含 resourceUri = "ui://stargate/card/{cardInstanceId}"

5. 前端 /resource-proxy?uri=... → CF-Agent → SG-Agent /mcp/resources/read
   ↓ 返回 HTML 壳子（Module Federation 加载器）

6. 前端 iframe 加载壳子，使用 Module Federation 动态加载 EmployeeChart 组件
```

**MCP 路径**（mode=mcp）：
```
1. SG-Agent 调 MCP Server `/tools/call` query_employee_trend
   ↓ MCP Server 返回 { _meta: { ui: { resourceUri } }, resource: { text: HTML } }

2. SG-Agent 调自身 GET /api/employee/trend
   ↓ 返回：{ data, token }

3. SG-Agent 拼 A2A 响应，resourceUri 由 MCP Server 提供
```

---

#### 3. **Stargate MCP-UI Server**（端口 3005）
**职责**：标准 MCP 协议实现、UI Resource 提供

**核心文件**：
- `packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/main.py` - FastMCP 定义
- `packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/tools.py` - Tool + Resource

**MCP Tool 定义**：
```python
@mcp.tool()
async def query_employee_trend() -> dict:
    r = get_ui_resource()  # 生成 UIResource 对象
    return {
        "_meta": { "ui": { "resourceUri": RESOURCE_URI } },
        "content": [{ "type": "text", "text": "已为您查询..." }],
        "resource": r["resource"],  # { uri, mimeType, text }
        "toolName": "query_employee_trend",
    }

@mcp.resource(RESOURCE_URI, mime_type="text/html;profile=mcp-app")
async def employee_trend_resource() -> str:
    return get_ui_resource()["resource"]["text"]
```

**UIResource 生成**：
```python
def get_ui_resource() -> dict:
    resource = create_ui_resource({
        "uri": "ui://stargate/employee-trend",
        "content": {
            "type": "rawHtml",
            "htmlString": build_html()  # Module Federation HTML
        },
        "encoding": "text",
    })
    return resource.model_dump(mode="json")

# 返回结构
{
    "type": "resource",
    "resource": {
        "uri": "ui://stargate/employee-trend",
        "mimeType": "text/html;profile=mcp-app",
        "text": "<!DOCTYPE html>...",  # 完整 HTML
    }
}
```

**HTML 壳子结构** (`build_html()`）：
```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
</head>
<body style="margin:0">
  <div id="root"></div>
  <script src="http://localhost:3004/remoteEntry.js"></script>  <!-- Module Federation 入口 -->
  <script>
    // 1. 验证容器是否存在
    if (typeof employeeChartCard === 'undefined') throw new Error('Container not found');
    
    // 2. 创建共享作用域，声明 React/React-DOM 为 singleton
    var shareScope = Object.create(null);
    shareScope['default'] = {};
    shareScope['default']['react'] = {
      '18.3.1': { get: function() { return function() { return React; }; }, loaded: 1, from: 'host' }
    };
    shareScope['default']['react-dom'] = {
      '18.3.1': { get: function() { return function() { return ReactDOM; }; }, loaded: 1, from: 'host' }
    };
    
    // 3. 初始化 Module Federation
    if (employeeChartCard.init) {
      try { employeeChartCard.init(shareScope['default']); } catch(e) {}
    }
    
    // 4. 动态加载组件 "./EmployeeChart"
    return employeeChartCard.get('./EmployeeChart');
    
    // 5. 渲染
    var Comp = factory().default;
    ReactDOM.createRoot(document.getElementById('root'))
      .render(React.createElement(Comp, {}));
  </script>
</body>
</html>
```

---

#### 4. **CodeFlicker 前端**（端口 3000）
**职责**：用户交互界面、消息展示、卡片渲染、事件时序图

**核心组件**：

**App.tsx** - 主应用容器：
```typescript
interface ChatMessage {
  id: string;
  role: 'user' | 'agent';
  parts: MessagePart[];  // 支持文本 + MCP-UI 资源
}

// 关键状态
const [messages, setMessages] = useState<ChatMessage[]>([]);
const [mode, setMode] = useState<'endpoint' | 'mcp'>('endpoint');
const [diagramHeight, setDiagramHeight] = useState(240);

// 核心流程
const sendMessage = async (text: string) => {
  // 1. POST /chat { message }
  // 2. 解析 parts（TextPart | McpUiResourcePart）
  // 3. 添加到消息列表
  // 4. CardMessage 渲染 MCP-UI 资源
};

// 事件监听
const eventEntries = useEventLog([
  'http://localhost:3002/events',
  'http://localhost:3001/events',
]);
```

**CardMessage.tsx** - MCP-UI 卡片渲染器：
```typescript
export const CardMessage: React.FC<Props> = ({ resourceUri, toolResult, ... }) => {
  return (
    <AppRenderer  // @mcp-ui/client
      toolName={toolName}
      toolResult={toolResult}
      toolResourceUri={resourceUri}
      sandbox={{ url: sandboxUrl }}  // sandbox_proxy.html
      
      onReadResource={async ({ uri }) => {
        // 跨域资源代理：/resource-proxy?uri=...
        const res = await fetch(`/resource-proxy?uri=${encodeURIComponent(uri)}`);
        return res.json();
      }}
      
      onCallTool={async (params) => {
        // 工具调用代理：POST /tool-call
        const res = await fetch('/tool-call', {
          method: 'POST',
          body: JSON.stringify({ toolName: params.name, arguments: params.arguments }),
        });
        return res.json().toolResult;
      }}
      
      onSizeChanged={({ height }) => {
        setIframeHeight(height);  // 动态调整 iframe 高度
      }}
      
      onMessage={async (params) => {
        // 处理来自卡片的消息，继续发送新请求
        onMessage(params.content[0].text);
      }}
    />
  );
};
```

**SequenceDiagram.tsx** - 时序图可视化：
- 从两个 SSE 源订阅事件
- 使用 ECharts 绘制参与者和消息箭头
- 支持热区提示、交互

**useEventLog.ts** - 事件聚合 Hook：
```typescript
export function useEventLog(urls: string[]) {
  const [entries, setEntries] = useState<EventLogEntry[]>([]);
  const seqRef = useRef(0);
  
  useEffect(() => {
    const sources = urls.map((url) => {
      const es = new EventSource(url);
      es.onmessage = (e) => {
        const entry: EventLogEntry = JSON.parse(e.data);
        const indexed = { ...entry, _seq: seqRef.current++ };
        // 按时间和序列号排序，最多保留 100 条
      };
      return es;
    });
    return () => sources.forEach(es => es.close());
  }, [urls]);
  
  return entries;
}
```

---

#### 5. **Resource Center Mock**（端口 3003）
**职责**：组件注册中心，存储 Module Federation 组件元数据

**核心接口**：
```typescript
app.get('/api/components/:name', (req, res) => {
  if (name === 'EmployeeChart') {
    res.json({
      componentName: 'EmployeeChart',
      containerName: 'employeeChartCard',
      remoteEntryUrl: 'http://localhost:3004/remoteEntry.js',
    });
  }
});
```

**注**：`remoteEntryUrl` 指向 `employee-chart-card` 构建产物的 Module Federation 入口。

---

#### 6. **Employee Chart Card**（webpack 构建为 Module Federation）
**职责**：业务卡片组件，提供数据展示和交互

**两个变体**：

**EmployeeChart.tsx** - 完整版（接收初始数据）：
```typescript
interface EmployeeChartProps {
  data?: { year: number; count: number }[];
  token?: string;
}

export const EmployeeChart: React.FC<EmployeeChartProps> = ({
  data: initialData,
  token: initialToken,
  sgAgentBaseUrl = 'http://localhost:3001',
}) => {
  // 1. 接收初始数据（来自 toolResult）
  const [data, setData] = useState(initialData ?? []);
  
  // 2. 监听 postMessage，处理 tool-result 通知
  useEffect(() => {
    window.addEventListener('message', (e) => {
      if (e.data.method === 'ui/notifications/tool-result') {
        const result = e.data.params?.result ?? e.data.params;
        if (result.data) setData(result.data);
      }
    });
  }, []);
  
  // 3. 绘制图表
  useEffect(() => {
    const chart = echarts.init(chartRef.current);
    chart.setOption({...});
  }, [data]);
  
  // 4. 响应式尺寸通知
  const notifySize = () => {
    window.parent.postMessage({
      jsonrpc: '2.0',
      method: 'ui/notifications/size-changed',
      params: { height: containerRef.current.height }
    }, '*');
  };
  
  // 5. 卡片内交互：分析、刷新、详情
  const handleRefresh = () => {
    window.parent.postMessage({
      jsonrpc: '2.0',
      id: Date.now(),
      method: 'tools/call',
      params: { name: 'query_employee_trend', arguments: {} }
    }, '*');
  };
  
  const handleHoverYear = async (year: number) => {
    const res = await fetch(
      `${sgAgentBaseUrl}/api/employee/detail/${year}`,
      { headers: { Authorization: `Bearer ${token}` } }
    );
    setDetail(await res.json());
  };
};
```

**EmployeeChartLazy.tsx** - 懒加载版（主动拉数据）：
```typescript
const EmployeeChartLazy: React.FC = () => {
  const [data, setData] = useState<{ year: number; count: number }[]>([]);
  
  // 组件挂载后主动调用 tools/call
  useEffect(() => {
    window.parent.postMessage({
      jsonrpc: '2.0',
      id: 1,
      method: 'tools/call',
      params: { name: 'query_employee_trend_lazy', arguments: {} }
    }, '*');
  }, []);
  
  // 监听响应
  useEffect(() => {
    window.addEventListener('message', (e) => {
      if (e.data.id === 1 && e.data.result?.data) {
        setData(e.data.result.data);
      }
    });
  }, []);
  
  // 绘制并通知尺寸
  // ...
};
```

---

### 四、关键数据结构与协议

#### 1. **A2A 协议数据结构**

**Message（单轮通信）**：
```json
{
  "role": "user" | "agent",
  "messageId": "msg-001",
  "contextId": "ctx-abc",
  "parts": [
    { "text": "查询员工数据" }
  ],
  "metadata": {}
}
```

**Part（内容容器）**：
```json
// 文本
{ "text": "Hello" }

// 文件（Base64）
{ "raw": "<base64>", "mediaType": "image/png", "filename": "photo.png" }

// 文件（外部 URL）
{ "url": "https://...", "mediaType": "application/pdf" }

// 结构化数据
{ "data": { "key": "value" }, "mediaType": "application/json" }
```

**Task（有状态工作单元）**：
```json
{
  "id": "task-123",
  "contextId": "ctx-abc",
  "status": {
    "state": "completed" | "working" | "submitted" | "input-required" | "failed",
    "message": { "role": "agent", "parts": [...] },
    "timestamp": "2026-04-13T10:00:00Z"
  },
  "artifacts": [
    {
      "artifactId": "artifact-001",
      "name": "report.pdf",
      "parts": [{ "url": "...", "mediaType": "application/pdf" }]
    }
  ]
}
```

**AgentCard（Agent 身份证）**：
```json
{
  "name": "stargate-agent",
  "description": "Stargate A2A Agent with MCP-UI support",
  "url": "http://localhost:3011",
  "version": "0.1.0",
  "capabilities": {
    "streaming": true,
    "pushNotifications": false,
    "stateTransitionHistory": false
  },
  "skills": [
    {
      "id": "query_employee_trend",
      "name": "查询员工趋势",
      "description": "查询快手历年员工人数趋势数据",
      "inputModes": ["text"],
      "outputModes": ["text", "data"]
    }
  ]
}
```

#### 2. **MCP-UI 协议数据结构**

**UIResource（核心数据结构）**：
```typescript
interface UIResource {
  type: 'resource';
  resource: {
    uri: string;                              // ui://stargate/employee-trend
    mimeType: 'text/html;profile=mcp-app';   // 固定值
    text?: string;                            // encoding='text' 时的 HTML
    blob?: string;                            // encoding='blob' 时的 Base64
    _meta?: {
      ui?: {
        resourceUri: string;
      };
    };
  };
}
```

**Tool 与 UI 的绑定**：
```typescript
interface Tool {
  name: string;
  description: string;
  inputSchema: object;
  _meta?: {
    ui?: {
      resourceUri: string;  // 指向对应 UIResource 的 uri
    };
  };
}
```

**AppRenderer Props**（前端组件）**：
```typescript
interface AppRendererProps {
  toolName: string;                    // 工具名称
  toolResult?: CallToolResult;         // 工具执行结果
  toolResourceUri: string;             // UI 资源 URI
  sandbox?: SandboxConfig;             // 沙箱配置
  
  // 资源读取
  onReadResource?: (params: { uri: string }) => Promise<object>;
  
  // 工具调用
  onCallTool?: (params: { name: string; arguments: object }) => Promise<object>;
  
  // 事件回调
  onSizeChanged?: (params: { height: number }) => void;
  onMessage?: (params: { content: object[] }) => Promise<object>;
  onError?: (error: Error) => void;
}
```

**postMessage 通信协议（Host ↔ iframe）**：

| 方向 | 消息类型 | Payload |
|------|---------|---------|
| Host → Guest | `ui/notifications/tool-input` | `{ toolInput: any }` |
| Host → Guest | `ui/notifications/tool-result` | `{ toolResult: object }` |
| Host → Guest | `ui/notifications/size-changed` | `{ maxHeight?: number }` |
| Guest → Host | `tools/call` | `{ name: string; arguments: object }` |
| Guest → Host | `ui/message` | `{ role, content }` |
| Guest → Host | `ui/notifications/size-changed` | `{ height: number }` |

#### 3. **项目级扩展协议**

**McpUiResourcePart**（融合 A2A + MCP-UI）**：
```typescript
interface McpUiResourcePart {
  kind: 'mcp_ui_resource';
  resourceUri: string;  // ui://stargate/card/{id} | ui://stargate/employee-trend
  toolName?: string;    // "query_employee_trend"
  toolResult?: {        // 业务数据包
    content?: Array<{ type: string; text: string }>;
    data?: Array<{ year: number; count: number }>;
    token?: string;
  };
  uiMetadata?: {
    'preferred-frame-size'?: { width: number; height: number };
  };
}
```

---

### 五、完整数据流示例

#### 场景1：Endpoint 模式（完整数据加载）

```
1. 用户输入
   用户消息：✉️ "查询快手员工趋势"

2. CodeFlicker 前端
   POST http://localhost:3002/chat
   { "message": "查询快手员工趋势" }

3. CodeFlicker Agent
   - 检查 mode=endpoint
   - LLM 意图识别 → "query_data"
   - 构造 A2A Message，发送至 Stargate Agent

4. A2A 通信（CF-Agent → SG-Agent）
   POST http://localhost:3011/sendMessage
   {
     "jsonrpc": "2.0",
     "method": "message/send",
     "params": {
       "message": {
         "role": "user",
         "parts": [{ "text": "查询快手员工趋势" }]
       }
     }
   }

5. Stargate Agent - handle_message()
   a. 解析消息内容
   b. LLM Tool 选择 → "query_employee_trend"
   c. 判断 mode=endpoint，执行 endpoint 路径：
      i.   GET http://localhost:3003/api/components/EmployeeChart
           ↓ 返回：{ componentName, containerName, remoteEntryUrl }
      
      ii.  GET http://localhost:3001/api/employee/trend（自身接口）
           ↓ 返回：{ data: [...], token: "mock-token" }
      
      iii. card_cache.put() → cardInstanceId = "uuid-123"
      
      iv.  构造 A2A Response：
           {
             "text": "已为您查询快手历年员工趋势数据，共 5 年记录。",
             "mcp_ui_resource": {
               "kind": "mcp_ui_resource",
               "resourceUri": "ui://stargate/card/uuid-123",
               "toolName": "query_employee_trend",
               "toolResult": {
                 "content": [{ "type": "text", "text": "已为您查询..." }],
                 "data": [
                   { "year": 2019, "count": 7000 },
                   { "year": 2020, "count": 10000 },
                   { "year": 2021, "count": 16000 },
                   { "year": 2022, "count": 22000 },
                   { "year": 2023, "count": 18000 }
                 ],
                 "token": "mock-stargate-token-12345"
               },
               "uiMetadata": {
                 "preferred-frame-size": { "width": 560, "height": 420 }
               }
             }
           }

6. CodeFlicker Agent 收到 A2A 响应
   - 解析 JSON（text + mcp_ui_resource）
   - 返回前端

7. 前端接收
   POST /chat 响应：
   {
     "parts": [
       {
         "kind": "text",
         "text": "已为您查询快手历年员工趋势数据，共 5 年记录。"
       },
       {
         "kind": "mcp_ui_resource",
         "resourceUri": "ui://stargate/card/uuid-123",
         "toolName": "query_employee_trend",
         "toolResult": { ... },
         "uiMetadata": { ... }
       }
     ]
   }

8. 前端渲染 CardMessage
   <CardMessage
     resourceUri="ui://stargate/card/uuid-123"
     toolResult={toolResult}
     uiMetadata={uiMetadata}
   />

9. CardMessage 通过 AppRenderer 加载资源
   GET /resource-proxy?uri=ui://stargate/card/uuid-123
   ↓ CF-Agent 代理
   ↓ SG-Agent /mcp/resources/read?uri=...

10. SG-Agent /mcp/resources/read 处理
    a. 解析 cardInstanceId 从 uri
    b. card_cache.get(cardInstanceId) → CardInstance
    c. 生成 HTML 壳子：
       - script src="http://localhost:3004/remoteEntry.js"（Module Federation）
       - MCP_INIT_SCRIPT 初始化共享作用域
       - 加载 EmployeeChart 组件
    d. 返回 UIResource

11. 前端 iframe 接收 HTML 壳子
    a. sandbox_proxy.html 创建内部 iframe
    b. srcdoc 注入 HTML
    c. iframe 加载 remoteEntry.js

12. Module Federation 加载流程
    a. 验证容器 employeeChartCard 存在
    b. 创建共享作用域，声明 React 为 singleton
    c. employeeChartCard.init(shareScope)
    d. 动态 import('./EmployeeChart')
    e. 获取默认导出组件

13. EmployeeChart 组件挂载
    a. 接收初始 props：data、token（来自 toolResult）
    b. 使用 ECharts 绘制折线图
    c. 监听 postMessage，处理 tool-result 通知
    d. 计算高度，调用 notifySize() → ui/notifications/size-changed

14. 前端 AppRenderer 响应 size-changed
    a. 更新 iframe 高度
    b. 触发 onLayout 回调
    c. 页面滚动到最新消息

15. 用户与卡片交互
    a. 点击"刷新数据" 按钮
    b. EmployeeChart.handleRefresh()
    c. postMessage({ method: 'tools/call', name: 'query_employee_trend' })
    
    ↓ AppRenderer 捕获 tools/call 事件
    ↓ 调用 onCallTool 回调
    
    POST http://localhost:3002/tool-call
    { "toolName": "query_employee_trend", "arguments": {} }
    
    ↓ CF-Agent /tool-call 处理
    ↓ 再次发送 A2A 请求至 SG-Agent
    ↓ SG-Agent 返回新数据
    ↓ CF-Agent 代理 toolResult 返回
    
    ↓ AppRenderer 接收响应
    ↓ postMessage({ method: 'ui/notifications/tool-result', ... })
    
    ↓ EmployeeChart 监听 message 事件
    ↓ 更新 state，重绘图表
```

#### 场景2：MCP 模式（通过 MCP Server）

```
1-4. [同上] 用户→CF-Agent→A2A 通信

5. Stargate Agent - handle_message()
   a. 解析消息内容
   b. LLM Tool 选择 → "query_employee_trend"
   c. 判断 mode=mcp，执行 mcp 路径：
      i.   通过 SSE 连接 http://localhost:3005/sse
           POST { method: "tools/call", name: "query_employee_trend" }
      
      ii.  MCP Server 返回：
           {
             "_meta": { "ui": { "resourceUri": "ui://stargate/employee-trend" } },
             "content": [{ type: "text", ... }],
             "resource": { uri, mimeType, text }
           }
      
      iii. SG-Agent 调自身 GET /api/employee/trend
           ↓ 返回：{ data, token }
      
      iv.  SG-Agent 使用 MCP Server 提供的 resourceUri（ui://stargate/employee-trend）
           构造 A2A Response
      
      v.   （后续流程与 endpoint 模式相同）

6-15. [同上] 前端渲染与交互
```

---

### 六、关键交互特性

#### 1. **事件时序图**（Sequence Diagram）

前端 `SequenceDiagram.tsx` 从两个 SSE 源聚合事件，实时绘制参与者和消息箭头：

```
参与者：Frontend, CF-Agent, LLM, SG-Agent, MCP-Server, ResourceCenter, BusinessAPI

消息箭头示例：
Frontend → CF-Agent: chat（用户输入）
CF-Agent → LLM: llm-call（意图检测）
CF-Agent → SG-Agent: A2A Task（发送 A2A 消息）
SG-Agent → ResourceCenter: http（查询组件）
SG-Agent → BusinessAPI: http（查询数据）
SG-Agent → CF-Agent: A2A Response（返回结果）
CF-Agent → Frontend: response（返回前端）
Frontend → Frontend: render（渲染卡片）
```

#### 2. **沙箱代理机制**（sandbox_proxy.html）

```html
<!-- CF-Agent 3002 -->
  ↓ POST /chat
<!-- CodeFlicker 前端 -->
  ↓ <iframe src="/sandbox_proxy.html"> × 1 个（总容器）
<!-- sandbox_proxy.html（Proxy iframe）-->
  ↓ 监听 postMessage
  ├─ 如果收到 method='ui/notifications/sandbox-resource-ready'
  │  └─ 创建内部 <iframe sandbox="..."> 
  │     ├─ srcdoc 注入 HTML
  │     └─ 代理消息到主 iframe（双向）
  └─ 否则直接转发
<!-- 内部 iframe（Guest）-->
  ↓ 运行 MCP App HTML + EmployeeChart 组件
```

#### 3. **Mode 切换**（Endpoint vs MCP）

```
前端 按钮：[Endpoint] [MCP Server]

点击 → POST http://localhost:3002/mode { mode: "mcp" }
↓ CF-Agent 切换 current_mode
↓ 后续 chat 请求自动使用新 mode
↓ 对应 SG-Agent 的不同数据获取路径
```

---

### 七、部署与启动

**依赖服务**：
1. **Python 后端** - 端口 3001, 3002, 3005, 3011
2. **Node.js 前端 + Mock** - 端口 3000, 3003, 3004

**启动顺序**：
```bash
# 终端1：Python 后端
cd packages/codeflicker-agent && npm run dev      # 3002
cd packages/stargate-agent && npm run dev         # 3001, 3011
cd packages/stargate-mcp-ui-server && npm run dev # 3005

# 终端2：Node.js 前端
cd packages/codeflicker-frontend && npm run dev   # 3000
cd packages/resource-center-mock && npm run dev   # 3003
cd packages/employee-chart-card && npm run dev    # 3004（Webpack Dev Server）
```

**环境变量**：
```bash
# .env
OPENAI_API_KEY=sk-xxx
OPENAI_BASE_URL=https://api.openai.com/v1
LLM_MODEL=gpt-4o
```

---

### 八、架构设计决策

**ADR-001**：MCP-UI Server 职责边界
- MCP Server 仅负责提供 shell HTML（UIResource）
- 业务数据由 Agent 或前端组件自行获取
- 保持服务无状态、无数据依赖

**ADR-002**：UI 组件加载方式
- 选用 **Module Federation** 而非 ESM CDN 或 iframe
- 支持私有组件、独立部署、共享依赖（React singleton）

**ADR-003**：stargate-mcp-ui-server 新增
- 独立 MCP 服务器，完全遵循 MCP-UI 规范
- Tool + Resource 均定义在 MCP 服务内
- 支持多个 Agent 通过标准 MCP 协议访问

---

### 九、关键技术协议总结

| 协议 | 层级 | 作用 | 实现库 |
|------|------|------|--------|
| **A2A** | Agent 间通信 | 多 Agent 协作、跨框架互操作 | python-a2a |
| **MCP** | 模型与工具/数据 | 连接 LLM 与外部系统 | mcp (FastMCP) |
| **MCP-UI** | UI 层扩展 | 工具返回可渲染 UI 资源 | mcp-ui-server / @mcp-ui/client |
| **Module Federation** | 前端组件共享 | 运行时动态加载、共享依赖 | webpack 5+ |
| **postMessage** | iframe 通信 | 沙箱内外通信、JSON-RPC | 浏览器原生 |
| **SSE（Server-Sent Events）** | 服务器推送 | 事件流、事件时序图 | Python asyncio / JavaScript EventSource |

---

### 十、完整项目源代码文件清单

**总文件数**：8,259 个（包括依赖）

**核心源代码文件**：
```
主要 Python 源码：
  - codeflicker-agent/src/codeflicker_agent/main.py (184 行)
  - codeflicker-agent/src/codeflicker_agent/llm.py (34 行)
  - stargate-agent/src/stargate_agent/main.py (362 行)
  - stargate-agent/src/stargate_agent/llm.py (59 行)
  - stargate-agent/src/stargate_agent/card_cache.py (31 行)
  - stargate-agent/src/stargate_agent/shell_builder.py (57 行)
  - stargate-mcp-ui-server/src/stargate_mcp_ui_server/main.py (52 行)
  - stargate-mcp-ui-server/src/stargate_mcp_ui_server/tools.py (88 行)

主要 TypeScript/React 源码：
  - codeflicker-frontend/src/App.tsx (164 行)
  - codeflicker-frontend/src/types.ts (33 行)
  - codeflicker-frontend/src/components/ChatMessage.tsx (46 行)
  - codeflicker-frontend/src/components/CardMessage.tsx (75 行)
  - codeflicker-frontend/src/components/SequenceDiagram.tsx (333 行)
  - codeflicker-frontend/src/hooks/useEventLog.ts (36 行)
  - employee-chart-card/src/EmployeeChart.tsx (140 行)
  - employee-chart-card/src/EmployeeChartLazy.tsx (78 行)
  - resource-center-mock/src/index.ts (26 行)

配置与文档：
  - design.md (301 行) - 完整架构设计
  - a2a-protocol.md (104 行) - A2A 协议总结
  - mcp-ui-protocol.md (105 行) - MCP-UI 协议总结
  - a2a-protocol-fields.md (367 行) - A2A 字段详解
  - mcp-ui-protocol-fields.md (321 行) - MCP-UI 字段详解
  - docs/architecture-decisions.md (106 行) - ADR 记录
  - docs/designs/... - 多个设计文档
```

---

### 结论

这是一个**多 Agent 协作的演示系统**，深度整合了 A2A、MCP、MCP-UI 三层协议，展示了如何在标准化框架下实现：

1. **Agent 间通信**（A2A）：CodeFlicker 与 Stargate 通过 A2A 协议进行安全、标准化的协作
2. **工具与数据连接**（MCP）：通过 MCP 服务暴露工具、资源、提示
3. **Rich UI 交互**（MCP-UI）：工具执行结果可返回可渲染的 HTML，前端通过 iframe + Module Federation 加载并交互
4. **实时事件可视化**：SSE 事件流聚合，实时绘制时序图

整个系统设计清晰、职责明确，充分展示了现代 AI Agent 生态的可能性。
