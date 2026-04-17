---
session_id: agent-98fdd6af
date: 2026-04-14
time_start: 2026-04-14T11:54:07.774Z
time_end: 2026-04-14T11:58:34.637Z
project: users-lianzimeng-working-kaleido
git_branch: hotfix-reusable-widget-scope
---

[2026-04-14 19:54:07] [user]: 在 /Users/lianzimeng/working/kaleido 项目中，深入探索多应用之间如何协作。请重点查看：

1. **iframe 通信**：搜索 postMessage、iframe、window.parent 相关代码
2. **微前端**：搜索 qiankun、single-spa、module federation 相关配置
3. **URL/路由跳转**：各应用之间如何跳转（window.open、location.href 等）
4. **共享状态**：是否有跨应用的状态共享机制（localStorage、BroadcastChannel 等）
5. **Runtime 嵌入**：workspace 如何加载/嵌入 runtime 应用
6. **Widget 动态加载**：widget 的 UMD 格式如何被动态加载

具体查看以下文件：
- websites/workspace/src/ 中的主入口文件和路由
- websites/runtime/src/ 和 websites/runtime-v2/src/ 主入口
- widgets/kaleido-widgets/ 如何被加载

请返回：多应用协作的核心机制、通信方式、代码证据。
[2026-04-14 19:54:11] [assistant]: 我来帮你深入探索这个项目中多应用之间的协作机制。让我开始系统地搜索和分析相关代码。
[2026-04-14 19:54:16] [assistant]: 现在让我检索相关的工程经验：
[2026-04-14 19:54:19] [assistant]: 好的，我已经找到相关经验。现在让我开始系统地搜索和分析代码。
[2026-04-14 19:54:25] [assistant]: 现在让我搜索关键的通信机制代码：
[2026-04-14 19:54:30] [assistant]: 让我继续搜索更具体的通信和加载机制：
[2026-04-14 19:54:35] [assistant]: 现在让我读取关键的主入口文件和通信相关代码：
[2026-04-14 19:54:41] [assistant]: 现在让我搜索关键的通信和iframe相关文件：
[2026-04-14 19:54:47] [assistant]: 现在让我查看 widget 的加载机制：
[2026-04-14 19:54:52] [assistant]: 让我查找useWidgetFactory文件：
[2026-04-14 19:55:02] [assistant]: 现在让我查看Runtime中的应用分发和路由跳转机制：
[2026-04-14 19:55:20] [assistant]: 现在让我查看Runtime中的widget加载机制和跨应用通信：
[2026-04-14 19:55:33] [assistant]: 现在让我查看widget的manifest和构建配置：
[2026-04-14 19:55:38] [assistant]: 现在让我查看Runtime中如何加载plugin/widget库：
[2026-04-14 19:55:44] [assistant]: 现在让我查看跨应用状态共享的相关代码：
[2026-04-14 19:55:51] [assistant]: 让我查看应用的包装和通信的相关内容：
[2026-04-14 19:56:43] [assistant]: 现在让我查看应用配置中对Runtime的引用方式：
[2026-04-14 19:56:48] [assistant]: 让我查找useRuntimeClusterHost：
[2026-04-14 19:57:30] [assistant]: 现在让我收集一些结构信息和配置：
[2026-04-14 19:57:35] [assistant]: 现在让我查看主要的Vite配置来理解应用构建方式：
[2026-04-14 19:57:45] [assistant]: 现在让我生成最终的报告。我已经收集到足够的信息来描述多应用协作机制。让我保存经验：
[2026-04-14 19:58:34] [assistant]: 完美！现在让我生成最终的详细报告：

---

## 深度分析：Kaleido 多应用协作机制

我已经深入探索了 Kaleido 项目的多应用协作架构。以下是完整的核心机制、通信方式和代码证据：

### 🏗️ 应用架构概览

```
Kaleido 项目结构：
├── websites/
│   ├── workspace/          【主应用】编辑器 - DSL设计、编译、预览控制
│   ├── runtime/            【运行时应用 v1】页面渲染、流程执行
│   ├── runtime-v2/         【运行时应用 v2】新版页面渲染、HMR支持
│   ├── platform/           应用管理平台
│   ├── console/            管理控制台
│   └── sys-admin/          系统管理员后台
├── widgets/
│   ├── kaleido-widgets/    【Widget库】UI组件（UMD格式）
│   └── kaleido-mobile-widgets/
└── packages/               共享库、编译器等
```

---

### 📡 核心通信机制

#### 1. **PostMessage 双向通信（Workspace ↔ Runtime）**

