---
session_id: agent-90b7492f
date: 2026-03-28
time_start: 2026-03-28T09:46:05.989Z
time_end: 2026-03-28T09:47:57.734Z
project: users-lianzimeng-working-kd-toft-chat-bot
git_branch: feat-multi-modal
---

[2026-03-28 17:46:05] [user]: 探索 /Users/lianzimeng/working/kd.toft.chat-bot/ 项目的架构，重点关注：
1. 项目类型（前端/后端/全栈）
2. 使用的技术栈和框架
3. 目录结构
4. 是否有 package.json 或其他配置文件
5. 启动/开发命令
6. API 接口定义位置
7. 环境变量配置

请提供详细的架构分析报告。
[2026-03-28 17:46:14] [assistant]: 我来帮你详细探索这个项目的架构。让我先查看项目的基本结构和配置文件。
[2026-03-28 17:47:57] [assistant]: # 项目架构分析报告

## 一、项目概述

**项目名称**: `@toft/chat-bot`  
**项目版本**: `0.0.212`  
**项目描述**: HR首页AI助手  
**项目类型**: **前端应用**（同时支持 Web 应用和浏览器插件）

---

## 二、技术栈与框架

### 2.1 核心技术栈

| 技术领域 | 技术选型 | 版本 |
|---------|---------|------|
| **前端框架** | React | ^17.0.2 |
| **开发语言** | TypeScript | ^5.8.3 |
| **构建工具** | Rsbuild | ^1.3.22 |
| **状态管理** | Zustand | ^4.5.2 |
| **路由管理** | React Router DOM | 5.2.1 |
| **样式方案** | Less | - |
| **HTTP 请求** | Axios + @toft/utils (fetch) | ^1.9.0 |
| **UI 组件库** | @toft/react + @toft/react-pro | 内部组件库 |

### 2.2 重要依赖库

- **echarts / echarts-for-react**: 图表渲染
- **monaco-editor**: 代码编辑器
- **tiptap**: 富文本编辑器
- **dnd-kit**: 拖拽功能
- **@microsoft/fetch-event-source**: SSE 流式响应
- **wxt**: 浏览器插件开发框架（支持 Chrome/Firefox）
- **immer**: 不可变数据管理
- **ahooks**: React Hooks 工具库
- **swr**: 数据请求与缓存

### 2.3 开发工具链

```json
{
  "代码规范": {
    "ESLint": "^9.28.0",
    "Prettier": "^3.5.3",
    "Stylelint": "^16.20.0",
    "Commitlint": "^19.8.1",
    "Husky": "^9.1.7"
  }
}
```

---

## 三、目录结构分析

```
/Users/lianzimeng/working/kd.toft.chat-bot/
├── public/                      # 静态资源
│   ├── index.html              # HTML 模板
│   └── *_favicon.ico           # 各业务线 favicon
├── src/
│   ├── apis/                   # API 接口定义
│   │   └── chat/               # 聊天相关接口
│   │       ├── V2.ts           # V2 版本 API
│   │       ├── Init.ts         # 初始化接口
│   │       ├── Kim.ts          # Kim 接口
│   │       ├── Login.ts        # 登录接口
│   │       ├── Task.ts         # 任务接口
│   │       ├── types.ts        # 类型定义（Swagger 生成）
│   │       └── ...
│   ├── app/                    # 应用入口
│   │   ├── AppRouter.tsx       # 路由配置
│   │   ├── Layout/             # 布局组件
│   │   ├── Sidebar/            # 侧边栏
│   │   ├── RightPanel/         # 右侧面板
│   │   └── configs/            # 路由/导航配置
│   ├── assets/                 # 静态资源（图片、SVG、Lottie）
│   ├── components/             # 通用组件
│   │   ├── ChatBot/            # ChatBot 核心组件
│   │   ├── ChatBotMessage/     # 消息组件
│   │   ├── ChatBotUserInput/   # 用户输入组件
│   │   ├── CardRenderer/       # 卡片渲染器
│   │   ├── Stream/             # 流式消息组件
│   │   └── ...
│   ├── common-components/      # 公共组件
│   │   ├── Uploader/           # 文件上传
│   │   ├── SendButton/         # 发送按钮
│   │   └── ...
│   ├── modules/                # 业务模块
│   │   ├── HrChatBot/          # HR 业务线
│   │   ├── StargateChatBot/    # Stargate 业务线
│   │   ├── StargateHRChatBot/  # Stargate HR 业务线
│   │   ├── IntelligenceChatBot/# 智能业务线
│   │   └── common-pages/       # 公共页面
│   ├── stores/                 # Zustand 状态管理
│   │   ├── useAppStore.ts      # 应用状态
│   │   ├── useConversationStore.ts # 会话状态
│   │   ├── useBotStore.ts      # Bot 状态
│   │   ├── useStreamStore.ts   # 流式消息状态
│   │   └── ...
│   ├── services/               # 服务层
│   │   ├── fetch.ts            # Axios 实例
│   │   └── event.ts            # 事件服务
│   ├── hooks/                  # 自定义 Hooks
│   ├── utils/                  # 工具函数
│   │   ├── request.ts          # 请求封装
│   │   ├── baseUrl.ts          # 基础 URL 配置
│   │   ├── env.ts              # 环境判断
│   │   └── ...
│   ├── types/                  # TypeScript 类型定义
│   ├── style/                  # 全局样式
│   ├── const.ts                # 常量定义
│   ├── index.tsx               # 应用入口
│   └── web-extension-src/      # 浏览器插件源码
│       ├── entrypoints/         # 插件入口
│       │   ├── background.ts   # 后台脚本
│       │   ├── content/        # Content Script
│       │   └── sidepanel/      # 侧边栏面板
│       └── public/             # 插件资源
├── rsbuild.config.ts           # Rsbuild 配置
├── wxt.config.ts               # 浏览器插件配置
├── tsconfig.json               # TypeScript 配置
├── package.json                # 项目配置
└── pnpm-lock.yaml              # 依赖锁定文件
```

