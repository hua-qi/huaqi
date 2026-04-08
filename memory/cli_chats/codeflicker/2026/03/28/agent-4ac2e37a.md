---
session_id: agent-4ac2e37a
date: 2026-03-28
time_start: 2026-03-28T21:49:52.114Z
time_end: 2026-03-28T21:51:21.875Z
project: users-lianzimeng-working-kd-toft-chat-bot
git_branch: feat-multi-modal
---

[2026-03-29 05:49:52] [user]: 在以下两个项目中找出所有与"模块联邦本地联调"相关的代码：
1. /Users/lianzimeng/working/kd.toft.chat-bot/
2. /Users/lianzimeng/working/chat-bot-cards/

搜索关键词：localCardLibraryEntry、localCards、enableDebug、本地联调、本地卡片库、getPublicPath、localCardLibrary 等。

请列出所有相关文件路径和具体代码行，包括：
- 相关变量定义
- 条件判断逻辑
- console.log 调试信息
- 注释说明

只需返回调研结果，不需要写代码。
[2026-03-29 05:50:12] [assistant]: 我将在这两个项目中搜索所有与"模块联邦本地联调"相关的代码。让我逐个项目进行系统搜索。
[2026-03-29 05:50:15] [assistant]: 现在开始在两个项目中搜索关键词。
[2026-03-29 05:50:20] [assistant]: 很好！我在第一个项目中找到了相关代码。现在让我获取完整的文件内容以提供更详细的信息。
[2026-03-29 05:50:24] [assistant]: 现在让我检查 chat-bot-cards 项目中的配置文件：
[2026-03-29 05:50:27] [assistant]: 现在让我读取完整的卡片库配置文件：
[2026-03-29 05:50:30] [assistant]: 现在让我进行更详细的搜索，查找可能遗漏的相关配置：
[2026-03-29 05:50:34] [assistant]: 现在让我进一步搜索 Debug 相关文件：
[2026-03-29 05:50:37] [assistant]: 最后让我检查 Debug 索引文件：
[2026-03-29 05:50:40] [assistant]: 现在让我尝试找到 Debug 模块的入口文件：
[2026-03-29 05:50:46] [assistant]: 完美！现在让我创建一个完整的总结报告。让我验证一下是否还有其他相关文件：
[2026-03-29 05:50:51] [assistant]: 完美！现在我已经收集到了所有相关信息。让我生成完整的报告。
[2026-03-29 05:51:21] [assistant]: ---

## 调研报告：模块联邦本地联调代码汇总

### 一、概况统计

**第一个项目** (kd.toft.chat-bot - 主应用): 找到 **7 个相关文件**  
**第二个项目** (chat-bot-cards - 卡片库): 找到 **3 个相关文件**  
**总计**: **10 个文件**，涵盖配置、路由、钩子函数、调试模块和文档

---

## 二、项目一：kd.toft.chat-bot（主应用）

### 1. **src/automation/const.ts** - URL 参数解析
**文件位置**: `/Users/lianzimeng/working/kd.toft.chat-bot/src/automation/const.ts`

```typescript
// 第 10 行：获取 URL 查询参数
const searchParams = new URLSearchParams(location.search);

// 第 12-13 行：定义调试开关
export const enableAutoSend = searchParams.get('enableAutoSend') === '1';
export const enableDebug = searchParams.get('enableDebug') === '1';

// 第 15-16 行：本地联调配置（关键变量）
// 本地联调配置：指向本地卡片库服务
export const localCardLibraryEntry = searchParams.get('localCards') || null;
```

**作用**: 从 URL 参数中提取 `localCards` 参数，用于指定本地卡片库的 remoteEntry.js 地址

---

### 2. **src/common-components/hooks/useDynamicRegisterCards.ts** - 核心联调逻辑
**文件位置**: `/Users/lianzimeng/working/kd.toft.chat-bot/src/common-components/hooks/useDynamicRegisterCards.ts`

