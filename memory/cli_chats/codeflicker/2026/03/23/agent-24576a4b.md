---
session_id: agent-24576a4b
date: 2026-03-23
time_start: 2026-03-23T03:41:49.247Z
time_end: 2026-03-23T03:42:29.910Z
project: users-lianzimeng-working-happy-scripts-toft-guard-demo
git_branch: 
---

[2026-03-23 11:41:49] [user]: 在 /Users/lianzimeng/working/happy/scripts/toft-guard-demo/node_modules/@toft/react 目录中，找到 Button 组件的类型定义和源码，列出 Button 组件支持的所有 props（包括 color、size、variant、disabled、loading、icon、shape 等），以及任何可能的子组件（如 ButtonGroup 等）。请返回完整的 props 列表和类型信息。
[2026-03-23 11:41:53] [assistant]: 我来帮你找到 Button 组件的类型定义和源码，以及所有支持的 props。让我先探索这个目录结构。
[2026-03-23 11:41:57] [assistant]: 现在让我读取 Button 组件的类型定义和源码文件：
[2026-03-23 11:42:01] [assistant]: 现在让我检查是否有任何 ButtonGroup 或其他子组件：
[2026-03-23 11:42:06] [assistant]: 让我查看主索引文件以检查是否有导出 ButtonGroup 或其他相关组件：
[2026-03-23 11:42:10] [assistant]: 让我再检查 Button 组件中是否有其他导出或组件定义：
[2026-03-23 11:42:29] [assistant]: 完美！现在让我创建一份完整的总结文档：

## 完整的 Button 组件分析报告

基于我对 `/Users/lianzimeng/working/happy/scripts/toft-guard-demo/node_modules/@toft/react` 目录的详细分析，以下是 Button 组件的完整文档：

### 文件位置
- **类型定义**: `/Users/lianzimeng/working/happy/scripts/toft-guard-demo/node_modules/@toft/react/es/button/Button.d.ts`
- **类型接口**: `/Users/lianzimeng/working/happy/scripts/toft-guard-demo/node_modules/@toft/react/es/button/types.d.ts`
- **源码实现**: `/Users/lianzimeng/working/happy/scripts/toft-guard-demo/node_modules/@toft/react/es/button/Button.js`

---

### Button 组件 Props 完整列表

#### 类型枚举定义：

```typescript
enum ColorType {
  default = "default",
  primary = "primary",
  success = "success",
  warning = "warning",
  danger = "danger"
}

enum FillType {
  solid = "solid",
  outline = "outline",
  dashed = "dashed",
  text = "text",
  none = "none"
}

enum ShapeType {
  circle = "circle",
  round = "round",
  square = "square",
  text = "text"
}

enum SizeType {
  max = "max",
  large = "large",
  middle = "middle",
  small = "small",
  min = "min"
}
```

#### ButtonProps 完整接口：

```typescript
interface ButtonProps extends Omit<React.ButtonHTMLAttributes<HTMLButtonElement>, 'size' | 'type'> {
  /**
   * 自定义类名前缀
   * @type string
   * @optional
   */
  prefixCls?: string;

  /**
   * 按钮颜色类型
   * @type 'default' | 'primary' | 'success' | 'warning' | 'danger'
   * @default 'default'
   */
  color?: keyof typeof ColorType;

  /**
   * 按钮填充模式
   * @type 'solid' | 'outline' | 'dashed' | 'text' | 'none'
   * @default 'solid'
   */
  fill?: keyof typeof FillType;

  /**
   * 按钮大小
   * @type 'max' | 'large' | 'middle' | 'small' | 'min'
   * @default 'middle'
   */
  size?: keyof typeof SizeType;

  /**
   * 按钮形状
   * @description 其中 text 建议使用 fill=text 来替代
   * @type 'circle' | 'round' | 'square' | 'text'
   * @default 'square'
   */
  shape?: keyof typeof ShapeType;

  /**
   * 加载状态（为 true 时按钮会显示加载动画并禁用）
   * @type boolean
   * @default false
   */
  loading?: boolean;

  /**
   * 按钮左侧图标
   * @type React.ReactNode
   * @optional
   */
  icon?: React.ReactNode;

  /**
   * 按钮右侧图标
   * @type React.ReactNode
   * @optional
   */
  rightIcon?: React.ReactNode;

  /**
   * 块级元素（宽度 100%）
   * @type boolean
   * @default false
   */
  block?: boolean;

  /**
   * 原生 HTML button type 属性
   * @type 'button' | 'submit' | 'reset'
   * @default 'button'
   */
  htmlType?: 'button' | 'submit' | 'reset';

  /**
   * 是否禁用按钮
   * @type boolean
   * 继承自 React.ButtonHTMLAttributes
   */
  disabled?: boolean;

  /**
   * 按钮内容
   * @type React.ReactNode
   * 继承自 React.ButtonHTMLAttributes
   */
  children?: React.ReactNode;

  /**
   * 点击事件处理器
   * @type (e: React.MouseEvent<HTMLButtonElement>) => void
   * 继承自 React.ButtonHTMLAttributes
   */
  onClick?: React.MouseEventHandler<HTMLButtonElement>;

  /**
   * 自定义样式
   * @type React.CSSProperties
   * 继承自 React.ButtonHTMLAttributes
   */
  style?: React.CSSProperties;

  /**
   * 自定义 CSS 类名
   * @type string
   * 继承自 React.ButtonHTMLAttributes
   */
  className?: string;

  // 其他 React.ButtonHTMLAttributes 支持的标准属性...
}
```

