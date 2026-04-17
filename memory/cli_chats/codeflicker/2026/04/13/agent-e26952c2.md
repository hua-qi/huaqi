---
session_id: agent-e26952c2
date: 2026-04-13
time_start: 2026-04-13T06:58:26.515Z
time_end: 2026-04-13T07:00:31.144Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: 
---

[2026-04-13 14:58:26] [user]: Please explore the codebase at /Users/lianzimeng/working/A2A-mcpUI thoroughly and return:
1. Overall directory structure (top-level and key subdirectories)
2. Key source files and their purposes
3. Package.json / dependencies
4. How the app is currently structured (components, routes, state management)
5. Any existing test setup
6. Any config files (tsconfig, vite/webpack, etc.)
Be very thorough - list all important files.
[2026-04-13 14:58:49] [assistant]: I'll conduct a thorough exploration of the codebase. Let me start by examining the directory structure and key files.
[2026-04-13 14:58:53] [assistant]: Now let me explore the key files and get more detailed information:
[2026-04-13 14:58:58] [assistant]: Now let me explore each package's package.json and key source files:
[2026-04-13 14:59:02] [assistant]: Now let me read the Python project files and frontend components:
[2026-04-13 14:59:05] [assistant]: Now let me explore the frontend components and hooks:
[2026-04-13 14:59:09] [assistant]: Now let me explore the Python agent code and server setup:
[2026-04-13 14:59:13] [assistant]: Let me continue exploring more agent files and card implementation:
[2026-04-13 14:59:18] [assistant]: Let me explore the remaining important files:
[2026-04-13 14:59:24] [assistant]: Now let me check for test files and configuration files:
[2026-04-13 14:59:27] [assistant]: Let me check the documentation files:
[2026-04-13 14:59:31] [assistant]: Let me check the remaining frontend files and the main entry points:
[2026-04-13 14:59:36] [assistant]: Let me check the sandbox proxy and any other key configuration:
[2026-04-13 15:00:31] [assistant]: Now let me compile the comprehensive report. Based on my thorough exploration, I have all the information needed:

## COMPREHENSIVE CODEBASE EXPLORATION REPORT

### 1. OVERALL DIRECTORY STRUCTURE

```
/Users/lianzimeng/working/A2A-mcpUI/
├── Root Package Files
│   ├── package.json              (monorepo root - pnpm)
│   ├── pnpm-workspace.yaml       (workspace config)
│   ├── pnpm-lock.yaml
│   ├── .env.example              (environment variables template)
│   ├── .gitignore
│
├── Documentation
│   ├── design.md                 (A2A protocol + Card rendering design)
│   ├── mcp-ui-protocol.md        (MCP UI protocol summary)
│   ├── mcp-ui-protocol-fields.md (protocol field reference)
│   ├── a2a-protocol.md           (A2A protocol summary)
│   ├── a2a-protocol-fields.md    (protocol field reference)
│   ├── docs/
│   │   ├── architecture-decisions.md (ADRs 001-003)
│   │   ├── designs/              (design proposals - 2026-04-10 to 2026-11-04)
│   │   └── plans/                (implementation plans)
│
├── packages/                     (pnpm monorepo)
│   ├── codeflicker-frontend/     (React + Vite - port 3000)
│   ├── codeflicker-agent/        (Python FastAPI - port 3002)
│   ├── stargate-agent/           (Python FastAPI + A2A Server - ports 3001/3011)
│   ├── stargate-mcp-ui-server/   (Python FastAPI MCP-UI Server - port 3005)
│   ├── employee-chart-card/      (React + Webpack Module Federation - port 3004)
│   └── resource-center-mock/     (Express TypeScript mock - port 3003)
│
└── node_modules/                 (shared dependencies)
```

---

### 2. KEY PACKAGES & THEIR PURPOSES

#### **A. codeflicker-frontend** - React UI Frontend
- **Port**: 3000
- **Role**: Chat interface for interacting with agents
- **Tech Stack**: React 18.3.1, TypeScript, Vite 5.2.0, @mcp-ui/client 7.0.0
- **Key Files**:
  - `/src/App.tsx` - Main chat app component
  - `/src/components/ChatMessage.tsx` - Renders text & card messages
  - `/src/components/CardMessage.tsx` - Renders MCP-UI resources via AppRenderer
  - `/src/components/EventLog.tsx` - Real-time event stream display
  - `/src/hooks/useEventLog.ts` - EventSource hook for SSE logs
  - `/src/types.ts` - TypeScript interfaces (TextPart, McpUiResourcePart)
  - `/public/sandbox_proxy.html` - iframe sandbox communication proxy