---

## 四、配置文件详解

### 4.1 package.json 核心脚本

```json
{
  "scripts": {
    "prepare": "husky",                        // Git hooks 初始化
    "dev": "rsbuild dev",                      // 本地开发服务器
    "start": "rsbuild dev",                    // 同 dev
    "build": "rsbuild build",                  // 生产环境构建
    "api": "node scripts/api.mjs && ...",      // API 接口生成
    
    // 浏览器插件相关
    "wxt-dev": "wxt",                          // 插件开发
    "wxt-build": "wxt build",                  // 插件构建
    "wxt-zip": "wxt zip"                       // 插件打包
  }
}
```

### 4.2 Rsbuild 配置要点

**文件位置**: `/rsbuild.config.ts`

```typescript
{
  // 开发服务器配置
  server: {
    port: 3000,
    open: true,
    proxy: {
      '/magic-ai': {
        target: 'https://stargate.staging.kuaishou.com/',
        changeOrigin: true
      },
      '/api/gateway': {
        target: 'https://wanxiang-uat.corp.kuaishou.com/'
      }
    }
  },
  
  // 输出配置
  output: {
    distPath: { root: 'build' },
    sourceMap: { js: 'source-map' }
  },
  
  // Less 主题定制
  less: {
    modifyVars: {
      '@prefix-cls': 'toft',
      'color-theme-default': '#1626FF'
    }
  }
}
```

### 4.3 TypeScript 配置

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "jsx": "react-jsx",
    "moduleResolution": "bundler",
    "baseUrl": "./",
    "paths": {
      "@src/*": ["./src/*"],
      "@utils/*": ["./src/utils/*"]
    }
  }
}
```

---

## 五、启动/开发命令

### 5.1 Web 应用开发

```bash
# 安装依赖
pnpm install

# 启动开发服务器（端口 3000）
pnpm dev
# 或
pnpm start

# 生产环境构建
pnpm build
```

### 5.2 浏览器插件开发

```bash
# 插件开发模式
pnpm wxt-dev

# 构建 Chrome 插件
pnpm wxt-build

# 构建 Firefox 插件
pnpm wxt-build:firefox

# 打包为 zip
pnpm wxt-zip
```

### 5.3 API 接口生成

```bash
# 从 Swagger 生成 API 接口和类型定义
pnpm api
```

---

## 六、API 接口定义

### 6.1 API 接口位置

**主要目录**: `/src/apis/chat/`

| 文件名 | 功能描述 |
|-------|---------|
| `V2.ts` | V2 版本核心接口（发送消息、获取历史、意图分类等） |
| `Init.ts` | 初始化接口 |
| `Kim.ts` | Kim 相关接口 |
| `Login.ts` | 登录接口 |
| `Task.ts` | 任务相关接口 |
| `Openapi.ts` | OpenAPI 接口 |
| `PersonalDataSource.ts` | 个人数据源接口 |
| `types.ts` | **Swagger 自动生成的类型定义（5921 行）** |

### 6.2 核心 API 示例

**文件位置**: `/src/apis/chat/V2.ts`

```typescript
// 发送消息（SSE 流式响应）
export const postChatRouteSendMessage = (data: SendMessageRequest) =>
  fetch<any>(`${baseUrl.prefix}/v2/api/chat/route-send-message`, {
    method: 'POST',
    data: data
  });

