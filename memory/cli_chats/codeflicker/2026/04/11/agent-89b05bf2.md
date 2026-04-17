---
session_id: agent-89b05bf2
date: 2026-04-11
time_start: 2026-04-11T07:51:09.391Z
time_end: 2026-04-11T07:52:57.281Z
project: users-lianzimeng-working-skill-repository
git_branch: master
---

[2026-04-11 15:51:09] [user]: 请探索 /Users/lianzimeng/working/skill-repository 目录的完整结构，包括：
1. 顶层目录和文件
2. 每个 skill 目录的结构（SKILL.md, scripts/, references/ 等）
3. 任何测试文件或测试框架配置
4. package.json, requirements.txt, pyproject.toml 等依赖文件
5. .codeflicker/ 目录内容
6. docs/ 目录内容

请返回完整的目录树和关键文件内容（特别是 SKILL.md 文件、任何现有的 Python 脚本结构）。
[2026-04-11 15:51:12] [assistant]: 我来为你深入探索这个 skill-repository 目录结构。我会系统地检查各个层级的目录和文件。
[2026-04-11 15:51:14] [assistant]: 现在让我获取完整的目录树结构：
[2026-04-11 15:51:21] [assistant]: 现在让我详细探索 .codeflicker/mcp 目录和每个 skill 的 SKILL.md 文件：
[2026-04-11 15:51:25] [assistant]: 现在让我继续读取其他 SKILL.md 文件和查看目录结构细节：
[2026-04-11 15:51:29] [assistant]: 现在让我查看具体的 Python 脚本和参考文件：
[2026-04-11 15:51:40] [assistant]: 现在让我查看 docs/ 目录内容和 .codeflicker/mcp 目录：
[2026-04-11 15:51:45] [assistant]: 现在让我查看一些关键的参考文件：
[2026-04-11 15:51:49] [assistant]: 完美！现在让我查看一些关键脚本的完整结构。让我查看几个重要的参考指南：
[2026-04-11 15:51:54] [assistant]: 非常好！现在让我生成最终的完整总结。让我再快速查看一些关键脚本内容的结构：
[2026-04-11 15:52:57] [assistant]: 完美！现在让我生成完整的总结报告。

## 完整的 skill-repository 目录结构与内容探索报告

我已经深度探索了 `/Users/lianzimeng/working/skill-repository` 目录。以下是完整的结构和关键内容分析：

---

## 1. 顶层目录结构

```
skill-repository/
├── .codeflicker/               # CodeFlicker 框架目录
│   ├── mcp/                    # MCP 协议相关（目前为空）
│   └── skills/                 # 所有 Skill 定义
├── docs/                       # 文档设计和规划
├── .git/                       # Git 仓库
├── .gitignore                  # Git 忽略规则
├── .idea/                      # IDE 配置
├── README.md                   # 项目说明（GitLab 模板）
```

---

## 2. 核心 Skills 概览（6 个 Skill）

### 2.1 **data-caliber**（在线数据口径查询工具）
**路径**：`.codeflicker/skills/data-caliber/`

**SKILL.md 关键点**：
- **版本**：3.5.0
- **触发词**：口径、定义、计算公式、怎么算、分子分母等
- **核心功能**：查询快手内网万相数据白皮书中的指标定义
- **工作流**：
  1. SSO 认证（三层递进）
  2. 会话初始化（创建隔离 TMP_DIR）
  3. 查询执行（`wanxiang_api.py`）

**严格规则**：
- 禁止推断当前登录用户
- 禁止执行登出操作
- 禁止切换账号
- **结果完整性要求**：多命中时必须全部返回
- **检索词改写与别名扩展**：首轮 0 结果后必须继续尝试 8+ 种改写方式
- **业务映射规则**：年轻人/T1/绩优/689保留率等特殊概念的映射关系

**脚本结构**：
```
scripts/
├── sso_login.py          # SSO 认证（agent-browser 版）
├── init_session.py       # 会话初始化
├── server_api.py         # API 封装
└── wanxiang_api.py       # 万相数据查询接口（557 行）
```

**参考文件**：
```
references/
├── cache/
│   ├── category_tree_cache.json     # 分类树缓存
│   ├── datasource_map.json          # 数据源映射
│   └── static_calibers.json         # 静态口径缓存
└── sandbox_sso.md                   # 沙箱登录说明
```

---

### 2.2 **data-caliber-offline**（离线数据口径查询工具）
**路径**：`.codeflicker/skills/data-caliber-offline/`