- **Key Features**:
  - Sends chat messages to CodeFlicker Agent (`/chat`)
  - Receives `mcp_ui_resource` parts with card URIs
  - Uses `@mcp-ui/client` AppRenderer to render cards
  - Displays real-time event logs from agents (SSE)
  - Proxy resource requests to agent (`/resource-proxy`)

#### **B. codeflicker-agent** - Python A2A Client Agent
- **Port**: 3002 (FastAPI), calls 3011 (Stargate A2A endpoint)
- **Role**: Intent detection & A2A message routing
- **Tech Stack**: Python 3.11, FastAPI 0.135.3, python-a2a 0.5.0+, OpenAI
- **Key Files**:
  - `/src/codeflicker_agent/main.py` - FastAPI server with chat endpoint
  - `/src/codeflicker_agent/llm.py` - Intent detection via LLM (query_data / general_chat)
  - `/src/codeflicker_agent/sse_logger.py` - EventSource event streaming
- **Key Responsibilities**:
  - Detects user intent (whether to query data or general chat)
  - Routes `query_data` intents to Stargate Agent via A2A protocol
  - Receives A2A responses with mcp_ui_resource parts
  - Exposes `/chat` endpoint for frontend
  - Exposes `/resource-proxy` to forward resource requests to Stargate
  - Streams events via `/events` (SSE) for logging

#### **C. stargate-agent** - Python A2A Server Agent
- **Ports**: 3001 (FastAPI), 3011 (Flask A2A Server)
- **Role**: MCP-UI card generation & business logic
- **Tech Stack**: Python 3.11, FastAPI, Flask (A2A), mcp-ui-server 1.0.0+
- **Key Files**:
  - `/src/stargate_agent/main.py` - FastAPI + A2A message handler
  - `/src/stargate_agent/llm.py` - Tool selection via LLM (query_employee_trend)
  - `/src/stargate_agent/card_cache.py` - In-memory card instance store (1hr TTL)
  - `/src/stargate_agent/sse_logger.py` - Event streaming