// 获取历史消息
export const getChatConversationsConversationIdMessagesRecent = (...) => 
  fetch<ResultListMessageResponseV2>(...);

// 意图分类
export const postChatIntent_classify = (data: IntentClassifyRequest) => 
  fetch<ResultIntentClassifyResponse>(...);

// @功能搜索
export const getChatAtSearch = (query: AtSearchV2ApiChatAtSearchGetParams) => 
  fetch<ResultList>(...);
```

### 6.3 API Base URL 配置

**文件位置**: `/src/utils/baseUrl.ts`

```typescript
const baseUrl = {
  prefix: '/magic-ai'  // 所有 API 的统一前缀
};
```

### 6.4 请求封装

**文件位置**: `/src/utils/request.ts`

```typescript
// 统一请求包装器
export const requestWrap = (fetch, fallbackResult) => {
  // 自动处理 code !== 0 的错误
  // 返回 result 数据
};

// 带错误提示的请求包装器
export const requestWrapWithACE = (fetch, fallbackResult) => {
  // 自动捕获错误并显示 message.error
};
```

---

## 七、环境变量配置

### 7.1 环境类型定义

**文件位置**: `/src/const.ts`

```typescript
export const ENV_CONFIG = [
  { host: 'localhost:3000', env: 'STAGING' },
  { host: 'stargate.staging.kuaishou.com', env: 'STAGING' },
  { host: 'stargate-home.corp.kuaishou.com', env: 'PREONLINE' },
  { host: 'stargate.corp.kuaishou.com', env: 'PROD' },
  { host: 'lumos.corp.kuaishou.com', env: 'PROD' }
];
```

### 7.2 环境判断工具

**文件位置**: `/src/utils/env.ts`

```typescript
// 获取当前环境
export const getCurrentEnv = (): EnvType => {
  return ENV_CONFIG.find(e => e.host === window.location.host)?.env || 'STAGING';
};

// 获取个人数据源 URL
export const getPersonalDataSourceUrl = (code?: string): string => {
  const env = getCurrentEnv();
  return PersonalDataSourceUrlMap[env];
};
```

### 7.3 业务相关环境配置

```typescript
// 个人数据源系统地址映射
export const PersonalDataSourceUrlMap = {
  STAGING: 'https://kaleido.staging.kuaishou.com/app/...',
  PREONLINE: 'https://kaleido.corp.kuaishou.com/app/...',
  PROD: 'https://kaleido.corp.kuaishou.com/app/...'
};