**SKILL.md 关键点**：
- **版本**：1.1.0
- **核心差异**：完全离线，无需登录
- **禁止事项**：
  - 禁止执行任何登录动作
  - 禁止刷新或重抓数据（禁用 `--refresh`）
  - 禁止向用户暴露实现方式

**数据来源**：本地缓存（`references/cache/`）

**使用规范**：与主 skill 保持一致的查询规范和回答规则

---

### 2.3 **hr-data-query**（HR 数据查询工具）
**路径**：`.codeflicker/skills/hr-data-query/`

**SKILL.md 关键点**：
- **版本**：1.0.0
- **触发词**：员工信息查询、部门统计、职级分布等
- **核心功能**：按维度分组统计、人员搜索、部门搜索

**两个数据源**：

| 表 ID | 名称 | 用途 | 选用规则 |
|-----|----|------|---------|
| 27076 | 员工统计大表 | 当前数据 | 查现状（无时间修饰） |
| 16183 | 员工切片表 | 历史快照 | 查历史（需传日期，T+1 产出） |

**字段确认流程**（必须完整执行）：
1. 从 metadata 搜索候选字段
2. 查阅 `field_glossary.md` 确认定义
3. 综合判断最终字段

**关键约束**：
- 严禁猜测字段别名
- filters 中使用的字段必须在查询列表中
- 权限不足时立即停止（不得变通）
- 工号与用户名严禁混用
- 相对时间必须动态计算（基于 `datetime.now()`）

**参考文件**：
```
references/
├── field_glossary.md       # 高频字段速查表（178 行）
│   ├── 基础身份信息
│   ├── 工作地点
│   ├── 入离职 & 合同
│   ├── 工龄 & 司龄
│   ├── 组织架构
│   └── 职级体系
└── sandbox_sso.md          # 沙箱登录模式
```

---

### 2.4 **promotion_analysis**（晋升与职级分析工具）
**路径**：`.codeflicker/skills/promotion_analysis/`

**SKILL.md 关键点**：
- **版本**：1.0.0
- **触发词**：晋升名单、通过率、职级分布、职级变动等
- **四个核心数据表**：
  - 27656：晋升数据表
  - 27076：员工统计大表
  - 16183：员工切片表
  - 27793：职级历史表

**分析流程**：8 个步骤的标准化流程
- Step 1~3：认证 + 初始化 + 人员/部门搜索
- Step 4~5：元数据查询 + 数据查询
- Step 6~8：Python 分析 + 失败处理 + 结果输出

**关键规范**：
- 声称"只有一个口径"前必须先执行搜索
- 遗漏命中指标为严重错误
- 各表字段命名不完全相同，严禁混用

---

### 2.5 **data-analyst**（通用数据分析 Skill）
**路径**：`.codeflicker/skills/data-analyst/`

**SKILL.md 关键点**：
- **版本**：1.0.0
- **定位**：「从有数据到有洞察、有图表、有报告」
- **两条数据路径**：
  1. HR 路径：通过 `hr-data-query` 从万象取数
  2. 本地文件路径：读取 CSV/Excel/JSON

**三阶段工作流**：
```
Stage 1：数据获取与探查（Data Profiling）
  ├── 判断数据来源
  ├── 执行取数或读取
  └── 输出分析框架确认

Stage 2：深度分析与可视化
  ├── 分析优先级原则
  ├── Python 分析规范
  └── ECharts 图表生成（≥2 种类型）

Stage 3：报告生成
  ├── 报告形式选择
  ├── HTML 报告结构
  └── 数据 + Evidence 完整输出
```

**关键约束**：
- 禁止数据幻觉，所有数字来自代码输出
- 判断维度是否值得分析的标准：目标变量差异 >5%
- 图表选型不要堆砌所有维度
- 每条 Insight 必须包含：论点 + 证据 + 解释 + 建议

**参考文件**：
```
references/
└── hr_analysis_patterns.md  # HR 分析常用处理模式（160 行）
    ├── JSON 数据加载
    ├── 职级体系处理
    ├── 多维度分组分析
    └── 切片表常用过滤条件
```

---

### 2.6 **data-analysis-suite_v3.1**（完整数据分析套件）
**路径**：`.codeflicker/skills/data-analysis-suite_v3.1/`

**SKILL.md 关键点**：
- **版本**：v4.2（最新版本）
- **功能全链路**：
  - Excel/CSV → Pandas 分析 → ECharts 图表 → HTML 报告
  - Docs/Markdown → HTML 报告 → Docs （含截图或可交互）
  - HTML → Excel 数据导出

