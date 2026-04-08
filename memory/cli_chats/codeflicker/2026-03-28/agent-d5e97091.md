---
session_id: agent-d5e97091
date: 2026-03-28
time_start: 2026-03-28T09:46:06.147Z
time_end: 2026-03-28T09:47:37.723Z
project: users-lianzimeng-working-kd-toft-chat-bot
git_branch: feat-multi-modal
---

[2026-03-28 17:46:06] [user]: 探索 /Users/lianzimeng/working/chat-bot-cards/ 项目的架构，重点关注：
1. 项目类型（前端/后端/全栈）
2. 使用的技术栈和框架
3. 目录结构
4. 是否有 package.json 或其他配置文件
5. 启动/开发命令
6. API 接口定义位置
7. 环境变量配置

请提供详细的架构分析报告。
[2026-03-28 17:46:15] [assistant]: 我来系统地探索这个项目的架构。让我从目录结构和关键配置文件开始分析。
[2026-03-28 17:47:37] [assistant]: 根据我的探索，以下是 `/Users/lianzimeng/working/chat-bot-cards/` 项目的详细架构分析报告：

---

# chat-bot-cards 项目架构分析报告

## 1. 项目类型

**前端组件库项目** - 基于 Module Federation 的微前端卡片组件库

这是一个专门为 Chat Bot 应用提供可复用 UI 卡片组件的项目，通过 Module Federation 技术实现组件的动态加载和远程调用。

---

## 2. 技术栈和框架

### 核心技术
| 类别 | 技术/框架 | 版本 |
|------|----------|------|
| **UI 框架** | React | ^17.0.2 |
| **开发语言** | TypeScript | ^5.5.2 |
| **构建工具** | Rsbuild (基于 Rspack) | ^1.3.2 |
| **微前端** | Module Federation (@module-federation/enhanced) | ^0.5.0 |
| **CLI 工具** | @toft/ai-cli | ^0.0.5-beta.1 |

### UI 组件库
| 类别 | 技术 | 版本 |
|------|------|------|
| **企业级组件** | Ant Design | ^5.17.4 |
| **内部组件库** | @toft/react | ^1.3.7 |
| **Pro 组件** | @toft/react-pro | ^1.1.8-feat-118.55 |

### 功能库
| 类别 | 技术 | 版本 |
|------|------|------|
| **状态管理** | immer | ^10.1.1 |
| **HTTP 请求** | axios | ^1.7.2 |
| **工具函数** | lodash | ^4.17.21 |
| **图表** | ECharts + echarts-for-react | ^5.5.0 |
| **代码编辑器** | Monaco Editor | ^4.7.0 |
| **Markdown 渲染** | react-markdown | ^10.1.0 |
| **流程图** | mermaid | ^11.10.1 |
| **PDF 预览** | pdfjs-dist + react-pdf | 5.4.394 |
| **样式预处理** | Less | ^4.2.0 |

---

## 3. 目录结构