- **Key Responsibilities**:
  - Receives A2A messages from CodeFlicker Agent
  - Calls resource-center-mock to fetch component metadata
  - Generates card instances (HTML shell for Module Federation)
  - Stores card instances in cache with UUID
  - Returns mcp_ui_resource with resourceUri (ui://stargate/card/{id})
  - Provides `/mcp/resources/read` endpoint to read cached cards as HTML
  - Provides `/api/card-instance/{id}` endpoint (for reference)
  - Provides `/api/employee/detail/{year}` endpoint for card interactions

#### **D. stargate-mcp-ui-server** - Python MCP-UI Server
- **Port**: 3005
- **Role**: Standalone MCP-UI server (alternative to stargate-agent's HTTP endpoints)
- **Tech Stack**: Python 3.11, mcp[server] 1.5.0+, FastMCP
- **Key Files**:
  - `/src/stargate_mcp_ui_server/main.py` - FastMCP server definition
  - `/src/stargate_mcp_ui_server/tools.py` - UI resource generation
  - `/tests/test_main.py` - Tool registration tests
  - `/tests/test_tools.py` - HTML generation tests
- **Key Features**:
  - Implements MCP-UI protocol (SSE + JSON-RPC)
  - Registers `query_employee_trend` tool
  - Returns `ui://stargate/employee-trend` resource URI
  - Generates complete HTML (inline React + Module Federation setup)
  - Fully compliant MCP-UI server (can replace stargate-agent's HTTP API)

#### **E. employee-chart-card** - React Module Federation Card
- **Port**: 3004 (Webpack dev server)
- **Role**: Reusable chart component loaded via Module Federation
- **Tech Stack**: React 18.3.1, ECharts 5.5.0, Webpack 5, Module Federation
- **Key Files**:
  - `/src/EmployeeChart.tsx` - Main chart component
  - `/src/index.tsx` - Dev entry point
  - `/webpack.config.js` - Module Federation setup (exposes ./EmployeeChart)
  - `/public/index.html` - Dev HTML
- **Key Features**:
  - Receives `data` prop (year/count array) and `token`
  - Renders line chart with ECharts
  - Notifies parent via `postMessage` about size changes
  - Fetches year details via `/api/employee/detail/{year}` (with Bearer token)
  - Supports "分析趋势" button to send message back to agent
  - Exposes as Module Federation remote: `employeeChartCard/EmployeeChart`
  - Singleton-shared React 18.3.1 with host

#### **F. resource-center-mock** - Component Registry Mock
- **Port**: 3003
- **Role**: Mock component registry for Module Federation lookup
- **Tech Stack**: Express 4.19.2, TypeScript 5.4.0
- **Key Files**:
  - `/src/index.ts` - Express server
- **Key Endpoints**:
  - `GET /api/components/EmployeeChart` → returns `{componentName, containerName, remoteEntryUrl}`
  - `GET /health`
- **Purpose**: Provides component metadata (remoteEntryUrl) used by agents to generate cards

---

### 3. PACKAGE.JSON & DEPENDENCIES

#### **Root Package**
```json
{
  "name": "a2a-mcpui",
  "private": true,
  "engines": { "node": ">=20" },
  "scripts": {
    "dev": "pnpm --parallel --stream -r run dev",
    "build": "pnpm -r run build"
  }
}
```

#### **Frontend Dependencies**
```
React 18.3.1, react-dom 18.3.1
@mcp-ui/client 7.0.0 (MCP-UI SDK for rendering)
TypeScript 5.4.0, Vite 5.2.0
```

#### **Python Common Dependencies** (pyproject.toml)
```
python-a2a >= 0.5.0 (A2A protocol client/server)
mcp[server] >= 1.5.0 (Model Context Protocol)
mcp-ui-server >= 1.0.0 (MCP-UI SDK)
openai >= 1.30.0 (LLM)
fastapi >= 0.111.0 (Web framework)
uvicorn[standard] >= 0.29.0 (ASGI server)
httpx >= 0.27.0 (Async HTTP client)
python-dotenv >= 1.0.1 (Env config)
```

#### **Card Component Dependencies**
```
echarts 5.5.0 (Charting library)
React 18.3.1, react-dom 18.3.1
Webpack 5.91.0 with Module Federation plugin
```

---

### 4. APP ARCHITECTURE & COMPONENT STRUCTURE

#### **Architecture Layers**
```
┌─────────────────────────────────────┐
│    Frontend (React + MCP-UI SDK)    │ Port 3000
│  ├─ ChatMessage renderer            │
│  ├─ CardMessage (AppRenderer)       │
│  └─ EventLog (SSE streams)          │
└──────────────┬──────────────────────┘
               │ /chat, /resource-proxy
┌──────────────▼──────────────────────┐
│  CodeFlicker Agent (FastAPI)        │ Port 3002
│  ├─ Intent detection (LLM)          │
│  ├─ A2A message routing             │
│  └─ Resource proxy                  │
└──────────────┬──────────────────────┘
               │ A2A protocol (HTTP/JSON-RPC)
┌──────────────▼──────────────────────┐
│  Stargate Agent (FastAPI + A2A)     │ Port 3001/3011
│  ├─ Tool selection (LLM)            │
│  ├─ Card instance generation        │
│  ├─ In-memory cache (card_cache)    │
│  ├─ Resource retrieval              │
│  └─ Business data APIs              │
└──────────────┬──────────────────────┘
               │ Component lookup
┌──────────────▼──────────────────────┐
│  Resource Center Mock (Express)     │ Port 3003
│  ├─ Component metadata registry     │
│  └─ remoteEntryUrl provider         │
└─────────────────────────────────────┘
               │ Module Federation load
┌──────────────▼──────────────────────┐
│  Employee Chart Card (React MF)     │ Port 3004
│  ├─ ECharts visualization           │
│  ├─ Interactive details             │
│  └─ Token-based API calls           │
└─────────────────────────────────────┘

Parallel: Stargate MCP-UI Server      Port 3005
          (Alternative MCP impl)
```

#### **Component Hierarchy - Frontend**
```
App.tsx
├── Chat panel (left)
│   ├─ Messages array state
│   ├─ Message input
│   ├─ ChatMessage (for each message)
│   │   ├─ CardMessage (if mcp_ui_resource)
│   │   │   └─ @mcp-ui/client AppRenderer
│   │   │       └─ Sandbox proxy iframe
│   │   │           └─ Remote EmployeeChart component
│   │   └─ Text parts (styled divs)
│   └─ Event log connection
└── EventLog panel (right)
    └─ Real-time event stream display
```

#### **State Management**
- **Frontend**: Local React state (useState, useRef)
  - `messages[]` - chat history
  - `input` - user input box
  - `loading` - request pending
  - `iframeHeight` - dynamic card height
- **Backend Agents**: No persistent state (stateless HTTP)
  - Cache: stargate-agent uses in-memory `card_cache` (1 hour TTL)
  - Session: Each request is independent
  - EventLog: Async queues for SSE emission

---

### 5. TEST SETUP

#### **stargate-mcp-ui-server Tests** (Only test suite found)
- **Framework**: pytest 9.0.3, pytest-asyncio 1.3.0
- **Location**: `/packages/stargate-mcp-ui-server/tests/`

**Test Files**:
1. **test_main.py** - MCP server structure tests
   - `test_mcp_server_name()` - Validates server name
   - `test_tool_registered()` - Checks tool registration
   - `test_query_employee_trend_returns_resource_uri()` - Validates tool output

2. **test_tools.py** - UI resource generation tests
   - `test_resource_uri_is_static()` - URI constant validation
   - `test_build_html_contains_employee_data()` - HTML content check
   - `test_build_html_contains_react_scripts()` - React/ReactDOM inclusion
   - `test_get_ui_resource_structure()` - Response schema validation

**pyproject.toml Configuration**:
```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"

[dependency-groups]
dev = [
    "pytest>=9.0.3",
    "pytest-asyncio>=1.3.0",
]
```

**No tests for**:
- codeflicker-frontend (React components)
- codeflicker-agent
- stargate-agent
- employee-chart-card

---

### 6. CONFIG FILES

#### **A. TypeScript Configs**

**codeflicker-frontend/tsconfig.json**:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true
  }
}
```

**employee-chart-card/tsconfig.json** (similar):
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true
  }
}
```

#### **B. Build Configs**

**codeflicker-frontend/vite.config.ts**:
```typescript
defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/chat': 'http://localhost:3002',
      '/resource-proxy': 'http://localhost:3002'
    }
  }
})
```

**employee-chart-card/webpack.config.js**:
- ModuleFederationPlugin with singleton React
- Exposes `./EmployeeChart` as `EmployeeChart`
- Public path: `http://localhost:3004/`
- Dev server: port 3004, CORS enabled

#### **C. Python Configs**

**codeflicker-agent/pyproject.toml**:
```toml
[project]
name = "codeflicker-agent"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "python-a2a>=0.5.0",
    "openai>=1.30.0",
    "fastapi>=0.111.0",
    "uvicorn[standard]>=0.29.0",
    "httpx>=0.27.0",
    "python-dotenv>=1.0.1",
]
```

**stargate-agent/pyproject.toml** (adds):
- `mcp[server]>=1.5.0`
- `mcp-ui-server>=1.0.0`

**stargate-mcp-ui-server/pyproject.toml** (simpler):
```toml
[project]
dependencies = [
    "mcp[server]>=1.5.0",
    "mcp-ui-server>=1.0.0",
    "uvicorn[standard]>=0.29.0",
    "python-dotenv>=1.0.1",
]
```

#### **D. Workspace Config**

**pnpm-workspace.yaml**:
```yaml
packages:
  - "packages/*"
```

#### **E. Environment Config**

**.env.example**:
```
OPENAI_API_KEY=sk-your-key-here
OPENAI_BASE_URL=https://api.openai.com/v1
LLM_MODEL=gpt-4o
```

**Used by**:
- codeflicker-agent (intent detection)
- stargate-agent (tool selection)

---

### 7. KEY SOURCE FILES - COMPLETE LIST

#### **Frontend Source Files**
| File | Purpose |
|------|---------|
| `/packages/codeflicker-frontend/index.html` | HTML entry |
| `/packages/codeflicker-frontend/src/main.tsx` | React root |
| `/packages/codeflicker-frontend/src/App.tsx` | Main chat component (109 lines) |
| `/packages/codeflicker-frontend/src/types.ts` | TS interfaces (27 lines) |
| `/packages/codeflicker-frontend/src/components/ChatMessage.tsx` | Message renderer (36 lines) |
| `/packages/codeflicker-frontend/src/components/CardMessage.tsx` | Card renderer with AppRenderer (52 lines) |
| `/packages/codeflicker-frontend/src/components/EventLog.tsx` | Event display (26 lines) |
| `/packages/codeflicker-frontend/src/hooks/useEventLog.ts` | SSE hook (23 lines) |
| `/packages/codeflicker-frontend/public/sandbox_proxy.html` | iframe proxy (59 lines) |

#### **Agent Source Files**

**CodeFlicker Agent**:
| File | Purpose |
|------|---------|
| `/packages/codeflicker-agent/src/codeflicker_agent/main.py` | FastAPI server (95 lines) |
| `/packages/codeflicker-agent/src/codeflicker_agent/llm.py` | Intent detection (34 lines) |
| `/packages/codeflicker-agent/src/codeflicker_agent/sse_logger.py` | Event streaming (31 lines) |

**Stargate Agent**:
| File | Purpose |
|------|---------|
| `/packages/stargate-agent/src/stargate_agent/main.py` | A2A server & FastAPI (234 lines) |
| `/packages/stargate-agent/src/stargate_agent/llm.py` | Tool selection (47 lines) |
| `/packages/stargate-agent/src/stargate_agent/card_cache.py` | Instance cache (31 lines) |
| `/packages/stargate-agent/src/stargate_agent/sse_logger.py` | Event streaming (31 lines) |

**Stargate MCP-UI Server**:
| File | Purpose |
|------|---------|
| `/packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/main.py` | FastMCP server (34 lines) |
| `/packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/tools.py` | HTML generation (67 lines) |

#### **Card Component Source Files**
| File | Purpose |
|------|---------|
| `/packages/employee-chart-card/src/EmployeeChart.tsx` | Chart component (92 lines) |
| `/packages/employee-chart-card/src/index.tsx` | Dev entry (16 lines) |
| `/packages/employee-chart-card/public/index.html` | Dev HTML (6 lines) |

#### **Other Services**
| File | Purpose |
|------|---------|
| `/packages/resource-center-mock/src/index.ts` | Mock registry (26 lines) |

#### **Test Files**
| File | Purpose |
|------|---------|
| `/packages/stargate-mcp-ui-server/tests/test_main.py` | Server tests (26 lines) |
| `/packages/stargate-mcp-ui-server/tests/test_tools.py` | Tool tests (31 lines) |

#### **Documentation Files**
| File | Purpose |
|------|---------|
| `/design.md` | Card rendering & protocol design (301 lines) |
| `/mcp-ui-protocol.md` | MCP-UI protocol summary (100+ lines) |
| `/a2a-protocol.md` | A2A protocol summary (100+ lines) |
| `/docs/architecture-decisions.md` | ADRs 001-003 (106 lines) |
| `/mcp-ui-protocol-fields.md` | Protocol field reference |
| `/a2a-protocol-fields.md` | Protocol field reference |

---

### 8. DATA FLOW & COMMUNICATION PATTERNS

#### **Chat Message Flow**
```
1. User types message in Frontend → App state update
2. POST /chat (Frontend → CodeFlicker Agent)
3. CodeFlicker Agent:
   - Detect intent (LLM)
   - If "query_data": Send A2A message to Stargate Agent
4. Stargate Agent:
   - Select tool (LLM) → query_employee_trend
   - Fetch component metadata from Resource Center
   - Create card instance → store in cache
   - Generate mcp_ui_resource response
5. CodeFlicker Agent receives A2A response
6. Frontend receives JSON with parts[]
7. If part.kind === "mcp_ui_resource":
   - Extract resourceUri (ui://stargate/card/{id})
   - Pass to CardMessage/AppRenderer
8. AppRenderer:
   - Call onReadResource → GET /resource-proxy?uri=...
   - Receive HTML shell from Stargate Agent
   - Create iframe with sandbox + srcdoc
   - Load remote EmployeeChart via Module Federation
9. Card renders, notifies parent of height via postMessage
10. User clicks card button (e.g., "分析趋势")
11. Card sends postMessage to SDK
12. SDK sends new A2A message → loop back to step 3
```

#### **Event Streaming (SSE)**
```
Frontend hooks EventSource → /events (CodeFlicker Agent)
                         → /events (Stargate Agent)
Both emit SSE events via sse_logger.emit()
Frontend displays in EventLog panel
```

#### **API Endpoints by Service**

**CodeFlicker Agent (3002)**:
- `GET /health` - Health check
- `GET /events` - SSE event stream
- `POST /chat` - Main chat endpoint
- `GET /resource-proxy?uri=...` - Proxy resource requests

**Stargate Agent (3001)**:
- `GET /health`
- `GET /events` - SSE stream
- `GET /mcp/resources/read?uri=...` - MCP resource retrieval
- `GET /api/card-instance/{id}` - Card metadata (cache reference)
- `POST /api/token/exchange` - Mock token exchange
- `GET /api/employee/detail/{year}` - Business data

**Stargate Agent A2A (3011)**:
- `POST /.well-known/agent-card` - Agent discovery
- `POST /sendMessage` - A2A message sync
- `POST /sendMessageStream` - A2A message with SSE

**Resource Center Mock (3003)**:
- `GET /api/components/{name}` - Component metadata
- `GET /health`

**Employee Chart Card (3004)**:
- Webpack dev server (serves remoteEntry.js)
- HTML/JS for Module Federation

**Stargate MCP-UI Server (3005)**:
- MCP-UI protocol (SSE + JSON-RPC 2.0)
- Tools: query_employee_trend
- Resources: ui://stargate/employee-trend

---

### 9. KEY DESIGN PATTERNS & DECISIONS

#### **From ADR-001: MCP-UI Server Responsibility Boundary**
- MCP-UI server returns only UI (HTML shell)
- Business data is fetched by component at runtime
- Server remains stateless and data-agnostic
- No card_cache in MCP-UI server

#### **From ADR-002: Module Federation for Component Loading**
- Chosen over ESM CDN (no private component support)
- Chosen over iframe (performance + shared React)
- Chosen over Web Components (integration complexity)
- Enables singleton React shared between host and remote

#### **From ADR-003: Standalone stargate-mcp-ui-server**
- New Python service for MCP-UI protocol compliance
- Tool + resource defined in mcp-server
- Can eventually replace stargate-agent's HTTP endpoints
- Positions toward MCP-UI adoption

#### **Card Instance Caching**
- UUID-based instance IDs
- 1-hour TTL (CONFIGURABLE)
- In-memory store (could be Redis/DB later)
- Prevents data exposure in URLs

#### **Token Flow**
- CodeFlicker → Stargate A2A (A2A protocol handles auth)
- Card uses mock token "mock-stargate-token-12345"
- Frontend calls card details API with Bearer token
- Component receives token via initial props

---

### 10. WORKFLOW & STARTUP SEQUENCE

#### **Development Startup Order**
```bash
1. npm i / pnpm install          (Install all deps)
2. export OPENAI_API_KEY=sk...   (Set env)
3. pnpm dev                      (Start all services in parallel):
   - resource-center-mock :3003  (Component registry)
   - employee-chart-card :3004   (Webpack Module Federation)
   - stargate-mcp-ui-server :3005 (MCP-UI server, optional)
   - codeflicker-agent :3002     (A2A client, intent detection)
   - stargate-agent :3001/:3011  (A2A server, card generation)
   - codeflicker-frontend :3000  (React chat UI)
```

#### **Typical User Interaction**
```
1. User opens http://localhost:3000
2. Types: "查询快手历年员工人数趋势"
3. Sends message → POST /chat
4. CodeFlicker Agent detects intent=query_data
5. Sends A2A to Stargate Agent
6. Stargate generates card instance
7. Frontend receives mcp_ui_resource
8. Card renders with chart
9. User clicks "2021 详情"
10. Card fetches /api/employee/detail/2021 with token
11. Details displayed in card
```

---

### 11. IMPORTANT NOTES ON ARCHITECTURE

1. **No Database** - All state is ephemeral:
   - Frontend: React state only
   - Agents: Stateless request/response (cache is in-memory)

2. **Protocol Layers**:
   - HTTP/REST: Frontend ↔ CodeFlicker Agent, Stargate Agent APIs
   - A2A (HTTP + JSON-RPC): CodeFlicker Agent ↔ Stargate Agent
   - MCP-UI: (Optional) SSE + JSON-RPC for resource protocol

3. **Module Federation Setup**:
   - Host: Frontend (React app)
   - Remote: EmployeeChart (webpack remote entry at 3004)
   - Container: employeeChartCard
   - Shared: React 18.3.1 (singleton)

4. **Security Considerations**:
   - Cards loaded in sandboxed iframe
   - Token-based API access
   - Resource URIs point to server-managed instances

5. **Scalability**:
   - Stateless agents (can scale horizontally)
   - Cache is in-memory (use Redis for scale)
   - Each component is independently deployable

---

This concludes the comprehensive codebase exploration. All important files, structures, dependencies, and architectural decisions have been documented.
