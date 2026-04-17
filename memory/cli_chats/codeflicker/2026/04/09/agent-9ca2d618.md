---
session_id: agent-9ca2d618
date: 2026-04-09
time_start: 2026-04-09T08:36:48.206Z
time_end: 2026-04-09T08:37:27.295Z
project: users-lianzimeng-working-a2a-mcpui
git_branch: 
---

[2026-04-09 16:36:48] [user]: 探索 /Users/lianzimeng/working/A2A-mcpUI 项目的整体结构，重点关注：
1. 项目整体目录结构
2. 消息/card 渲染相关的代码文件
3. 是否有文档文件（md文件）描述了 card 渲染逻辑
4. 消息内容的数据类型定义
请返回：目录结构概览、card渲染相关文件路径、文档文件路径、以及消息内容的类型定义
[2026-04-09 16:36:52] [assistant]: 我来帮你探索这个项目的整体结构和关键文件。让我从几个方面进行全面的搜索。
[2026-04-09 16:36:56] [assistant]: 有趣！让我继续探索更多目录结构和代码文件：
[2026-04-09 16:37:06] [assistant]: 看起来这个项目比较简洁。让我查看所有文件：
[2026-04-09 16:37:10] [assistant]: 现在让我读取这些关键文档文件来理解项目结构和 card 渲染逻辑：
[2026-04-09 16:37:27] [assistant]: 完美！现在我有了完整的信息。让我为你整理一份详细的报告：

## 项目探索报告

### 1. **项目整体目录结构**

```
/Users/lianzimeng/working/A2A-mcpUI/
├── design.md                  (核心：A2A 协议渲染 Card 设计方案)
├── mcp-ui-protocol.md         (MCP UI 协议文档)
└── a2a-protocol.md            (A2A 协议文档)
```

**说明**：这是一个设计文档项目，主要包含三份关键协议和设计规范文档。没有代码实现文件（.ts、.tsx、.js等）。

---

### 2. **Card 渲染相关的核心文档**

**主要文档**：`/Users/lianzimeng/working/A2A-mcpUI/design.md`

**Card 渲染核心内容**：
- **A2A 协议扩展**：在 message 的 `parts` 中新增 `kind: "mcp_ui_resource"` 类型
- **关键数据结构**：
  ```json
  {
    "data": {
      "kind": "mcp_ui_resource",
      "resourceUri": "ui://stargate/card/{cardInstanceId}",
      "uiMetadata": {
        "preferred-frame-size": { "width": 400, "height": 300 }
      }
    },
    "mediaType": "application/json"
  }
  ```

- **渲染链路三个阶段**：
  1. **Card Instance 生产阶段**：Stargate Agent 生成 cardInstanceId
  2. **前端渲染阶段**：SDK 通过 cardInstanceId 换取实例、token，使用 Module Federation 加载卡片
  3. **卡片交互阶段**：卡片使用 stargateToken 调用业务 API

---

### 3. **消息内容的数据类型定义**

#### **A2A Message 结构**
```typescript
{
  "message": {
    "role": "ROLE_AGENT",           // Agent 角色
    "parts": [                       // 消息部分数组
      {
        "data": {                    // 核心数据
          "kind": "mcp_ui_resource",
          "resourceUri": "ui://stargate/card/{cardInstanceId}",
          "uiMetadata": {
            "preferred-frame-size": { "width": number, "height": number }
          }
        },
        "mediaType": "application/json"
      }
    ]
  }
}
```

#### **Card Instance 数据结构**
```typescript
{
  cardInstanceId: string,          // 实例 ID
  componentName: string,            // 组件名称
  remoteEntryUrl: string,          // Module Federation 入口 URL
  props: object                    // 业务数据（敏感数据）
}
```

#### **Token 注入数据**
```typescript
{
  stargateToken: string            // 注入到卡片的访问 token
}
```

#### **卡片交互数据（postMessage）**
```typescript
// 方案 A：卡片直接调用 API
{ /* 普通业务事件 */ }

// 方案 B：经由 Agent 链路处理
{
  type: "agent_submit",
  payload: object                  // 表单数据
}
```

---

### 4. **相关协议文档文件路径**

| 文件路径 | 用途 |
|---------|------|
| `/Users/lianzimeng/working/A2A-mcpUI/design.md` | **A2A 协议渲染 Card 设计方案**（P0 项目核心） |
| `/Users/lianzimeng/working/A2A-mcpUI/mcp-ui-protocol.md` | **MCP UI 协议总结**（UI 层协议基础） |
| `/Users/lianzimeng/working/A2A-mcpUI/a2a-protocol.md` | **A2A 协议总结**（Agent 间通信基础） |

---

### 5. **架构各模块职责总结**

| 模块 | 职责 |
|---|---|
| **Stargate 资源管理中心** | 存储卡片的 componentName + remoteEntryUrl 索引 |
| **Stargate Agent** | 组装 card instance，管理缓存，鉴权，换 token |
| **A2A 协议扩展** | parts 里携带 `mcp_ui_resource` + `resourceUri` |
| **CodeFlicker Agent** | 识别并透传 `mcp_ui_resource`（不做额外处理） |
| **前端渲染 SDK** | 串行完成：换 instance → 换 token → MF 加载 → 渲染 |
| **卡片组件库** | 独立构建，暴露 remoteEntry.js，自带交互逻辑 |

---

### 6. **关键设计决策**

1. **为什么用 cardInstanceId**：敏感数据留在服务端，前端持 token 换取，后端在此做鉴权和脱敏
2. **为什么用 Module Federation**：支持动态加载，直接消费索引，无需额外构建
3. **Token 注入方式**：先用 CodeFlicker token 换 Stargate token，再通过 `initial-render-data` 注入

---

**总结**：该项目是一套完整的 A2A 协议扩展方案设计文档，用于在 CodeFlicker 中渲染 Stargate Agent 返回的业务自定义卡片。关键是通过 `mcp_ui_resource` 协议扩展、cardInstanceId 机制和 Module Federation 实现安全、可扩展的卡片渲染系统。