#### 导入部分（第 8 行）：
```typescript
import { enableDebug, localCardLibraryEntry } from '@src/automation/const';
```

#### 条件判断 1 - 本地卡片库替换（第 16-32 行）：
```typescript
// 本地联调：如果 URL 中有 localCards 参数，指向本地卡片库服务
console.log('[useDynamicRegisterCards] localCardLibraryEntry:', localCardLibraryEntry);
if (localCardLibraryEntry) {
  console.log('[useDynamicRegisterCards] 启用本地卡片库:', localCardLibraryEntry);
  return result.map((r) => {
    if (r?.name === 'ihr_widget_provider') {
      console.log('[useDynamicRegisterCards] 替换 entry:', r.entry, '->', localCardLibraryEntry);
      return {
        ...r,
        entry: localCardLibraryEntry  // 关键：替换为本地 remoteEntry 地址
      };
    }
    return r;
  });
}
```

#### 条件判断 2 - 调试模式判断（第 39 行）：
```typescript
if (enableDebug) return;
```
**作用**: 在 Debug 模式下直接返回，不执行下面的初始化逻辑

#### 条件判断 3 - 本地联调时清除模块联邦缓存（第 42-49 行）：
```typescript
// 本地联调时：强制清除模块联邦缓存，确保加载最新版本
if (localCardLibraryEntry) {
  console.log('[useDynamicRegisterCards] 本地联调模式，清除模块联邦缓存');
  // @ts-ignore
  if (window?.__FEDERATION__?.__SHARE__?.loadedRemote?.ihr_widget_provider) {
    // @ts-ignore
    delete window.__FEDERATION__.__SHARE__.loadedRemote.ihr_widget_provider;
  }
}
```
**作用**: 强制删除已加载的模块，确保重新加载最新版本

#### 条件判断 4 - 配置 getPublicPath（第 51-60 行）：
```typescript
// 为本地联调配置 remotes
const remotes = data.map((r: any) => {
  // 如果是本地卡片库，添加 getPublicPath 确保子 chunk 从正确地址加载
  if (r?.name === 'ihr_widget_provider' && localCardLibraryEntry) {
    return {
      ...r,
      getPublicPath: () => 'http://localhost:3001/'  // 硬编码本地服务地址
    };
  }
  return r;
});
```
**作用**: 为本地卡片库设置公共路径，确保异步 chunk 从正确地址加载

---

### 3. **rsbuild.config.ts** - Webpack 代理配置
**文件位置**: `/Users/lianzimeng/working/kd.toft.chat-bot/rsbuild.config.ts`

#### 代理配置（第 128-132 行）：
```typescript
// Module Federation 本地联调：代理卡片库的所有 chunk 请求
'/static/js/async': {
  target: 'http://localhost:3001',  // 代理到本地卡片库服务
  changeOrigin: true
}
```
**作用**: 将异步 chunk 请求代理到本地卡片库服务，避免跨域问题

#### 服务器配置（第 67-68 行）：
```typescript
server: {
  port: 3000,  // 主应用运行在 3000 端口
```

---

### 4. **src/automation/index.ts** - 条件导入模块
**文件位置**: `/Users/lianzimeng/working/kd.toft.chat-bot/src/automation/index.ts`

```typescript
import { enableAutoSend, enableDebug } from './const';

if (enableAutoSend) {
  import('./AutoSend/index');
}

if (enableDebug) {
  import('./Debug/index');  // 仅在 enableDebug=1 时加载 Debug 模块
}
```
**作用**: 条件加载 Debug 模块

---

### 5. **src/automation/Debug/index.tsx** - Debug UI 模态框
**文件位置**: `/Users/lianzimeng/working/kd.toft.chat-bot/src/automation/Debug/index.tsx`

