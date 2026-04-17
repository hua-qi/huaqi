---
session_id: agent-b68fe3b8
date: 2026-04-14
time_start: 2026-04-14T11:50:27.053Z
time_end: 2026-04-14T11:52:28.466Z
project: users-lianzimeng-working-kaleido
git_branch: hotfix-reusable-widget-scope
---

[2026-04-14 19:50:27] [user]: 在 /Users/lianzimeng/working/kaleido 项目中，判断哪个是主应用（入口应用）。请查看：
1. 根目录 package.json 的 scripts，看默认 dev/start/build 指向哪个
2. websites/ 下各应用的 package.json，看哪个有最多依赖或最复杂的配置
3. 是否有 nginx 或部署配置指示主入口
4. 是否有路由聚合或微前端配置（如 qiankun、module federation）

返回：哪个是主应用，以及判断依据。
[2026-04-14 19:50:32] [assistant]: 我来帮你分析这个项目，找出主应用（入口应用）。让我先获取 git remote URL，然后系统地查看项目结构。
[2026-04-14 19:50:36] [assistant]: 现在让我查询相关经验，然后开始系统地探索项目结构：
[2026-04-14 19:50:41] [assistant]: 好的，现在让我查看项目结构和各个关键配置文件：
[2026-04-14 19:50:46] [assistant]: 很好，现在让我查看 websites/ 下各应用的 package.json 和关键配置文件：
[2026-04-14 19:50:51] [assistant]: 现在让我查看 pnpm-workspace.yaml 和部署相关配置：
[2026-04-14 19:50:55] [assistant]: 现在让我查看各应用的 vite 配置和源代码入口：
[2026-04-14 19:51:03] [assistant]: 让我检查 docs 应用的配置：
[2026-04-14 19:51:07] [assistant]: 现在让我检查各应用的 nginx 配置和 vite 配置中的关键信息：
[2026-04-14 19:51:13] [assistant]: 让我检查 workspace 应用的 nginx 配置，因为它似乎是最核心的应用：
[2026-04-14 19:51:18] [assistant]: 现在让我查看 vite 配置和源代码结构，了解更多关于应用的架构：
[2026-04-14 19:51:22] [assistant]: 现在让我查看根目录的 Dockerfile 和启动脚本来了解部署配置：
[2026-04-14 19:51:27] [assistant]: 让我查看更多 README 详情：
[2026-04-14 19:51:30] [assistant]: 现在让我查看关键的启动脚本和 CI 配置，以及是否有关于微前端架构的文档：
[2026-04-14 19:51:36] [assistant]: 现在让我查看根目录启动脚本和 start.sh：
[2026-04-14 19:51:41] [assistant]: 让我查看 scripts/start.js：
[2026-04-14 19:51:47] [assistant]: 让我查看各应用的入口 HTML 文件和分析应用的依赖：
[2026-04-14 19:51:52] [assistant]: 现在让我统计各应用的依赖数量和复杂度：
[2026-04-14 19:51:56] [assistant]: 现在让我查看各应用中是否有特殊的配置和路由信息：
[2026-04-14 19:52:01] [assistant]: 让我查看平台和系统管理的路由配置：
[2026-04-14 19:52:10] [assistant]: 现在让我总结一下所有的发现，并调用 save_experience：
[2026-04-14 19:52:28] [assistant]: 完美！现在让我为你生成完整的分析报告。

## 分析报告：Kaleido 项目主应用确定

### 结论
**`@kaleido/workspace` 是主应用（入口应用）**

---

### 判断依据

#### 1. **根目录 Package.json Scripts 指向**
在根目录 `/Users/lianzimeng/working/kaleido/package.json` 中，主要的 dev 脚本包括：
```json
"workspace:dev": "pnpm workspace dev",
"platform:dev": "pnpm platform dev", 
"runtime:dev": "pnpm runtime dev",
"runtime-v2:dev": "pnpm runtime-v2 dev"
```
- `workspace:dev` 排在最前面，作为首要开发命令
- workspace 应用对应最完整的开发流程支持

#### 2. **Applications Under websites/ 的配置对比**

| 应用 | 名称 | 说明 | 构建配置 | 依赖性质 |
|------|------|------|--------|---------|
| **workspace** | @kaleido/workspace | IDE编辑器核心 | 32768MB内存 | 最复杂的代理配置 |
| platform | @kaleido/platform | 首页/应用列表 | 16384MB内存 | 基础展示 |
| runtime | @kaleido/runtime | 应用运行时 | 16384MB内存 | 执行层 |
| runtime-v2 | @kaleido/runtime-v2 | 新版运行时 | 16384MB内存 | 执行层(包含SDK) |
| sys-admin | @kaleido/sys-admin | 系统管理 | 16384MB内存 | 管理工具 |
| console | console | 应用控制台 | 16384MB内存 | 简单展示 |

