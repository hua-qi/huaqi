---
session_id: agent-df05551f
date: 2026-04-14
time_start: 2026-04-14T07:59:38.686Z
time_end: 2026-04-14T08:03:27.935Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: main
---

[2026-04-14 15:59:38] [user]: 在 /Users/lianzimeng/working/A2A-mcpUI 项目中，找到以下内容：
1. `mcp_ui_server` Python 包的源码位置（可能在 packages 目录或者安装在虚拟环境里）
2. `create_ui_resource()` 函数的实现，特别是它是否支持 `content.type = "externalUrl"` 以及 `iframeUrl`
3. stargate-mcp-ui-server 里目前的 tool 实现，看是否有用到 externalUrl
4. 前端的 CardMessage.tsx 或 AppRenderer 相关代码，看是否处理了 externalUrl 类型（即直接把 URL 放进 iframe src）

返回关键代码片段和文件路径。
[2026-04-14 16:00:11] [assistant]: 我来帮你全面搜索这些内容。首先获取git remote URL并搜索相关经验。
[2026-04-14 16:00:21] [assistant]: 现在搜索项目中的相关经验：
[2026-04-14 16:00:31] [assistant]: 很好，有相关经验记录。现在让我开始全面搜索项目结构：
[2026-04-14 16:00:46] [assistant]: 很好，现在让我并行搜索关键文件的内容：
[2026-04-14 16:00:59] [assistant]: 现在继续搜索前端和其他相关文件：
[2026-04-14 16:01:06] [assistant]: 继续搜索更多相关文件：
[2026-04-14 16:01:26] [assistant]: 现在搜索前端的 App 和 AppRenderer 组件以及是否处理了 externalUrl：
[2026-04-14 16:01:44] [assistant]: 现在让我检查 stargate-agent 的 main.py 中对 create_ui_resource 的使用情况：
[2026-04-14 16:01:57] [assistant]: 让我查看 stargate-mcp-ui-server 的 main.py：
[2026-04-14 16:02:11] [assistant]: 完美！现在让我获取协议文档中关于 externalUrl 的更多信息，并查看 protocol-fields 中的完整示例：
[2026-04-14 16:02:25] [assistant]: 现在让我检查 ChatMessage 组件以了解前端如何处理 mcp_ui_resource：
[2026-04-14 16:02:55] [assistant]: 完美！现在让我生成最终的综合报告。让我先保存经验，然后返回报告：
[2026-04-14 16:03:27] [assistant]: 现在生成详细报告：

## 搜索结果总结

我已完整调查了你提出的四个关键问题。以下是详细的代码位置和实现分析：

---

## 1. `mcp_ui_server` Python 包源码位置

**主要源码位置：**
```
虚拟环境中的安装位置：
/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-mcp-ui-server/.venv/lib/python3.11/site-packages/mcp_ui_server/

关键文件：
- core.py      （create_ui_resource() 的实现）
- types.py     （数据类型定义）
- __init__.py  （公共 API 导出）
- exceptions.py
- utils.py
```

---

## 2. `create_ui_resource()` 函数实现及 `externalUrl` 支持

### 函数定义位置
**文件：** `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-mcp-ui-server/.venv/lib/python3.11/site-packages/mcp_ui_server/core.py`

### 关键代码片段 - 支持的内容类型

```python
def create_ui_resource(options_dict: dict[str, Any]) -> UIResource:
    """
    支持三种内容类型：
    1. rawHtml      - 直接传入 HTML 字符串
    2. externalUrl  - 传入外部 URL，通过 iframeUrl 字段
    3. remoteDom    - 远程 DOM（React 或 WebComponents）
    """
    options = CreateUIResourceOptions.model_validate(options_dict)
    
    # externalUrl 的处理逻辑（行 ~128-137）：
    elif content_type == "externalUrl":
        from .types import ExternalUrlPayload
        if not isinstance(content, ExternalUrlPayload):
            raise InvalidContentError("Content must be ExternalUrlPayload when type is 'externalUrl'")
        iframe_url = content.iframeUrl
        if not iframe_url:
            raise InvalidContentError(
                "content.iframeUrl must be provided as a non-empty string when content.type is 'externalUrl'"
            )
        actual_content_string = iframe_url
        mime_type = "text/uri-list"
```