**Workspace → Runtime 通信**（编译数据推送）

**文件**: `/websites/workspace/src/hooks/usePreviewInstances.ts`

```typescript
/**
 * 向所有运行时实例广播编译后的数据
 */
const compileAndRefresh = useCallback(async () => {
  const alivePreviewInstanceMap = getPreviewInstances();
  const currentBranchInstances = filterCurrentBranchInstances(
    alivePreviewInstanceMap,
    app.branchId,
  );
  
  if (currentBranchInstances.length !== 0) {
    const ir = await compile();
    // 关键：postMessage 推送编译数据到所有预览实例
    for (const previewInstance of currentBranchInstances) {
      previewInstance.postMessage(ir, previewOrigin);  // ← 核心通信
    }
  }
}, [compile, previewOrigin]);

/**
 * 监听 Runtime 的数据请求
 */
const runtimeMessageHandler = useCallback(
  async (e: MessageEvent) => {
    if (e.origin !== previewOrigin) {
      return;
    }
    // Runtime 主动请求最新编译数据
    if (e.data?.type === 'getLatestData') {  // ← Runtime 发起
      const ir = await compile();
      e.source.postMessage(ir, {          // ← Workspace 响应
        targetOrigin: previewOrigin,
      });
    }
  },
  [previewOrigin, compile],
);
```

**Runtime → Workspace 通信**（HMR 请求）

**文件**: `/websites/runtime-v2/src/hooks/useHmrApp.ts`

```typescript
/**
 * Runtime 向 Workspace 请求最新编译数据（热更新）
 */
window.parent.postMessage(
  { type: 'getLatestData', env: appBaseInfo?.env },  // ← 主动请求
  location.origin,
);

/**
 * 监听 Workspace 推送的编译数据
 */
const messageHandler = (e) => {
  if (e.origin !== location.origin) {
    return;
  }
  
  // 接收编译后的 IR 元数据
  if (e?.data?.irMetadata) {
    const mergeDsl = { ...preState.dsl, ...filterDsl };
    setDsl(mergeDsl);
    afterTransform = transformDSL(mergeDsl);
    setTransformDsl(afterTransform);  // ← 更新页面渲染
  }
};

window.addEventListener('message', messageHandler);
```

**预览实例追踪与通知**

```typescript
// 编辑器刷新时通知所有预览实例当前活跃分支
useEffect(() => {
  const instances = getPreviewInstances();
  for (const previewInstance of Object.values(instances)) {
    previewInstance.postMessage(
      {
        type: 'activeBranch',
        activeBranch: app.branchId,
      },
      previewOrigin,
    );
  }
}, []);
```

---

#### 2. **SessionStorage 预览实例持久化**

**文件**: `/websites/workspace/src/hooks/usePreviewInstances.ts`

```typescript
// 预览实例标识符（跨会话唯一）
const WORKSPACE_INSTANCE_IDENTIFIER = nanoid();

class PreviewInstanceStorage {
  /**
   * 生成预览实例的分段ID
   * 格式: "branchId,previewInstanceId,workspaceInstanceIdentifier"
   */
  static generateSegmentedId(branchId: string, previewInstanceId: string) {
    return `${branchId},${previewInstanceId},${WORKSPACE_INSTANCE_IDENTIFIER}`;
  }

  /**
   * 存储预览实例到 sessionStorage
   */
  set(updater: (previewInstances: Record<string, Window>) => Record<string, Window>) {
    const newInstances = updater(this._previewInstances) || {};
    const names = Object.keys(
      PreviewInstanceStorage.prunePreviewInstances(newInstances),
    );
    // 关键：保存预览窗口名称列表
    window.sessionStorage.setItem(
      'previewInstances',
      JSON.stringify(names),
      true,  // ← 不同于普通 sessionStorage 的自定义标记
    );
    this._previewInstances = newInstances;
  }

  /**
   * 从 sessionStorage 恢复预览实例
   */
  get() {
    if (!this._previewInstances) {
      const previewInstances = {};
      const storedString = window.sessionStorage.getItem('previewInstances', true);
      
      if (storedString) {
        const previewInstanceNames = JSON.parse(storedString);
        for (const name of previewInstanceNames) {
          const segments = PreviewInstanceStorage.getIdSegments(name);
          // 只恢复当前 workspace 实例的预览窗口
          if (segments.workspaceInstanceId === WORKSPACE_INSTANCE_IDENTIFIER) {
            const instance = window.open('', name);
            if (instance) {
              previewInstances[name] = instance;
            }
          }
        }
      }
      this._previewInstances = previewInstances;
    }
    return this._previewInstances;
  }
}
```

