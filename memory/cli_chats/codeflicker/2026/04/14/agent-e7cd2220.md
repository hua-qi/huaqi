---
session_id: agent-e7cd2220
date: 2026-04-14
time_start: 2026-04-14T11:40:16.712Z
time_end: 2026-04-14T11:42:50.971Z
project: users-lianzimeng-working-kaleido
git_branch: hotfix-reusable-widget-scope
---

[2026-04-14 19:40:16] [user]: 探索 /Users/lianzimeng/working/kaleido 项目的 Vite 和 monorepo 配置。请查看：
1. 根目录的 package.json、pnpm-workspace.yaml 或 lerna.json 等 monorepo 配置文件
2. 各个 vite.config.ts/js 文件
3. tsconfig.json 和相关配置
4. 根目录结构（packages/ 或 apps/ 目录结构）

请返回：
- monorepo 工具（pnpm workspace/lerna/nx 等）及配置内容
- workspace 包的组织方式
- vite 配置的关键内容（包括所有 vite.config 文件路径和主要配置项）
- 包之间的依赖关系
[2026-04-14 19:40:21] [assistant]: 我来帮你探索这个项目的 Vite 和 monorepo 配置。让我首先从获取 git remote URL 开始，然后搜索相关经验。
[2026-04-14 19:40:50] [assistant]: 现在让我搜索相关工程经验。
[2026-04-14 19:41:10] [assistant]: 现在开始探索项目结构和配置文件。让我并行获取多个关键信息：
[2026-04-14 19:41:24] [assistant]: 现在让我读取根目录的 tsconfig 和 nx.json 配置，以及几个关键的 vite.config 文件：
[2026-04-14 19:41:28] [assistant]: 现在让我检查几个包的 tsconfig 和 package.json，以及了解包的结构：
[2026-04-14 19:41:35] [assistant]: 现在让我检查一些关键的依赖关系和其他配置：
[2026-04-14 19:41:40] [assistant]: 现在让我查看一些widget的配置和检查project.json文件：
[2026-04-14 19:41:45] [assistant]: 让我检查 tsconfig.dumi.json 和检查packages目录结构：
[2026-04-14 19:41:50] [assistant]: 现在让我查看一些关键的依赖关系和 widget 配置：
[2026-04-14 19:41:55] [assistant]: Now let me get the dev-tools docs vite config:
[2026-04-14 19:42:02] [assistant]: Perfect! Now let me save the experience and compile the report:
[2026-04-14 19:42:50] [assistant]: ## Kaleido 项目 Vite 和 Monorepo 配置探索报告

### 1. Monorepo 工具与配置

**工具**: pnpm workspace + NX

**配置文件**:
- `/Users/lianzimeng/working/kaleido/pnpm-workspace.yaml` - pnpm workspace 配置
- `/Users/lianzimeng/working/kaleido/nx.json` - NX 任务执行配置
- `/Users/lianzimeng/working/kaleido/package.json` - 根项目配置

**pnpm-workspace.yaml 配置**:
```yaml
packages:
  - 'websites/*'    # 7个前端应用
  - 'packages/*'    # 核心库和工具包
  - 'servers/*'     # 后端服务
  - 'widgets/*'     # Widget库
  - 'config/*'      # 配置包
  - '!**/test/**'   # 排除测试目录
  - '!packages/dev-tools'  # 排除dev-tools（特殊处理）
```

**NX 缓存配置** (`nx.json`):
- 支持缓存操作: build, test, build:mobile, test:db, ci, ci:mobile
- 输出路径映射到 `{workspaceRoot}/dist/{projectRoot}`

---

### 2. Workspace 包的组织方式

#### **packages/** 目录 (14个库包)
主要包含可复用的组件库和工具库:

| 包名 | 用途 |
|------|------|
| `@kaleido/bpd-editor` | BPD流程编辑器 |
| `@kaleido/page-editor` | 页面编辑器 |
| `@kaleido/flow-editor` | 流程编辑器 |
| `@kaleido/entity-editor` | 实体编辑器 |
| `@kaleido/flow-nodes` | 流程节点库 |
| `@kaleido/nav-editor` | 导航编辑器 |
| `@kaleido/widget-core` | Widget核心库 |
| `@kaleido/builder-widgets` | 构建器Widgets |
| `@kaleido/common-components` | 通用组件库 |
| `@kaleido/compiler` | 编译器工具 |
| `@kaleido/template` | 模板工具 |
| `@kaleido/templates` | 模板定义 |
| `@kaleido/shared` | 共享工具库 |
| `@kaleido/debug` | 调试工具 |

#### **websites/** 目录 (7个应用)
完整的前端应用，基于 Vite 构建:

- `console` - 控制台应用
- `workspace` - 工作空间应用  
- `runtime` - 运行时应用
- `runtime-v2` - 运行时v2版本
- `platform` - 平台应用
- `sys-admin` - 系统管理应用
- `docs` - 文档应用 (基于 UmiJS)

#### **widgets/** 目录 (3个widget库)
独立的Widget库，支持跨应用加载:

- `kaleido-widgets` - 主Widget库 (UMD格式)
- `kaleido-mobile-widgets` - 移动端Widgets (UMD格式)
- `builder-widgets` - 构建器Widgets

#### **servers/** 目录
- `builder-server` - NestJS后端服务

#### **config/** 目录
- 共享配置包（dumi、release-it等）

---

### 3. Vite 配置的关键内容

#### **所有 vite.config 文件路径**:

**packages/库级配置**:
- `/Users/lianzimeng/working/kaleido/packages/bpd-editor/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/packages/common-components/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/packages/entity-editor/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/packages/flow-editor/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/packages/nav-editor/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/packages/page-editor/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/packages/compiler/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/packages/flow-nodes/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/packages/template/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/packages/widget-core/vite.config.ts`

**websites/应用级配置**:
- `/Users/lianzimeng/working/kaleido/websites/console/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/websites/workspace/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/websites/runtime/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/websites/runtime-v2/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/websites/platform/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/websites/sys-admin/vite.config.ts`

**widgets/配置**:
- `/Users/lianzimeng/working/kaleido/widgets/builder-widgets/vite.config.ts`
- `/Users/lianzimeng/working/kaleido/widgets/kaleido-widgets/vite.config.mjs`
- `/Users/lianzimeng/working/kaleido/widgets/kaleido-mobile-widgets/vite.config.mjs`

#### **库级 Vite 配置 (packages/*/vite.config.ts) 主要配置项**:

```typescript
// 通用库级配置模板（bpd-editor/common-components等）
defineConfig({
  cacheDir: '../../node_modules/.vite/[package-name]',
  
  plugins: [
    dts({  // TypeScript 定义文件生成
      entryRoot: 'components',
      tsConfigFilePath: joinPathFragments(__dirname, 'tsconfig.json'),
      skipDiagnostics: true,
    }),
    react(),
    viteTsConfigPaths({ root: './' }),  // tsconfig paths 别名支持
  ],

  build: {
    lib: {
      entry: 'components/index.ts',
      name: 'Kaleido[PackageName]',
      fileName: 'index',
      formats: ['es', 'cjs', 'umd'],  // 多格式输出
    },
    rollupOptions: {
      external: ['react', 'react-dom', 'react/jsx-runtime'],  // 外部化依赖
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM',
          'react/jsx-runtime': 'jsxRuntime',
        },
      },
    },
    sourcemap: true,
  },

  test: {  // Vitest配置
    globals: true,
    cache: { dir: '../../node_modules/.vitest' },
    environment: 'jsdom',
    include: ['src/**/*.{test,spec}.{js,mjs,cjs,ts,mts,cts,jsx,tsx}'],
  },
})
```

#### **应用级 Vite 配置 (websites/*/vite.config.ts) 主要配置项**:

以 workspace 应用为例，含复杂功能:

```typescript
defineConfig(async ({ mode }) => {
  // 1. 环境加载
  const VITE_ENV = loadEnv(mode, envDir);
  
  // 2. 插件配置
  plugins: [
    // HTML转换（处理版本分发脚本）
    { name: 'html-transform', transformIndexHtml() { ... } },
    
    svgr(),  // SVG作为React组件
    autoImport('vite'),  // 自动导入
    react(),
    
    cdnProtector({  // CDN保护
      projectNames: ['kaleido-extranet', 'KS-EFF', 'KS-IS-AVATAR'],
      domains: ['h*.static.yximgs.com', '*.kwimgs.com'],
      dynamicImportRetryEnable: true,
      ajaxRetryEnable: true,
      radarSeedEnable: true,
    }),
    
    prismjs({ languages: ['clike'] }),  // 代码高亮
  ],

  // 3. CSS预处理
  css: {
    preprocessorOptions: {
      less: {
        javascriptEnabled: true,
        modifyVars: TOFT_MODIFY_VARS_LIST,  // TOFT主题定制
      },
    },
  },

  // 4. 路径别名
  resolve: { alias },

  // 5. 代理配置
  server: {
    middlewareMode: false,
    proxy: {
      '/java/api/workspace': { target: VITE_ENV.VITE_WORKSPACE_PROXY },
      '/node/api/workspace': { target: VITE_ENV.VITE_BUILDER_SERVER_PROXY },
      '/magic-ai': { target: VITE_ENV.VITE_WORKSPACE_PROXY, ws: true },
      '/lsp': { target: VITE_ENV.VITE_WORKSPACE_PROXY, ws: true },
      // ... 更多代理
    },
    port: VITE_ENV.VITE_PORT_WORKSPACE,
    host: '0.0.0.0',
    https: { cert: '...', key: '...' },
  },

  // 6. 构建配置
  build: {
    chunkSizeWarningLimit: 30000,
    cssCodeSplit: false,
    minify: false,
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          if (id.includes('node_modules')) return 'vendor';
        },
      },
    },
  },

  // 7. 环境变量注入
  define: {
    'process.env': {
      NODE_ENV, DB_ENV, K_ENV, PUBLIC_URL,
      platform, workspace, runtime, 'sys-admin',
      DEPLOY_ENV, PLATFORM_COMMIT_ID, PLATFORM_VERSION,
    },
  },

  base: process.env.PUBLIC_URL,
})
```