### ExternalUrlPayload 类型定义

**文件：** `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-mcp-ui-server/.venv/lib/python3.11/site-packages/mcp_ui_server/types.py`

```python
class ExternalUrlPayload(BaseModel):
    """External URL content payload."""
    type: Literal["externalUrl"]
    iframeUrl: str

# 完整的内容类型定义
ResourceContentPayload = RawHtmlPayload | ExternalUrlPayload | RemoteDomPayload
```

### 使用示例

```python
# 创建 externalUrl 类型的 UI 资源
resource = create_ui_resource({
    "uri": "ui://my-widget",
    "content": {
        "type": "externalUrl",
        "iframeUrl": "https://my-server.com/widget"  # 这是关键字段
    },
    "encoding": "text",  # 或 "blob"
    "uiMetadata": {
        "preferred-frame-size": ["800px", "600px"]
    }
})
```

### MIME Type 映射
- `rawHtml`     → `text/html`
- `externalUrl` → `text/uri-list`
- `remoteDom`   → `application/vnd.mcp-ui.remote-dom+javascript; framework=react` 或 `framework=webcomponents`

---

## 3. Stargate-MCP-UI-Server 的 Tool 实现

**文件：** `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/tools.py`

### 当前实现（仅使用 rawHtml）

```python
def get_ui_resource() -> dict:
    resource = create_ui_resource({
        "uri": RESOURCE_URI,
        "content": {
            "type": "rawHtml",           # ← 当前只用 rawHtml
            "htmlString": build_html()   # ← 直接生成 HTML
        },
        "encoding": "text",
    })
    return resource.model_dump(mode="json")

def get_lazy_ui_resource() -> dict:
    resource = create_ui_resource({
        "uri": LAZY_RESOURCE_URI,
        "content": {
            "type": "rawHtml",
            "htmlString": build_lazy_html()
        },
        "encoding": "text",
    })
    return resource.model_dump(mode="json")
```

**MCP Server 主文件：** `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-mcp-ui-server/src/stargate_mcp_ui_server/main.py`

```python
@mcp.tool()
async def query_employee_trend() -> dict:
    r = get_ui_resource()
    return {
        "_meta": {"ui": {"resourceUri": RESOURCE_URI}},  # ← 工具通过 resourceUri 绑定 UI
        "content": [{"type": "text", "text": "已为您查询快手历年员工趋势数据，共 5 年记录。"}],
        "resource": r["resource"],
        "toolName": "query_employee_trend",
    }

@mcp.resource(RESOURCE_URI, mime_type="text/html;profile=mcp-app")
async def employee_trend_resource() -> str:
    r = get_ui_resource()
    return r["resource"]["text"]
```

**结论：** ❌ 当前 stargate-mcp-ui-server 中**未使用** `externalUrl` 类型

---

## 4. 前端 CardMessage.tsx 和 AppRenderer 的 externalUrl 处理

**主要文件：**

### CardMessage.tsx
**路径：** `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/CardMessage.tsx`

```typescript
export const CardMessage: React.FC<Props> = ({
  resourceUri,
  toolName,
  toolResult,
  uiMetadata,
  onMessage,
  onLayout,
}) => {
  // AppRenderer 是来自 @mcp-ui/client 的组件
  <AppRenderer
    ref={rendererRef}
    toolName={toolName as string}
    toolResult={toolResult as any}
    sandbox={{ url: sandboxUrl }}
    toolResourceUri={resourceUri}
    
    // ← 关键回调：处理资源读取
    onReadResource={async ({ uri }, extra) => {
      const source = extra && Object.keys(extra).length > 0 ? 'app' : 'host';
      const res = await fetch(`/resource-proxy?uri=${encodeURIComponent(uri)}&source=${source}`);
      return res.json();  // ← 通过代理获取资源内容
    }}
    
    onCallTool={handleCallTool as any}
    onSizeChanged={({ height }) => {
      if (height !== undefined) {
        setIframeHeight(height);
        onLayout?.();
      }
    }}
    // ...其他事件处理
  />
};
```

