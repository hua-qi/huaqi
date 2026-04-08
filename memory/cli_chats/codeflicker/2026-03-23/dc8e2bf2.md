---
session_id: dc8e2bf2
date: 2026-03-23
time_start: 2026-03-23T09:30:40.036Z
time_end: 2026-03-23T09:35:51.480Z
project: users-lianzimeng
git_branch: 
---

[2026-03-23 17:30:40] [user]: 2.5 前端工程师跳槽的八股文
[2026-03-23 17:31:17] [assistant]: 以下是 2.5 年前端工程师跳槽常考八股文：

---

## 一、JavaScript 核心

- **原型链** / 继承（class、寄生组合继承）
- **闭包** 原理与使用场景
- **事件循环**（Event Loop）：宏任务/微任务执行顺序
- **Promise** 原理、手写 Promise.all / race / allSettled
- **async/await** 底层（Generator + 自动执行器）
- **this** 指向（call/apply/bind 手写）
- **深拷贝** 手写（考虑循环引用）
- **防抖/节流** 手写
- **作用域链**、变量提升、暂时性死区
- **WeakMap/WeakSet** vs Map/Set

---

## 二、CSS

- **BFC** 触发条件与应用
- **Flex / Grid** 布局核心属性
- **层叠上下文**（z-index 失效原因）
- **CSS 动画性能**（transform vs top/left，合成层）
- **响应式**（媒体查询、rem/vw/vh）
- **选择器优先级**计算

---

## 三、浏览器 / 网络

- **浏览器渲染流程**（Parse HTML → CSSOM → Layout → Paint → Composite）
- **重排(reflow) vs 重绘(repaint)**
- **从输入 URL 到页面展示**全流程
- **HTTP 1.1 / 2 / 3** 区别
- **HTTPS** 握手过程
- **跨域** CORS / JSONP / 反向代理
- **缓存**（强缓存 Cache-Control / 协商缓存 ETag、Last-Modified）
- **Cookie / localStorage / sessionStorage / IndexedDB** 区别
- **XSS / CSRF** 攻击与防御

---

## 四、Vue（2.5年经验重点）

- **响应式原理**：Vue2 defineProperty vs Vue3 Proxy
- **虚拟 DOM & diff 算法**（key 的作用）
- **nextTick** 原理
- **computed vs watch** 区别
- **组件通信**方式（8种）
- **keep-alive** 原理
- **Vue3 Composition API** vs Options API
- **Pinia vs Vuex** 对比
- **生命周期**对应关系（setup 替代 created/beforeCreate）

---

## 五、React（如有使用）

- **Hooks** 原理（useState/useEffect/useCallback/useMemo）
- **Fiber 架构** & 调度机制
- **受控 vs 非受控**组件
- **合成事件**
- **diff 算法**（同层比较、key 规则）

---

## 六、工程化

- **Webpack** 核心概念（loader / plugin 区别、Tree Shaking、code splitting）
- **Vite** 为什么快（ESM + esbuild）
- **Babel** 工作原理（AST）
- **npm / pnpm** 依赖管理差异
- **CI/CD** 基本流程

---

## 七、性能优化

- **首屏优化**：懒加载、预加载、SSR、骨架屏
- **图片优化**：WebP、懒加载、雪碧图
- **Bundle 优化**：分包、externals、gzip
- **运行时优化**：虚拟列表、requestAnimationFrame

---

## 八、算法（高频手写）

| 题目 | 考察点 |
|------|--------|
| 数组去重 | Set / 哈希 |
| 扁平化 / 树结构互转 | 递归 |
| 手写 new / instanceof | 原型 |
| LRU 缓存 | Map + 双向链表 |
| 二叉树遍历 | DFS/BFS |
| 排序（快排、归并） | 分治 |

---

## 九、项目亮点准备

重点准备 **STAR 法则**（情况→任务→行动→结果）：
- 你做过最难的性能优化？指标改善多少？
- 如何封装通用组件/工具库？
- 遇到过最复杂的 Bug 怎么排查？

---

**2.5年经验定位**：偏重原理理解 + 工程实践，纯背概念不够，需要能延伸讲清楚为什么。
[2026-03-23 17:33:14] [user]: 将上述八股文生成 docs 文档
[2026-03-23 17:35:51] [assistant]: 文档已生成：https://docs.corp.kuaishou.com/d/home/fcACCPn08nY9tfUHqKTNZr6Cg