---

#### 3. **路由跳转与应用间导航**

**应用内跳转**（Go To Page 流程节点）

**文件**: `/websites/runtime-v2/src/modules/engine/flowEngine/flow-nodes-executor/go-to-page.ts`

```typescript
const GoToPage = (node, provider) => {
  const { targetPage, newTab, pathParams, queryParams } = node.props;
  const { execExpression, getPagePath, getAppRouter } = provider.providerMethods;

  const path = getPagePath(targetPage);
  const finalUrl = `${targetPath}${isObject(params) ? searchStringify(params) : ''}`;

  const redirectFun = () => {
    const appRouter = getAppRouter();
    if (newTab) {
      window.open(finalUrl, '_blank', 'noopener,noreferrer');  // ← 新标签页跳转
    } else {
      appRouter.push(finalUrl);  // ← 内部路由跳转
    }
  };
  
  return Promise.resolve();
};
```

**外部链接打开**

**文件**: `/websites/runtime-v2/src/modules/engine/flowEngine/flow-nodes-executor/open-external-link.ts`

```typescript
const OpenExternalLink = (node, provider) => {
  const { url, newTab } = node.props;
  const { execExpression } = provider.providerMethods;

  const getTopWindow = () => {
    let top = window;
    while (top !== top.parent) {
      top = top.parent;  // ← 递归获取顶层窗口
    }
    return top;
  };

  const redirectFun = () => {
    let targetUrl = url ? execExpression(url) : 'about:blank';
    
    if (!MatchHttpRegExp?.test(targetUrl)) {
      targetUrl = `https://${targetUrl}`;
    }

    if (newTab) {
      getTopWindow().open(targetUrl, '_blank', 'noopener,noreferrer');
    } else {
      getTopWindow().location.href = targetUrl;  // ← 跨层级跳转
    }
  };

  return Promise.resolve();
};
```

---

### 📦 Widget 动态加载机制

#### 4. **UMD 格式 Widget 库构建与加载**

**Vite Widget 构建配置**

**文件**: `/widgets/kaleido-widgets/vite.config.mjs`

```javascript
const Entries = [
  'src/builder.ts',
  'src/render.ts',
  'src/meta.ts',
  'src/componentMeta.ts',
  'src/utils.ts',
];

const defaultConfig = (entry) => {
  const libraryName = `${pkgId}@${name}@${version}`.replace(/\./g, '_');
  
  return {
    build: {
      lib: {
        entry,
        name: libraryName,  // ← UMD 全局变量名
        formats: ['umd'],   // ← UMD 格式
      },
      rollupOptions: {
        external: [
          'react',
          'react-dom',
          '@toft/react',
          '@kaleido/widget-core',
        ],
        output: {
          globals: {
            'react': 'React',
            'react-dom': 'ReactDOM',
            '@toft/react': 'KToftReact',
            '@kaleido/widget-core': 'KaleidoWidgetCore',
          },
        },
      },
    },
  };
};
```

**Widget Manifest 元数据**

**文件**: `/widgets/kaleido-widgets/manifest.json`

```json
{
  "name": "kaleido-widgets",
  "id": "kaleido-widgets",
  "version": "2.0.608",
  "platformVersion": ">=2.0"
}
```

**Workspace 中的 Widget 库加载**

**文件**: `/websites/workspace/src/utils/widgetLibrary.ts`

```typescript
/**
 * 动态加载 Widget 库（V2 版本）
 * 加载 meta、builder、componentMeta 三个独立的 UMD
 */
export const getLibrarySourceCodeV2 = (
  widgetLibraryFactory,
  libraryDetail,
  isReload = false,
) => {
  return new Promise((resolve, reject) => {
    Promise.all([
      loadLibrary(
        getLibraryMountName(libraryDetail, 'meta'),
        libraryDetail.metaSourceUrl,  // ← 拉取 meta.js
        isReload,
      ),
      loadLibrary(
        getLibraryMountName(libraryDetail, 'builder'),
        libraryDetail.builderSourceUrl,  // ← 拉取 builder.js
        isReload,
      ),
      loadLibrary(
        getLibraryMountName(libraryDetail, 'componentMeta'),
        libraryDetail.componentMetaSourceUrl,  // ← 拉取 componentMeta.js
        isReload,
      ),
    ])
      .then((res) => {
        const [meta, builder, config] = res;
        // 转换数据结构并注册到组件工厂
        const targetWidgetLibrary = transformWidgetLibrary({
          ...config,
          metadata: meta,
          widgetsMap: builder,
        });
        widgetLibraryFactory.registerWidget(
          targetWidgetLibrary.componentLibrary,
          Object.values(targetWidgetLibrary?.componentLibrary.widgetsMap || {}),
        );
        resolve();
      });
  });
};