### ChatMessage.tsx
**路径：** `/Users/lianzimeng/working/A2A-mcpUI/packages/codeflicker-frontend/src/components/ChatMessage.tsx`

```typescript
// 前端如何处理 mcp_ui_resource 类型的消息部分
if (part.kind === 'mcp_ui_resource') {
  return (
    <CardMessage
      key={i}
      resourceUri={part.resourceUri}      // ← 传递资源 URI
      toolName={part.toolName}
      toolResult={part.toolResult}
      uiMetadata={part.uiMetadata}
      onMessage={onCardMessage}
      onLayout={onLayout}
    />
  );
}
```

### AppRenderer 工作流程

根据 `mcp-ui-protocol.md` 的描述，AppRenderer 的工作流程：

1. **接收 resourceUri** → 通过 `onReadResource` 回调读取资源
2. **解析 MIME Type** 
   - 如果 `mimeType = "text/html;profile=mcp-app"` → 作为 MCP App 在 iframe 中渲染
   - 如果 `mimeType = "text/uri-list"` → **externalUrl 类型**
3. **render.text/blob 内容** → 直接放入 iframe 的 `src` 或作为页面内容

### 前端是否支持 externalUrl？

**结论：** ⚠️ **部分支持但需验证**

- AppRenderer 理论上支持通过 MIME Type `text/uri-list` 标识 externalUrl 资源
- 但在当前代码中**未看到明确的 iframe src 绑定逻辑**
- 前端处理流程依赖 `@mcp-ui/client` 库内部实现（该库代码未在本项目中展示）

---

## 5. stargate-agent 中的 create_ui_resource 使用

**文件：** `/Users/lianzimeng/working/A2A-mcpUI/packages/stargate-agent/src/stargate_agent/main.py`

```python
# 行 176-181
resource = create_ui_resource({
    "uri": uri,
    "content": {
        "type": "rawHtml",  # ← 也只用 rawHtml
        "htmlString": html
    },
    "encoding": "text",
})
r = resource.model_dump(mode="json")
```

**结论：** ❌ stargate-agent 中也**未使用** `externalUrl` 类型

---

## 关键发现总结

| 维度 | 状态 | 详情 |
|------|------|------|
| **externalUrl 在 SDK 中的支持** | ✅ 完全支持 | `mcp_ui_server.create_ui_resource()` 完全支持 externalUrl 类型，字段为 `iframeUrl` |
| **stargate-mcp-ui-server 使用** | ❌ 未使用 | 仅使用 `rawHtml` 类型 |
| **stargate-agent 使用** | ❌ 未使用 | 仅使用 `rawHtml` 类型 |
| **前端 AppRenderer 支持** | ⚠️ 理论支持 | 通过 `text/uri-list` MIME Type 标识，但具体渲染逻辑在 @mcp-ui/client 库中 |
| **前端明确处理** | ❌ 不明确 | 未在项目代码中看到明确的 iframe src 绑定逻辑 |

---

## 协议文档中的定义

**文件：** `/Users/lianzimeng/working/A2A-mcpUI/mcp-ui-protocol-fields.md`

```markdown
| `content.type` | `"rawHtml"` \| `"externalUrl"` | 是 | 内容来源类型 |
| `content.iframeUrl` | string | externalUrl 必填 | 外部 URL，自动注入 `<base>` 标签 |
```

**文件：** `/Users/lianzimeng/working/A2A-mcpUI/mcp-ui-protocol.md`

```markdown
| 类型 | 说明 |
|---|---|
| `rawHtml` | 直接传入 HTML 字符串 |
| `externalUrl` | 传入外部 URL，服务端获取页面 HTML 并注入 `<base>` 标签 |
```