**关键发现：**
- workspace 需要 **32768MB** 内存（最高），说明业务复杂度最大
- workspace 的 vite.config.ts 包含完整的开发代理配置

#### 3. **Workspace Vite 配置特征** (`/websites/workspace/vite.config.ts`)

```typescript
// 完整的开发代理配置
proxy: {
  '/java/api/workspace': {...},      // 工作区后端
  '/magic-ai': {...},                 // AI功能
  '/lsp': {...},                      // 语言服务
  '/node/api/workspace': {...},       // Builder Server
  '/external-api/workspace/...': {...},
  '/api/runtime': {...}               // 运行时API
}

// 主题定制配置（最复杂）
TOFT_MODIFY_VARS_LIST = {
  '@prefix-cls': 'kaleido-ide',
  '@pro-prefix-cls': 'kaleido-ide-pro',
  // ... 150+ 行主题变量
}
```
- 这些配置完全不存在于其他应用
- workspace 是唯一需要连接多个后端服务的应用

#### 4. **Nginx 部署配置指示** (`/websites/workspace/scripts/nginx.conf`)

```nginx
server {
    listen 8080;
    server_name ~-ide\..*$;  # 主域名前缀: kaleido-ide
    # ...
}

# 重定向逻辑：多个域名重定向到 workspace
location ~* ^/(w|workspace|)/ {
    return 302 https://kaleido-ide.corp.kuaishou.com$request_uri;
}
```

**关键信息：**
- workspace 对应主域名 `kaleido-ide`（IDE 入口）
- runtime 域名重定向到 workspace，说明 runtime 是从 workspace 入口跳转的
- workspace 是流量入口

#### 5. **微前端/路由聚合架构**

**Docs 应用** (使用 qiankun 微前端)：
```typescript
// .umirc.ts
qiankun: {
  master: {
    apps: [
      { name: 'kaleido/builder-widgets', entry: '...' },
      { name: 'kaleido/entity-editor', entry: '...' },
      { name: 'kaleido/page-editor', entry: '...' },
      { name: 'kaleido/nav-editor', entry: '...' },
      { name: 'kaleido/flow-editor', entry: '...' },
      { name: 'kaleido/flow-nodes', entry: '...' },
    ]
  }
}
```
- docs 是文档聚合应用
- workspace 是**编辑功能主体**（不使用微前端，而是直接导入库）

**Workspace 的库导入** (main.tsx)：
```typescript
import * as kaleidoCommonComponents from '@kaleido/common-components';
import * as KaleidoWidgetCore from '@kaleido/widget-core';
import * as KaleidoWidgetCoreBuilder from '@kaleido/widget-core/builder';
import { useGenerateDataWithWidgetsMap } from '@kaleido/page-editor';

// 直接在 workspace 中使用
Reflect.set(window, '__useGenerateDataWithWidgetsMap', useGenerateDataWithWidgetsMap);
```
- workspace 直接集成所有编辑功能
- 这是主应用才会做的集成方式

#### 6. **应用入口标识**

| 应用 | 入口HTML标题 | 说明 |
|------|-----------|------|
| **workspace** | **"Kaleido IDE"** | 🔵 IDE核心入口 |
| platform | (platform) | 首页入口 |
| runtime | "kaleido render" | 运行时 |
| runtime-v2 | (无标题) | 新版运行时 |
| sys-admin | "Kaleido IDE" | 系统管理 |
| console | "Kaleido 应用控制台" | 应用控制台 |

- workspace 和 sys-admin 共享"Kaleido IDE"身份
- workspace 包含完整的编辑器功能

---

### 总结架构关系图

```
Entry Flow:
┌─ Platform (首页/应用列表) 
│   └─> Workspace (IDE 编辑)
│       ├─> Runtime (应用执行)
│       ├─> Runtime-V2 (新版执行)
│       └─> 包含所有编辑器库
│
├─ Sys-Admin (系统管理)
├─ Console (应用控制台)
└─ Docs (文档聚合)
    ├─ Builder-Widgets (微前端)
    ├─ Page-Editor
    ├─ Flow-Editor
    └─ 其他编辑器...
```

**主应用：`@kaleido/workspace`** (位置：`/Users/lianzimeng/working/kaleido/websites/workspace/`)