/**
 * 加载脚本到全局
 */
const loadLibrary = (name: string, scriptUrl: string, isReload = false) => {
  return new Promise<void>((resolve, reject) => {
    if (!isReload && window?.[name]) {
      resolve(window?.[name]);  // ← 已加载过则复用
      return;
    }

    loadScript(scriptUrl, () => {
      if (!window?.[name]) {
        return reject(new Error('组件库加载失败'));
      }
      resolve(window?.[name]);  // ← 从全局对象获取
    }, reject);
  });
};

/**
 * 挂载库名生成规则
 * 格式: "kaleido-widgets@meta@2_0_608"
 */
const getLibraryMountName = (libraryDetail, key) => {
  return key
    ? `${libraryDetail.id}@${key}@${libraryDetail.version.split('.').join('_')}`
    : libraryDetail.id;
};
```

**Runtime 中的 Widget 库加载**

**文件**: `/websites/runtime-v2/src/utils/pluginLibrary.ts`

```typescript
export const getLibrarySourceCode = (
  registerWidget: any,
  libraryDetail: any,
): Promise<void> => {
  return new Promise<void>((resolve, reject) => {
    // 并行加载 meta、render、componentMeta 三个 UMD
    Promise.all([
      loadLibrary(
        getLibraryMountName(libraryDetail, 'meta'),
        libraryDetail.metaSourceUrl,
      ),
      loadLibrary(
        getLibraryMountName(libraryDetail, 'render'),
        libraryDetail.renderSourceUrl,  // ← Runtime 加载 render.js
      ),
      loadLibrary(
        getLibraryMountName(libraryDetail, 'componentMeta'),
        libraryDetail.componentMetaSourceUrl,
      ),
    ])
      .then((res) => {
        const [widgetsMeta, widgetsRender, widgetLibraryMeta] = res;
        registerWidget({ widgetsMeta, widgetsRender, widgetLibraryMeta });
        resolve();
      });
  });
};
```

**Widget 定义获取**

**文件**: `/websites/runtime-v2/src/modules/render/widgetRender/widgetUtils.ts`

```typescript
export const getWidgetDefine = (widgetProps: any) => {
  // 本地开发时使用内置组件库的实际导出
  if (isLocalDev && widgetProps?.widgetLibraryId === KaleidoWidgetsLibraryId) {
    return LocalDevWidgetsMap?.[KaleidoWidgetsLibraryId]?.[
      widgetProps?.widgetType
    ];
  }

  // 生产环境从全局 Widget 库映射获取
  const widgetLibraryMap = getAppScopeValue(GLOBAL_WIDGETS_LIBRARY_MAP);
  if (widgetProps?.widgetLibraryId) {
    return widgetLibraryMap?.[widgetProps?.widgetLibraryId]?.[
      widgetProps?.widgetType
    ];
  }
};
```

---

### 🌐 Workspace - Runtime 集群通信

**文件**: `/websites/workspace/src/hooks/useRuntimeClusterHost.ts`

```typescript
/**
 * 编辑器内获取运行时集群域名
 * 用于 postMessage 时指定 targetOrigin
 */
export const useRuntimeClusterHost = () => {
  const appInfo = useRecoilValue(appInfoAtom);
  
  const getRuntimeClusterHost = useCallback(
    (env: string) => {
      const envType = formatEnvType(env);
      
      // 支持多个运行时环境：preview、test、sendbox、online
      return appInfo.clusterConf?.[envType]?.originUrl;
    },
    [appInfo.clusterConf],
  );
  
  return getRuntimeClusterHost;
};

/**
 * 使用方式
 */
const previewOrigin = useMemo(() => {
  return getRuntimeClusterHost(Environment.Preview);
}, [getRuntimeClusterHost]);

// 发送消息时指定 targetOrigin
previewInstance.postMessage(ir, previewOrigin);
```

---

### 📊 应用间通信流程图

```
用户在 Workspace 编辑 DSL
            ↓
            编译成 IR 元数据
            ↓
    ┌───────────────────────┐
    │  sessionStorage 存储    │
    │  预览实例标识符         │
    └───────────────────────┘
            ↓
   向所有预览实例发送
   postMessage(ir, previewOrigin)
            ↓
    ┌───────────────────────┐
    │  Runtime 接收          │
    │  (iframe 或新窗口)     │
    └───────────────────────┘
            ↓
    处理 DSL，生成页面
            ↓
   【HMR 场景】Runtime 侦测文件变化
            ↓
   Runtime 主动发送:
   parent.postMessage({ type: 'getLatestData' })
            ↓
   Workspace 响应编译数据