#### **Widget 配置 (kaleido-widgets/vite.config.mjs)**:

```javascript
// 多入口构建 UMD 格式
const Entries = [
  'src/builder.ts',
  'src/render.ts',
  'src/meta.ts',
  'src/componentMeta.ts',
  'src/utils.ts',
];

Entries.forEach((entry) => {
  build({
    configFile: false,
    ...defaultConfig(entry),  // 使用 vite build() 直接构建
  });
});

// 配置中：
build: {
  lib: {
    entry,
    name: `${pkgId}@${name}@${version}`.replace(/\./g, '_'),
    formats: ['umd'],  // 只输出 UMD
  },
  rollupOptions: {
    external: [
      'react', 'react-dom', 'react/jsx-runtime',
      '@toft/react', '@toft/react-pro',
      '@kaleido/common-components',
      '@kaleido/widget-core',
    ],
    output: {
      globals: { /* ... */ },
      entryFileNames: `${name}.js`,
    },
  },
  minify: 'esbuild',
  sourcemap: 'hidden',
}
```

---

### 4. TypeScript 配置

#### **根 tsconfig.json** (`/Users/lianzimeng/working/kaleido/tsconfig.json`):
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "lib": ["es6", "dom"],
    "strict": true,
    "declaration": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "moduleResolution": "nodenext",
    "jsx": "react",
    "baseUrl": "./",
    "paths": {
      "@@/*": [".dumi/tmp/*"],
      "@kaleido/builder-server/*": ["./servers/builder-server/*"],
      "@kaleido/page-editor/*": ["./packages/page-editor/*"],
      "@kaleido/widget-core/*": ["./packages/widget-core/*"],
      "@kaleido/shared/*": ["./packages/shared/*"]
    }
  }
}
```

#### **Vite 专用 tsconfig** (`tsconfig.vite.json`):
```json
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "module": "ESNext",
    "moduleResolution": "Node",
    "jsx": "react-jsx",
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true
  }
}
```

#### **Dumi 专用 tsconfig** (`tsconfig.dumi.json`):
```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "NodeNext",
    "jsx": "react-jsx",
    "strict": true,
    "declaration": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "moduleResolution": "NodeNext"
  }
}
```

#### **包级 tsconfig 示例** (packages/bpd-editor/tsconfig.json):
```json
{
  "extends": "../../tsconfig.dumi.json",
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@@/*": [".dumi/tmp/*"],
      "@kaleido/bpd-editor": ["components"],
      "@kaleido/bpd-editor/*": ["components/*", "*"]
    }
  }
}
```

---

### 5. 包之间的依赖关系

#### **直接内部依赖** (workspace:* 协议):

**@kaleido/page-editor** 依赖:
- @kaleido/shared
- @kaleido/widget-core
- @kaleido/compiler

**@kaleido/flow-editor** 依赖:
- @kaleido/shared
- @kaleido/flow-nodes
- @kaleido/compiler

**@kaleido/common-components** 依赖:
- @kaleido/shared

**@kaleido/flow-nodes** 依赖:
- @kaleido/shared

**@kaleido/widget-core** 依赖:
- 无内部依赖（基础库）

**@kaleido/shared** 依赖:
- @toft/react (外部)

#### **外部关键依赖**:
- React 17.0.2
- Antd 5.11.3
- @toft/react 系列 (快手自有UI库)
- Vite 4.4.7
- NX 17.0.3
- TypeScript 5.3.3

#### **构建工具链**:
- vite-plugin-dts - 类型定义生成
- @vitejs/plugin-react - React支持
- vite-tsconfig-paths - 路径别名
- vite-plugin-svgr - SVG组件化
- rollup-plugin-visualizer - 构建分析
- @nx/vite - NX与Vite集成

---

### 6. 关键配置特性总结

| 特性 | 说明 |
|------|------|
| **包管理** | pnpm workspace，支持workspace协议依赖 |
| **任务执行** | NX + pnpm，支持缓存、并行执行 |
| **多包输出** | 库级: es/cjs/umd；应用级: esm；widget: umd |
| **类型支持** | 自动生成.d.ts，支持多个tsconfig继承 |
| **开发代理** | 支持多个后端服务的代理配置 |
| **主题定制** | LESS变量定制（通过modifyVars） |
| **自动导入** | 支持unplugin-auto-import自动导入 |
| **版本分发** | 动态版本脚本注入到index.html |
| **CDN保护** | @ks-cdn/cdn-web-protector-vite集成 |
| **构建分析** | rollup-plugin-visualizer支持 |

---

### 7. 项目版本信息

- **项目名称**: kaleido-fe
- **项目版本**: 1.0.25
- **Node版本要求**: 16.18+ (通过 @types/node)
- **TypeScript**: 5.3.3+
- **Vite**: 4.4.7+
- **NX**: 17.0.3