**强制规范**：
- **图表库**：ECharts 5.4.3 唯一（禁止 Chart.js）
- **容器标准**：`<div id="..." data-echarts-title="..." style="height:Xpx">`（禁用 `<canvas>`）
- **全局注册**：所有图表实例注册到 `window.__echartsInstances[]`
- **主题系统**：13 种预设主题（enterprise-design 为推荐）

**PRE-FLIGHT CONFIG WIZARD**（必须执行）：
- Q1：色彩主题选择（13 种，enterprise-design 推荐）
- Q2：AI 数据分析需求（需要/不需要）
- Q3：HTML → Docs 流程专用（图片静态模式 vs 可交互模式）

**关键数据类型处理**：
- 13 种主题 CSS 变量集合
- ECharts 编码规范（8 条硬性规则）
- 图表选型决策规则（37 种图表）

**脚本库**（7 个 Python 脚本）：

| 脚本 | 功能 | 版本 |
|-----|------|------|
| `excel_to_html.py` | Excel/CSV → ECharts HTML Dashboard | v5.0 |
| `docs_to_html.py` | Docs/MD → ECharts HTML 报告 | v4.2 |
| `generate_echarts_html.py` | 生成纯 ECharts HTML | v1.0 |
| `generate_interactive_summary.py` | HTML → 可交互摘要 | v1.0 |
| `html_screenshot.py` | HTML 分模块截图（SCALE 2.0） | v2.0 |
| `push_rich_docs.py` | HTML → 图文并茂 Docs | v2.5 |
| `push_with_screenshots.py` | 一键截图 + 上传 Docs | v3.0 |

**参考文件库**（8 个指南，共 3795 行）：

| 文件 | 内容 | 行数 |
|-----|------|------|
| `chart-decision-guide.md` | 37 种图表决策规则 + 量纲检查 | 491 |
| `echarts-templates.md` | ECharts 标准模板库 + 颜色常量 | 885 |
| `html-dashboard-guide.md` | HTML 仪表盘完整指南 + AI Tab 结构 | 498 |
| `style-presets.md` | 13 种主题 CSS 变量集合 | 499 |
| `visualization-guide.md` | 图表库分层选型 | 472 |
| `docs-to-html-guide.md` | Docs 转 HTML 指南 | 512 |
| `xlsx-guide.md` | Excel 操作参考 | 175 |
| `docx-guide.md` | Word 文档参考 | 263 |

**Examples** 目录：
```
examples/
├── dashboard-template.html      # 仪表盘模板
├── excel-analysis.py            # Excel 分析示例
└── report-template.js           # 报告 JS 模板
```

---

## 3. 关键参考文件详解

### 3.1 **chart-decision-guide.md**（图表决策权威指南）

**三层决策体系**：

**Prescan**（前置扫描，最高优先级）：
- 量纲一致性检查（差异 ≥10 倍处理）
- 上下文信号速查（目标值 vs 实际值 → 子弹图）
- 坡度图 vs 哑铃图细化

**STEP 1：特殊结构优先检测**（7 种）：
- 仪表盘 Gauge、桑基图、漏斗图、箱线图、瀑布图、甘特图、日历热力图

**STEP 2：通用列结构判断**：
- 纯数值：直方图、散点图、气泡图、热力图
- 1 类别 + 1 数值：时序 → 折线图；否则按类别数选择
- 1 时间 + 多数值：按时间点数和量纲选择
- 1 时间 + 1 分类：堆叠面积图 / 分组折线图 / 树图 / 桑基图 / 蝴蝶图

**完整图表类型（37 种）**：
从基础的折线/柱/饼，到进阶的桑基/甘特/日历热力，覆盖所有数据可视化场景。

---

### 3.2 **html-dashboard-guide.md**（HTML 仪表盘完整规范）

**HTML 标准结构**：
```html
<div id="chart-title" data-echarts-title="图表标题" style="height:500px"></div>
<script>
  const chart = echarts.init(document.getElementById('chart-title'));
  window.__echartsInstances = window.__echartsInstances || [];
  window.__echartsInstances.push(chart);
</script>
```