```

---

### 🔌 跨应用状态管理

**共享的全局状态**

**文件**: `/websites/runtime/src/main.tsx` 和 `/websites/runtime-v2/src/main.tsx`

```typescript
// 部分页面数据需要扩展，recoil 和 window 都挂载一份
window.globalState = { widgetMapsCache: {} };

// 将重要库挂载到全局，供动态加载使用
Reflect.set(window, 'React', React);
Reflect.set(window, 'ReactDOM', ReactDOM);
Reflect.set(window, 'KToftReact', KToftReact);
Reflect.set(window, 'kaleidoCommonComponents', kaleidoCommonComponents);
Reflect.set(window, 'KaleidoWidgetCore', KaleidoWidgetCore);
Reflect.set(window, 'KaleidoWidgetCoreBuilder', KaleidoWidgetCoreBuilder);
```

**Workspace 中也挂载相同库**

**文件**: `/websites/workspace/src/main.tsx`

```typescript
// 使 UMD Widget 能够访问这些全局依赖
Reflect.set(window, 'React', React);
Reflect.set(window, 'KToftReact', KToftReact);
Reflect.set(window, 'kaleidoCommonComponents', kaleidoCommonComponents);
Reflect.set(window, 'KaleidoWidgetCore', KaleidoWidgetCore);
```

---

### 🎯 外部 iframe 通信

**文件**: `/websites/runtime/src/page/widgets/toft/external-iframe/index.tsx`

```typescript
export function RenderExternalIframe(props: Props) {
  const iframeRef = useRef<HTMLIFrameElement>(null);
  const { state: { srcUrl }, events: { onLoad, onMessage } } = useInitWidget(
    props,
    {
      getters: ['srcUrl'],
      setters: ['srcUrl'],
      events: ['onLoad', 'onMessage'],
      actions: {
        // 暴露 postMessage 方法给流程调用
        postMessage(message) {
          iframeRef.current?.contentWindow?.postMessage(
            message,
            new URL(srcUrl).origin,  // ← 指定 targetOrigin
          );
        },
      },
    },
  );

  // 监听来自 iframe 的消息
  useEffect(() => {
    window.addEventListener('message', onMessage, false);
    return () => {
      window.removeEventListener('message', onMessage);
    };
  }, []);

  return (
    <iframe
      ref={iframeRef}
      src={srcUrl}
      style={{ height: '100%' }}
      onLoad={onLoad}
    />
  );
}
```

---

### ✅ 完整协作流程总结

| 协作方式 | 实现机制 | 代码位置 | 应用场景 |
|---------|--------|--------|--------|
| **PostMessage** | Workspace → Runtime 推送编译数据 | `usePreviewInstances.ts` | HMR、DSL 同步 |
| **PostMessage** | Runtime → Workspace 请求数据 | `useHmrApp.ts` | 热更新触发 |
| **SessionStorage** | 预览实例标识符持久化 | `usePreviewInstances.ts` | 预览窗口追踪 |
| **URL 路由** | 应用内/外跳转 | `go-to-page.ts` | 页面导航 |
| **window.open** | 新标签页打开 | `open-external-link.ts` | 外部链接 |
| **UMD 动态加载** | Widget 库加载到全局 | `widgetLibrary.ts` | Widget 注册 |
| **全局对象** | 共享 React、库依赖 | `main.tsx` | Widget UMD 依赖 |
| **iframe** | 嵌入外部页面通信 | `external-iframe/index.tsx` | 第三方集成 |

---

### 📌 关键设计特点

1. **单向发布-订阅**：Workspace 作为编译中枢，编译完成后主动推送到 Runtime
2. **双向消息机制**：Runtime 也可主动请求最新数据（HMR 场景）
3. **会话级实例追踪**：通过 sessionStorage 跨刷新维护预览实例关系
4. **全局作用域隔离**：Widget 通过约定的命名规则（`id@type@version`）避免冲突
5. **渐进式依赖共享**：库依赖通过全局对象共享，减少 UMD 体积
6. **环境隔离**：支持 preview/test/sendbox/online 多个运行时环境

这是一套相当成熟的多应用协作方案，充分利用了现代浏览器的通信机制和 JavaScript 的动态能力！