// 文件加密 Key
export const FileEncryptionKey = {
  STAGING: 'magicAI.file',
  PREONLINE: 'magicAI.file',
  PROD: 'ee.ihr.ai.clientEncrypt'
};
```

### 7.4 代理配置（开发环境）

**文件位置**: `/rsbuild.config.ts`

```typescript
proxy: {
  '/magic-ai': {
    target: 'https://stargate.staging.kuaishou.com/',
    changeOrigin: true,
    onProxyRes: (proxyRes, req, res) => {
      // 强制设置 SSE 响应头
      if (req.url?.includes('/route-send-message')) {
        proxyRes.headers['Content-Type'] = 'text/event-stream';
      }
    }
  }
}
```

---

## 八、业务模块架构

### 8.1 多业务线支持

项目支持多个业务线，通过 **AppType** 区分：

| AppType | 业务线 | URL Domain |
|---------|-------|-----------|
| `HR` | HR 业务 | `/hr` |
| `FINANCE` | 财务业务 | `/fin` |
| `HR_ANALYTICS` | HR 分析 | `/hr_analytics` |
| `KURSOR` | Kursor 业务 | `/kursor` |
| `INTELLIGENCE` | 智能业务 | `/intelligence` |
| `ZONGHE` | 综合业务 | `/zonghe` |
| `FINANCE_IR` | 财务 IR | `/finance_ir` |

**定义位置**: `/src/const.ts`

```typescript
export const AppType2DomainMap = {
  HR: 'hr',
  FINANCE: 'fin',
  HR_ANALYTICS: 'hr_analytics',
  // ...
};
```

### 8.2 路由架构

**文件位置**: `/src/app/configs/routerConfig.tsx`

```typescript
// 路由结构
[
  // 分享看板
  { path: '/:domain/share/dashboard/:shareDashboardId', component: DashboardSharePage },
  
  // ChatBot 主路由
  { 
    path: '/:domain', 
    layout: Layout,
    children: [
      { path: '', component: ChatBot },
      { path: '/c/:conversationId', component: ChatBot },
      { path: '/dashboard/:dashboardId', component: DashboardPage },
      { path: '/space/:spaceId', component: ChatBot },
      { path: '/report', component: ReportPage }
    ]
  },
  
  // 浏览器插件路由
  { path: '/e/:domain', layout: Layout, children: [...] },
  
  // Artifacts 路由
  { path: '/artifacts', component: ArtifactsPanelContainer }
]
```

### 8.3 状态管理架构

使用 **Zustand** 进行状态管理，主要 Store 包括：

| Store | 文件路径 | 功能 |
|-------|---------|-----|
| `useAppStore` | `/src/stores/useAppStore.ts` | 应用全局状态（终端类型、插件配置） |
| `useConversationStore` | `/src/stores/useConversationStore.ts` | 会话管理 |
| `useBotStore` | `/src/stores/useBotStore.ts` | Bot 信息 |
| `useStreamStore` | `/src/stores/useStreamStore.ts` | 流式消息状态 |
| `useUserInfoStore` | `/src/stores/useUserInfoStore.ts` | 用户信息与权限 |
| `useSidebarStore` | `/src/stores/useSidebarStore.ts` | 侧边栏状态 |
| `useArtifactsStore` | `/src/stores/useArtifactsStore.ts` | Artifacts 面板状态 |

---

## 九、浏览器插件架构

### 9.1 插件配置

**文件位置**: `/wxt.config.ts`

```typescript
{
  name: 'Stargate AI',
  version: '0.0.3',
  permissions: ['storage', 'activeTab', 'scripting', 'tabs', 'sidePanel'],
  
  // 入口点
  entrypoints: {
    background: 'background.ts',
    content: 'content/index.tsx',
    sidepanel: 'sidepanel/App.tsx'
  }
}
```

### 9.2 插件源码目录

```
src/web-extension-src/
├── entrypoints/
│   ├── background.ts           # 后台脚本
│   ├── content/                # Content Script（注入页面）
│   └── sidepanel/              # 侧边栏面板
└── public/
    └── icon/                   # 插件图标
```

---

## 十、关键特性总结

### 10.1 核心功能

1. **多业务线支持**: 通过 AppType 支持多个独立业务
2. **流式消息**: 支持 SSE (Server-Sent Events) 流式响应
3. **多模态输入**: 支持文本、图片、文件等多种输入类型
4. **卡片系统**: 支持多种卡片类型（Markdown、Code、HTML、Image、File、SubBoard 等）
5. **看板功能**: 支持收藏卡片到看板、导出文档
6. **浏览器插件**: 支持 Chrome/Firefox 双浏览器插件
7. **移动端适配**: 自动检测移动端并重定向

### 10.2 技术亮点

- **Swagger 自动生成 API**: 使用 `swagger-typescript-api` 自动生成类型定义
- **模块化设计**: 组件、Hooks、Utils 高度模块化
- **类型安全**: 全面的 TypeScript 类型定义
- **状态管理**: Zustand 轻量级状态管理
- **样式方案**: Less + CSS Modules
- **代码规范**: ESLint + Prettier + Stylelint + Husky

### 10.3 API 类型定义

项目使用 Swagger 自动生成 API 类型定义，**types.ts 文件高达 5921 行**，包含：

- 请求/响应接口定义
- 枚举类型
- 业务数据模型

---

## 十一、开发建议

### 11.1 本地开发

```bash
# 1. 安装依赖
pnpm install

# 2. 启动开发服务器
pnpm dev

# 3. 访问 http://localhost:3000/hr
```

### 11.2 API 开发

```bash
# 更新 API 定义
pnpm api
```

### 11.3 插件开发

```bash
# 开发模式
pnpm wxt-dev

# 在浏览器中加载 dist 目录
```

---

## 十二、总结

这是一个**企业级 React 前端应用**，主要特点：

1. **多业务线架构**: 支持 HR、Finance 等多个业务线
2. **现代化技术栈**: React 17 + TypeScript + Rsbuild + Zustand
3. **流式消息支持**: SSE 实现实时消息推送
4. **多端适配**: Web 应用 + 浏览器插件
5. **完善的工程化**: 代码规范、自动化构建、类型安全
6. **灵活的扩展性**: 模块化设计，易于添加新业务线