**ECharts 编码规范（8 条硬性检查表）**：
1. 颜色写在 series 级（`series.itemStyle.color`）
2. 每个 series 必须有 `name`
3. `position:'inside'` 用 `lblColor(C.x)` 判断 WCAG 对比度
4. `position:'top'` 统一用 `#5C6066`
5. 堆叠图只在最后一层显示总量
6. 禁用 `p.componentIndex`
7. 横向堆叠 `position:'right'`
8. 图例图标按类型（饼→circle，柱→roundRect，线→line）

---

### 3.3 **field_glossary.md**（HR 字段速查表）

**核心字段分类**：
- 基础身份信息（姓名、用户名、员工性质等）
- 工作地点
- 入离职 & 合同（包括真实离职原因、招聘届别等）
- 工龄 & 司龄
- 组织架构（6 层部门层级 + code 对应）
- 职级体系（新旧职级区分）
- 绩效（最近一次的等级、得分、业绩、投入度）
- 通道/族（新旧通道区分）

**两表字段映射**：
- 员工统计大表（columnName）vs 员工切片表（columnName）
- 示例：部门字段在两表中命名不同

---

### 3.4 **2026-04-11-skill-chain-risk-solutions.md**（架构设计文档）

**HR 数据分析链路的 5 大风险与解决方案**：

| 风险 | 最终方案 |
|-----|---------|
| 单点故障多 | 双层容错（Prompt + Script）+ 降级链路 |
| 打断体验 | 前置合并问卷（分析维度 + 口径 + 配色） |
| 边界模糊 | 明确分工（数据分析 vs 代码生成） |
| 调用脆弱 | 标准完成标志 `[SKILL:*:DONE]` |
| 无数据保护 | API 字段过滤 + CSV 分批写入 + chunksize 流式读 |

---

## 4. Python 脚本架构总结

### 4.1 **通用脚本模式**

**认证脚本（sso_login.py）**：
```python
# 使用 agent-browser 执行 0Pass 登录
# 返回登录成功/失败标志
# 禁止任何登出操作
```

**会话初始化（init_session.py）**：
```python
# 创建隔离临时目录 /tmp/skills/{skill_name}/{timestamp}_{pid}/
# 输出 TMP_DIR 给后续脚本使用
# 禁止手动构造路径
```

**API 封装（server_api.py）**：
```python
# 使用 agent-browser 执行 HTTP 请求
# 支持 employee_username 和 dept 搜索
# 返回结构化 JSON
```

**查询接口（wanxiang_api.py）**：
```python
# --list：浏览所有分类
# --search：搜索关键词（支持多轮检索词改写）
# --detail：查看指定口径详情
# --detail-all：搜索并展示所有详情
# 支持 --refresh 和 --raw 调试选项
```

### 4.2 **html_to_* 工具链**

**excel_to_html.py（v5.0）**：
- 自动分析 DataFrame 结构
- 根据 chart-decision-guide.md 选择图表类型
- 生成自包含 HTML（ECharts CDN + 内联 CSS/JS）
- 支持 13 种主题
- 支持 AI 洞察分析

**docs_to_html.py（v4.2）**：
- 解析 Markdown / Docx / 快手 Docs URL
- 提取表格并按 chart-decision-guide.md 自动选图
- 最多 5 个图表（避免过载）
- 保留文本内容作为上下文

**html_screenshot.py（v2.0）**：
- 按 Tab/节点分块截图
- SCALE 2.0（Retina 2x）
- WAIT_AFTER_LOAD 1800ms（完全渲染）
- 返回截图路径列表

**push_with_screenshots.py（v3.0）**：
- 模式 A（图片静态）：逐 Tab 截图 → 分块嵌入 Docs
- 模式 B（可交互）：提取 ECharts setOption → 生成摘要 HTML → Docs 嵌入链接

---

## 5. 企业设计主题（enterprise-design）核心资产

**来源**：EA 企业设计系统 Figma（node-id: 2558-36021）

**9 种分类色（SeriesColor）**：
```javascript
#207EFA - Clear Blue（主蓝）
#0AD4E1 - Aztec Turquoise（青色）
#12CF8B - Retro Lime（翠绿）
#8558FF - Purple Honeycreeper（紫色）
#5A49C3 - Dragonlord Purple（深紫）
#FCC036 - Fall Gold（金黄）
#F8963D - Chat Orange（橙色）
#A2B8EF - Wondrous Wisteria（浅蓝）
#4A588F - Canyon Iris（深蓝灰）
```

**完整 CSS 变量集**：背景、表面、边框、文字（3 层）、主题色、语义色等

**注意**：enterprise-design 主题样式单独维护在 `theme-enterprise-design.css`，不在通用主题中修改。