```typescript
function Debug() {
  const [name, setName] = useState('');
  const [url, setUrl] = useState('');
  const [visible, onOpen, onClose] = useVisible(true);
  const { dispatchRegisterCard } = useDispatchRegisterCard();

  function handleSend() {
    dispatchRegisterCard({
      name,
      entry: url.replace(/\/$/, '') + '/remoteEntry.js'  // 构建 remoteEntry 地址
    });
    onClose();
  }
  // ... UI 渲染
}
```
**作用**: 提供 UI 界面，手动输入卡片库的 name 和 url，用于实时注册调试

---

### 6. **src/automation/Debug/hooks/useAddRegisterCardsEventListener.ts**
**文件位置**: `/Users/lianzimeng/working/kd.toft.chat-bot/src/automation/Debug/hooks/useAddRegisterCardsEventListener.ts`

```typescript
useEffect(() => {
  if (!enableDebug) return;  // 仅在调试模式下执行
  eventEmitter.on(AutomationEventEnum.SendRegisterCard, (cardData) => {
    latestSendRegisterCardFn.current(cardData);
  });
  return () => {
    eventEmitter.off(AutomationEventEnum.SendRegisterCard);
  };
}, [latestSendRegisterCardFn]);
```
**作用**: 监听卡片注册事件，仅在调试模式下激活

---

### 7. **dev-linkup.sh** 和 **dev-linkup.md** - 启动脚本和文档
**文件位置**: 
- `/Users/lianzimeng/working/kd.toft.chat-bot/dev-linkup.sh`
- `/Users/lianzimeng/working/kd.toft.chat-bot/dev-linkup.md`

#### Shell 脚本关键信息（第 72-79 行）：
```bash
echo "  本地联调:  http://localhost:3000/hr?localCards=http://localhost:3001/remoteEntry.js"
echo "  卡片库:    http://localhost:3001"
echo "使用说明:"
echo "  1. 打开本地联调地址"
echo "  2. 修改卡片库代码后自动热更新"
echo "  3. 刷新页面即可看到更新"
```

#### Markdown 文档关键信息：
```markdown
## 3. 使用本地卡片库

浏览器访问：
http://localhost:3000/hr?localCards=http://localhost:3001/remoteEntry.js

参数说明：
- localCards - 本地卡片库 remoteEntry.js 地址

## 配置变更记录

### 2. kd.toft.chat-bot（主应用）
- **文件**: src/automation/const.ts
- **变更**: 新增 localCardLibraryEntry 配置，从 URL 参数读取本地卡片库地址

- **文件**: src/common-components/hooks/useDynamicRegisterCards.ts
- **变更**: 当 localCards 参数存在时，将 ihr_widget_provider 指向本地服务
```

---

## 三、项目二：chat-bot-cards（卡片库）

### 1. **ai-card.config.mjs** - 卡片库配置文件
**文件位置**: `/Users/lianzimeng/working/chat-bot-cards/ai-card.config.mjs`

```typescript
export default defineConfig({
    name: "ihr_widget_provider",           // 模块名称（与主应用相同）
    filename: "remoteEntry.js",             // Module Federation 入口文件名
    version: "0.0.54-beta.5",
    // ... exposes 配置（列出所有暴露的卡片组件）
    server: {
        port: 3001,  // 卡片库运行在 3001 端口（与主应用不同，避免冲突）
        proxy: {
            "/api/runtime/application/": {
                target: "https://kaleido.staging.kuaishou.com",
                changeOrigin: true,
                // ... 代理配置
            }
        }
    }
});
```

---

### 2. **src/index.tsx** - 公共路径设置
**文件位置**: `/Users/lianzimeng/working/chat-bot-cards/src/index.tsx`

```typescript
// 设置 Module Federation chunk 加载路径（必须在任何导入之前）
// @ts-ignore
__webpack_public_path__ = 'http://localhost:3001/';  // 关键：本地卡片库的公共路径

import('./bootstrap');
```

**作用**: 在模块加载前设置公共路径，确保所有 chunk 从 localhost:3001 加载

---

### 3. **src/bootstrap.tsx** - Bootstrap 入口
**文件位置**: `/Users/lianzimeng/working/chat-bot-cards/src/bootstrap.tsx`