---

### 子组件：Button.Link

Button 组件有一个内置的 Link 子组件 `Button.Link`，用于创建链接样式的按钮。

#### ButtonLinkProps 接口：

```typescript
interface ButtonLinkProps extends AnchorHTMLAttributes<HTMLAnchorElement> {
  /**
   * 自定义类名前缀
   * @type string
   * @optional
   */
  prefixCls?: string;

  /**
   * 链接按钮的左侧图标
   * @type React.ReactNode
   * @optional
   */
  icon?: React.ReactNode;

  /**
   * 是否禁用链接按钮
   * @type boolean
   * @default false
   */
  disabled?: boolean;

  /**
   * 链接地址（来自 AnchorHTMLAttributes）
   * @type string
   */
  href?: string;

  /**
   * 链接内容（来自 AnchorHTMLAttributes）
   * @type React.ReactNode
   */
  children?: React.ReactNode;

  /**
   * 点击事件处理器（来自 AnchorHTMLAttributes）
   * @type React.MouseEventHandler<HTMLAnchorElement>
   */
  onClick?: React.MouseEventHandler<HTMLAnchorElement>;

  /**
   * 自定义 CSS 类名（来自 AnchorHTMLAttributes）
   * @type string
   */
  className?: string;

  // 其他 AnchorHTMLAttributes 支持的标准属性...
}
```

---

### 默认值

```typescript
Button.defaultProps = {
  color: 'default',
  size: 'middle',
  fill: 'solid',
  shape: 'square',
  htmlType: 'button'
}
```

---

### 组件特殊属性

```typescript
// Button 组件的静态标识，用于组件识别
Button.__TOFT_BUTTON = true;

// Button 子组件
Button.Link = ButtonLink;
```

---

### 使用示例

```typescript
import Button from '@toft/react';

// 基本按钮
<Button>Click me</Button>

// 带颜色的按钮
<Button color="primary">Primary Button</Button>
<Button color="success">Success Button</Button>
<Button color="warning">Warning Button</Button>
<Button color="danger">Danger Button</Button>

// 不同填充模式
<Button fill="outline">Outline Button</Button>
<Button fill="dashed">Dashed Button</Button>
<Button fill="text">Text Button</Button>
<Button fill="none">None Fill Button</Button>

// 不同大小
<Button size="max">Max Size</Button>
<Button size="large">Large Button</Button>
<Button size="middle">Middle Button</Button>
<Button size="small">Small Button</Button>
<Button size="min">Min Size</Button>

// 不同形状
<Button shape="circle">●</Button>
<Button shape="round">Round</Button>
<Button shape="square">Square</Button>

// 带图标
<Button icon={<Icon type="download" />}>Download</Button>
<Button icon={<Icon type="download" />} rightIcon={<Icon type="arrow-right" />}>
  Download
</Button>

// 加载状态
<Button loading>Loading...</Button>

// 禁用状态
<Button disabled>Disabled</Button>

// 块级按钮
<Button block>Full Width Button</Button>

// 提交表单
<Button htmlType="submit">Submit</Button>

// Link 按钮（需要通过 Button.Link 访问）
<Button.Link href="/page">Link Button</Button.Link>
<Button.Link href="/page" icon={<Icon type="external-link" />}>
  Open Link
</Button.Link>
<Button.Link href="/page" disabled>
  Disabled Link
</Button.Link>
```

---

### 类名规则

组件生成的 CSS 类名规则（使用 prefixCls 或默认的 'toft-button'）：

- `{prefixCls}-disabled`: 禁用状态
- `{prefixCls}-{size}`: 大小类 (size !== 'middle')
- `{prefixCls}-block`: 块级元素
- `{prefixCls}-only-icon`: 仅有图标（无文本）
- `{prefixCls}-{color}`: 颜色类
- `{prefixCls}-{color}-{fill}`: 颜色+填充组合类
- `{prefixCls}-align`: 有加载/图标时的对齐类
- `{prefixCls}-shape-{shape}`: 形状类
- `{prefixCls}-link`: Link 按钮类
- `{prefixCls}-link-disabled`: Link 按钮禁用类
- `{prefixCls}-icon`: 图标容器
- `{prefixCls}-text`: 文本容器
- `{prefixCls}-right-icon`: 右侧图标容器
- `{prefixCls}-icon-text`: 图标+文本组合类
- `{prefixCls}-right-icon-text`: 右侧图标+文本组合类

---

### 总结

**Button 组件特点：**
1. ✅ 完整的颜色系统（5 种颜色）
2. ✅ 灵活的填充模式（5 种模式）
3. ✅ 多种大小选项（5 种尺寸）
4. ✅ 多种形状支持（4 种形状）
5. ✅ 内置 Loading 动画支持
6. ✅ 支持左右图标
7. ✅ 内置 Link 子组件用于链接按钮
8. ✅ 支持块级显示
9. ✅ 支持所有标准 HTML button 属性
10. ✅ ConfigProvider 主题支持

**没有独立的 ButtonGroup 组件**，但 Button 本身已经是功能完整的组件。