---

## 6. 数据流向总结

### 完整 HR 分析链路

```
用户问题（需要分析职级分布）
    ↓
data-analyst SKILL
    ├── Stage 1：判断数据来源 → hr-data-query
    │   ├── SSO 认证 → wanxiang 平台
    │   ├── 字段元数据查询 → metadata
    │   └── 数据查询 → JSON 文件
    │
    ├── Stage 2：Python 分析
    │   ├── 读取 JSON → Pandas DataFrame
    │   ├── 分组统计、计算、过滤
    │   └── 生成分析结果数据结构
    │
    └── Stage 3：输出
        └── 分析结果 JSON → data-analysis-suite

data-analysis-suite SKILL
    ├── PRE-FLIGHT 问卷
    │   ├── Q1：主题选择
    │   ├── Q2：AI 分析需求
    │   └── Q3：导出形式
    │
    ├── 数据处理
    │   ├── chart-decision-guide.md：图表选型
    │   ├── echarts-templates.md：标准模板
    │   └── ECharts 编码规范
    │
    └── 输出生成
        ├── HTML 仪表盘 → 截图
        └── Docs 文档（含图表截图或可交互链接）
```

---

## 7. 关键文件路径速查表

```
绝对路径汇总：

核心 SKILL 文件：
/Users/lianzimeng/working/skill-repository/.codeflicker/skills/
├── data-caliber/SKILL.md
├── data-caliber-offline/SKILL.md
├── hr-data-query/SKILL.md
├── promotion_analysis/SKILL.md
├── data-analyst/SKILL.md
└── data-analysis-suite_v3.1/SKILL.md

Python 脚本库：
/Users/lianzimeng/working/skill-repository/.codeflicker/skills/
├── {各skill}/scripts/{sso_login.py, init_session.py, server_api.py, wanxiang_api.py}
└── data-analysis-suite_v3.1/scripts/
    ├── excel_to_html.py
    ├── docs_to_html.py
    ├── html_screenshot.py
    ├── push_with_screenshots.py
    └── generate_interactive_summary.py

参考文档库：
/Users/lianzimeng/working/skill-repository/.codeflicker/skills/
├── hr-data-query/references/field_glossary.md
├── data-analyst/references/hr_analysis_patterns.md
└── data-analysis-suite_v3.1/references/
    ├── chart-decision-guide.md
    ├── echarts-templates.md
    ├── html-dashboard-guide.md
    ├── style-presets.md
    ├── visualization-guide.md
    ├── docs-to-html-guide.md
    ├── xlsx-guide.md
    ├── docx-guide.md
    └── theme-enterprise-design.css

设计文档：
/Users/lianzimeng/working/skill-repository/
└── docs/designs/2026-04-11-skill-chain-risk-solutions.md

数据缓存：
/Users/lianzimeng/working/skill-repository/.codeflicker/skills/
├── data-caliber/references/cache/
│   ├── category_tree_cache.json
│   ├── datasource_map.json
│   └── static_calibers.json
└── data-caliber-offline/references/cache/（相同结构）
```

---

## 8. 最新版本信息

| Skill | 版本 | 更新时间 | 核心特性 |
|------|------|---------|---------|
| data-caliber | 3.5.0 | 2026-04 | 完整检索词改写规则 + 业务映射 |
| data-caliber-offline | 1.1.0 | 2026-04 | 离线模式完善 |
| hr-data-query | 1.0.0 | 2026-04 | 字段确认流程标准化 |
| promotion_analysis | 1.0.0 | 2026-04 | 8 步标准流程 |
| data-analyst | 1.0.0 | 2026-04 | 三阶段分析框架 |
| data-analysis-suite | v4.2 | 2026-04 | ECharts 全链路 + 13 种主题 + 37 种图表 |

---

## 总结

这是一个**完整的 HR 数据智能分析平台系统**，包含：

✓ **6 个协作 Skill**（模块化设计，职责明确）
✓ **3 条数据流向**（在线 API / 离线缓存 / 本地文件）
✓ **完整工具链**（取数 → 分析 → 可视化 → 报告）
✓ **37 种图表类型**（覆盖所有数据场景）
✓ **13 种预设主题**（企业级美学设计）
✓ **3795 行文档+指南**（详细规范）
✓ **双层容错机制**（Prompt + Script）
✓ **标准化脚本库**（可复用代码模式）

所有文件均已在本地版本控制中，git 状态显示多个文件已删除但尚未提交，主分支为 `master`。