```typescript
// 设置 Module Federation chunk 加载路径（必须在任何导入之前）
// @ts-ignore
__webpack_public_path__ = 'http://localhost:3001/';  // 重复设置，保证生效

import { render } from 'react-dom';
import App from './App';

const container = document.getElementById('root');
render(<App />, container);
```

**作用**: 双重设置公共路径，确保可靠性

---

## 四、关键技术点总结

| 关键点 | 位置 | 说明 |
|------|------|------|
| **URL 参数** | `src/automation/const.ts` | `?localCards=http://localhost:3001/remoteEntry.js` |
| **本地卡片库地址** | `src/automation/const.ts` L16 | `localCardLibraryEntry` 变量存储 |
| **Entry 替换** | `useDynamicRegisterCards.ts` L18-26 | 将 `ihr_widget_provider` 的 entry 替换为本地地址 |
| **缓存清除** | `useDynamicRegisterCards.ts` L42-48 | 删除 `window.__FEDERATION__.__SHARE__.loadedRemote` |
| **getPublicPath** | `useDynamicRegisterCards.ts` L54-58 | 设置为 `http://localhost:3001/` |
| **Webpack 代理** | `rsbuild.config.ts` L130-132 | 代理 `/static/js/async` 到本地服务 |
| **Debug 模式** | `src/automation/index.ts` L7-9 | `enableDebug=1` 时加载 Debug UI |
| **Debug UI** | `Debug/index.tsx` | 手动输入卡片库信息并注册 |
| **卡片库端口** | `ai-card.config.mjs` L45 | 固定为 3001 |
| **公共路径** | `src/index.tsx` & `bootstrap.tsx` L3 | 硬编码 `http://localhost:3001/` |

---

## 五、使用流程

1. **启动卡片库**: `cd chat-bot-cards && pnpm start-card` → 运行在 `http://localhost:3001`
2. **启动主应用**: `cd kd.toft.chat-bot && pnpm dev` → 运行在 `http://localhost:3000`
3. **访问本地联调 URL**: `http://localhost:3000/hr?localCards=http://localhost:3001/remoteEntry.js`
4. **主应用逻辑**:
   - 解析 URL 参数，获取 `localCards` 值
   - 替换 `ihr_widget_provider` 的 entry 为本地地址
   - 清除模块联邦缓存
   - 设置 `getPublicPath` 为 `http://localhost:3001/`
   - 初始化 Module Federation
5. **Webpack 代理**: `/static/js/async` → `http://localhost:3001`
6. **可选调试模式**: 添加 `?enableDebug=1` 参数，启用 Debug UI 进行手动卡片注册

---

## 六、所有相关文件清单

### 项目一：kd.toft.chat-bot
- `/Users/lianzimeng/working/kd.toft.chat-bot/src/automation/const.ts`
- `/Users/lianzimeng/working/kd.toft.chat-bot/src/common-components/hooks/useDynamicRegisterCards.ts`
- `/Users/lianzimeng/working/kd.toft.chat-bot/rsbuild.config.ts`
- `/Users/lianzimeng/working/kd.toft.chat-bot/src/automation/index.ts`
- `/Users/lianzimeng/working/kd.toft.chat-bot/src/automation/Debug/index.tsx`
- `/Users/lianzimeng/working/kd.toft.chat-bot/src/automation/Debug/hooks/useAddRegisterCardsEventListener.ts`
- `/Users/lianzimeng/working/kd.toft.chat-bot/dev-linkup.sh`
- `/Users/lianzimeng/working/kd.toft.chat-bot/dev-linkup.md`

### 项目二：chat-bot-cards
- `/Users/lianzimeng/working/chat-bot-cards/ai-card.config.mjs`
- `/Users/lianzimeng/working/chat-bot-cards/src/index.tsx`
- `/Users/lianzimeng/working/chat-bot-cards/src/bootstrap.tsx`