```
chat-bot-cards/
├── ai-card-dist/                 # 构建产物（Module Federation 远程入口）
│   ├── remoteEntry.js            # Module Federation 入口文件
│   └── static/                   # 静态资源
│       ├── css/                  # 样式文件
│       ├── js/                   # JavaScript 文件
│       ├── font/                 # 字体文件
│       └── image/                # 图片资源
│
├── src/                          # 源代码目录
│   ├── components/               # 组件目录
│   │   ├── CodeCard/             # 代码展示卡片
│   │   ├── ConsoleCard/          # 控制台卡片
│   │   ├── TextCard/             # 文本卡片
│   │   ├── MarkdownCard/         # Markdown 渲染卡片
│   │   ├── FileCard/             # 文件预览卡片
│   │   │   ├── DocxRenderer/     # Word 文档渲染
│   │   │   ├── ExcelRenderer/    # Excel 渲染
│   │   │   ├── PdfRenderer/      # PDF 渲染
│   │   │   ├── ImageRenderer/    # 图片渲染
│   │   │   ├── JsonRenderer/     # JSON 渲染
│   │   │   ├── MarkdownRenderer/ # Markdown 渲染
│   │   │   └── TextRenderer/     # 文本渲染
│   │   ├── ImageCard/            # 图片卡片
│   │   ├── HtmlCard/             # HTML 卡片
│   │   ├── TableCard/            # 表格卡片
│   │   ├── TableFlashDataCard/   # 闪存数据表格卡片
│   │   ├── SqlCard/              # SQL 卡片
│   │   ├── EchartsOptionCard/    # ECharts 配置卡片
│   │   ├── FilterChartsCard/     # 筛选图表卡片
│   │   ├── DeptInfoCard/         # 部门信息卡片
│   │   ├── EmpInfoCard/          # 员工信息卡片
│   │   ├── EmpKpiCard/           # 员工 KPI 卡片
│   │   ├── EmpRankCard/          # 员工排名卡片
│   │   ├── EmpTimelineCard/      # 员工时间线卡片
│   │   ├── IndBarChartCard/      # 指标柱状图卡片
│   │   ├── IndLineChartCard/     # 指标折线图卡片
│   │   ├── SubBoardCard/         # 子看板卡片
│   │   ├── UnauthorizedCard/     # 无权限卡片
│   │   ├── RefreshCard/          # 刷新卡片
│   │   ├── NullDataCard/         # 空数据卡片
│   │   ├── CodeAndConsoleCard/   # 代码+控制台组合卡片
│   │   ├── KaleidoFrameCard/     # Kaleido 框架卡片
│   │   ├── common/               # 公共组件
│   │   │   ├── ChatBotMessageCard/  # 聊天消息卡片
│   │   │   ├── CopyButton/       # 复制按钮
│   │   │   ├── FileAnnotation/   # 文件标注
│   │   │   ├── FilePreviewDrawer/ # 文件预览抽屉
│   │   │   ├── FileTable/        # 文件表格
│   │   │   ├── JumpToButton/     # 跳转按钮
│   │   │   └── KpiItem/          # KPI 项
│   │   ├── types.ts              # 类型定义
│   │   └── utils.ts              # 工具函数
│   │
│   ├── hooks/                    # 自定义 Hooks
│   │   └── useProcessTableData.tsx
│   │
│   ├── services/                 # 服务层
│   │   └── fetch.ts              # Axios 实例配置
│   │
│   ├── assets/                   # 静态资源
│   ├── style/                    # 全局样式
│   ├── App.tsx                   # 开发调试应用
│   ├── bootstrap.tsx             # 应用启动入口
│   ├── index.tsx                 # 入口文件
│   └── env.d.ts                  # 类型声明
│
├── index.ts                      # 组件导出入口
├── ai-card.config.mjs            # Module Federation 配置
├── package.json                  # 项目配置
├── tsconfig.json                 # TypeScript 配置
└── pnpm-lock.yaml               # 依赖锁定文件
```

---

## 4. 配置文件详解

### package.json 关键配置

```json
{
  "name": "ihr-cards",
  "type": "module",
  "scripts": {
    "start-card": "ai-cli start",        // 启动开发服务器
    "release-card": "ai-cli release",    // 发布构建
    "release-card:test": "ai-cli release:test"  // 测试环境发布
  }
}
```

### ai-card.config.mjs - Module Federation 配置

```javascript
export default defineConfig({
  name: "ihr_widget_provider",           // 模块名称
  filename: "remoteEntry.js",            // 远程入口文件名
  version: "0.0.54-beta.4",              // 版本号
  exposes: {                             // 暴露的组件
    "./CodeCard": "./src/components/CodeCard/CodeCard.tsx",
    "./TextCard": "./src/components/TextCard/TextCard.tsx",
    "./MarkdownCard": "./src/components/MarkdownCard/MarkdownCard.tsx",
    "./FileCard": "./src/components/FileCard/FileCard.tsx",
    // ... 更多组件
  },
  server: {
    port: 3000,                          // 开发服务器端口
    proxy: {                             // API 代理配置
      "/api/runtime/": {
        target: "https://kaleido.staging.kuaishou.com",
        changeOrigin: true
      }
    }
  }
});
```

---

## 5. 启动/开发命令

| 命令 | 说明 |
|------|------|
| `pnpm start-card` | 启动开发服务器（端口 3000） |
| `pnpm release-card` | 生产环境构建发布 |
| `pnpm release-card:test` | 测试环境构建发布 |

---

## 6. API 接口定义位置

### 服务层文件
- **主要文件**: `/src/services/fetch.ts`

### Axios 实例配置

```typescript
// 请求拦截器 - 添加自定义请求头
axiosInstance.interceptors.request.use((config) => {
  config.headers["switchUsername"] = "liuyouchao";
  config.headers["appType"] = "HR";
  return config;
});

// 响应拦截器 - 统一错误处理
axiosInstance.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response) {
      toast.error("网络出了点小问题，请稍后重试");
    }
    return Promise.reject(error);
  }
);
```

### API 代理配置
- **开发环境代理**: 
  - `/api/runtime/application/` -> `https://kaleido.staging.kuaishou.com`
  - `/api/runtime/gxMzjBmX7BRg` -> `https://kaleido.staging.kuaishou.com`

### 组件中的数据请求
各卡片组件通过 `requestBody` prop 接收请求参数，由父组件统一管理数据获取。

---

## 7. 环境变量配置

**未发现 .env 文件**，项目通过以下方式管理环境配置：

1. **代理配置**: 在 `ai-card.config.mjs` 中配置 API 代理
2. **请求头注入**: 在 `fetch.ts` 中通过拦截器注入请求头
3. **硬编码配置**: 部分配置直接写在代码中（如 switchUsername）

---

## 8. 核心特性

### Module Federation 微前端架构
- 作为远程模块暴露组件
- 支持运行时动态加载
- 主应用可通过 `remoteEntry.js` 引用组件

### 卡片组件体系

| 类别 | 组件 | 用途 |
|------|------|------|
| **基础展示** | TextCard, MarkdownCard, HtmlCard, ImageCard | 文本/富媒体展示 |
| **代码相关** | CodeCard, SqlCard, CodeAndConsoleCard, ConsoleCard | 代码展示与执行 |
| **文件预览** | FileCard | 多格式文件预览（PDF/Excel/Word/图片等） |
| **数据可视化** | TableCard, EchartsOptionCard, IndBarChartCard, IndLineChartCard, FilterChartsCard | 数据表格与图表 |
| **业务卡片** | DeptInfoCard, EmpInfoCard, EmpKpiCard, EmpRankCard, EmpTimelineCard | HR 业务数据展示 |
| **状态卡片** | UnauthorizedCard, NullDataCard, RefreshCard, CardLoading | 状态提示 |

---

## 9. 组件接口规范

所有卡片组件遵循统一的接口定义：

```typescript
interface ICard {
  cardType: string;                                    // 卡片类型
  params: any;                                         // 卡片参数
  hasAuth: boolean;                                    // 权限状态
  loading: boolean;                                    // 加载状态
  requestBody: any;                                    // 请求体
  setRequestBody: React.Dispatch<any>;                 // 设置请求体
  loadingForRerequest: boolean;                         // 重新请求加载状态
  setLoadingForRerequest: React.Dispatch<boolean>;     // 设置重新请求状态
  totalItemsForTableCard?: number;                     // 表格总条数
  onJumpClick: (url: string) => void;                  // 跳转回调
}
```

---

## 10. 构建产物

- **输出目录**: `ai-card-dist/`
- **入口文件**: `remoteEntry.js`
- **资源分离**: CSS/JS/Font/Image 独立目录
- **SourceMap**: 生成对应的 .map 文件便于调试
- **代码分割**: 异步加载优化

---

## 总结

这是一个**基于 Module Federation 的前端卡片组件库**，采用 React + TypeScript + Rsbuild 技术栈，为 Chat Bot 应用提供可复用的 UI 组件。项目特点：

1. **微前端架构** - 支持 Module Federation 远程加载
2. **丰富的组件** - 20+ 卡片组件覆盖多种展示场景
3. **文件预览能力** - 支持 PDF/Excel/Word/图片等多种格式
4. **图表可视化** - 集成 ECharts 提供强大的图表能力
5. **企业级组件** - 基于 Ant Design 和内部组件库构建
