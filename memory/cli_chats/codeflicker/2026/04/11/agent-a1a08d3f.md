---
session_id: agent-a1a08d3f
date: 2026-04-11
time_start: 2026-04-11T08:07:58.763Z
time_end: 2026-04-11T08:08:43.816Z
project: users-lianzimeng-working-skill-repository
git_branch: master
---

[2026-04-11 16:07:58] [user]: 请分析 /Users/lianzimeng/working/skill-repository 目录下所有 skill 的结构和作用。重点阅读每个 skill 目录下的 SKILL.md 文件（如果存在）以及主要脚本文件。请返回每个 skill 的名称和功能描述。
[2026-04-11 16:08:01] [assistant]: 我来帮你分析 skill-repository 目录下所有 skill 的结构和作用。首先，让我探索目录结构。
[2026-04-11 16:08:06] [assistant]: 看起来 SKILL.md 文件在 git 中被删除了。让我检查 git 历史中的这些文件，并查看 .codeflicker/skills 目录下的 SKILL.md 文件：
[2026-04-11 16:08:11] [assistant]: 现在让我查看README文件以获取更多上下文信息：
[2026-04-11 16:08:43] [assistant]: 完美！现在我有了所有必要的信息。让我整理一份详细的分析报告。

## Skill Repository 结构与功能分析报告

基于对 `/Users/lianzimeng/working/skill-repository` 目录下所有 skill 的分析，以下是每个 skill 的详细功能描述：

---

### 1. **data-analysis-suite** (v4.2.0)
**位置**: `.codeflicker/skills/data-analysis-suite_v3.1/`

**功能**: 统一的数据到展示全流程工具

**核心作用**:
- Excel/CSV 表格数据分析与处理（支持 .xlsx, .xls, .csv 格式）
- Word 文档 (.docx, .doc) 处理
- 生成交互式 HTML 数据看板（使用 ECharts 5.4.3）
- Markdown/文档转换为美观的 HTML 页面
- HTML 数据导出回 Excel
- 支持 13 种视觉主题（企业设计、商务简约、深色科技风等）
- AI 数据分析与洞察生成
- 支持 37 种图表类型（柱状图、饼图、折线图、散点图、热力图等）

**关键脚本**:
- `excel_to_html.py`: Excel/CSV → 交互式 HTML（v5.0）
- `docs_to_html.py`: 文档 → HTML 报告
- `generate_interactive_summary.py`: HTML → 可交互摘要
- `push_rich_docs.py`: HTML → 图文并茂 Docs 文档
- `html_screenshot.py`: HTML 分模块高清截图工具
- `generate_echarts_html.py`: ECharts HTML 生成器

**适用场景**: 
- 需要可视化展示数据的各种业务分析
- 生成交互式仪表板和报告
- 数据图表生成与美化

---

### 2. **data-analyst** (v1.0.0)
**位置**: `.codeflicker/skills/data-analyst/`

**功能**: 通用数据分析与洞察工具

**核心作用**:
- 从原始数据到洞察、图表、报告的完整工作流
- HR 数据分析（职级分布、离职分析、招聘质量、人才盘点、梯队分析）
- 本地文件分析（CSV、Excel、JSON）
- 仪表板创建
- 趋势分析、分布分析、比较分析
- 深度数据洞察提炼
- 自动报告生成

**三阶段工作流**:
1. **Stage 1**: 数据获取与探查（Data Profiling）- 前置偏好问卷、数据源判断、HR 取数路径、本地文件读取
2. **Stage 2**: 深度分析与可视化 - 分析优先级原则、Python 分析、ECharts 图表生成
3. **Stage 3**: 报告生成 - HTML/Markdown/Excel 格式输出

**适用场景**: 
- HR 人力指标分析
- 需要生成专业分析报告的场景
- 数据驱动的商业决策支持

---

### 3. **data-caliber** (v3.5.0)
**位置**: `.codeflicker/skills/data-caliber/`

**功能**: 快手内网人力指标口径查询工具

**核心作用**:
- 查询快手内网万相数据白皮书中的指标定义与数据口径
- 支持 HR 指标完整库查询
- 返回指标的定义、公式、分子分母、统计范围等信息

**支持的指标查询包括**:
- 年轻人相关：年轻人、年轻人跃迁、晋升年轻人占比
- 职级相关：T1/T2/T4、平均职级、职级停留
- 绩效相关：绩优、绩效 CD、绩优离职
- 招聘相关：校招生、名校率、校招浓度
- 离职相关：离职、主动离职、被动离职、绩优离职
- 保留相关：盘点689保留率、T1保留率、绩优保留率、汰换率
- 管理相关：管理浓度、内升占比
- 其他：入职、YTD 入职人数、月离职率、YTD 离职率

**执行流程**:
1. SSO 内网认证（支持 0Pass 登录和沙箱模式）
2. 会话初始化
3. 通过 `wanxiang_api.py` 查询口径
4. 检索词改写与别名扩展
5. 多命中完整返回（禁止遗漏）

**适用场景**: 
- 需要查询 HR 指标的精确定义和计算规则
- 确认数据统计口径和范围

---

### 4. **data-caliber-offline** (v1.1.0)
**位置**: `.codeflicker/skills/data-caliber-offline/`

**功能**: 离线版本的人力指标口径查询工具（无需在线登录）

**核心作用**:
- 与 data-caliber 功能相同，但使用本地离线数据
- 无需 SSO 认证和在线接口调用
- 支持同样的指标查询功能

**关键差异**:
- **禁止**执行任何登录操作（`sso_login.py`, `init_session.py`）
- **禁止**刷新或重新抓取数据
- **禁止**向用户暴露实现细节（离线模式、本地缓存等）

**本地数据源**:
- `references/cache/category_tree_cache.json`
- `references/cache/datasource_map.json`
- `references/cache/static_calibers.json`

**适用场景**: 
- 需要快速查询指标口径但无内网访问
- 离线工作环境
- 演示/培训场景

---

### 5. **hr-data-query** (v1.0.0)
**位置**: `.codeflicker/skills/hr-data-query/`

**功能**: HR 数据查询与统计分析工具

**核心作用**:
- 员工数据查询：按姓名/工号/部门查询员工信息
- 组织架构查询：部门、在离职状态
- 分组统计：按性质/职级/学历/性别/工作地点等维度统计

**数据源**:
- 表 27076：员工统计大表（当前最新数据，T0）
- 表 16183：员工切片表（历史快照数据，T+1 产出）

**执行流程**:
1. SSO 内网认证
2. 会话初始化（获取隔离 TMP_DIR）
3. 人员/部门搜索（可选）
4. 查询字段元数据
5. 查询数据（支持大数据量分批拉取 >50,000 行）
6. Python 分析代码执行
7. 结果输出

**关键特性**:
- 支持 2 张员工表的灵活切换
- 大数据量分批拉取（每批 5000 行）
- 严格的字段权限检查
- 动态时间计算（禁止硬编码日期）
- 详细的错误处理与降级策略

**适用场景**: 
- 员工信息查询
- 组织统计分析
- HR 报表生成基础数据

---

### 6. **promotion_analysis** (v1.0.0)
**位置**: `.codeflicker/skills/promotion_analysis/`

**功能**: 晋升与职级变动分析工具

**核心作用**:
- 晋升提报、评审与通过全流程分析
- 职级变动追踪与分析
- 职级分布统计

**支持的分析场景**:
- 晋升名单与明细
- 晋升通过率
- 破格/直通晋升查询
- 门槛筛选
- 职级停留时长
- 30% 跃迁分析
- 春/秋季窗口对比
- 晋升历史追踪
- 职级变动历史
- 晋升/降级/调整筛选
- 职级分布查询
- 通道转换
- 变动原因分析

**核心数据表**:
- 表 27656：晋升数据表（核心，一人一周期一条记录）
- 表 27076：员工统计大表（补充员工基础信息）
- 表 16183：员工切片表（历史数据）
- 表 27793：职级历史表（职级变动流水）

**执行流程**:
1. SSO 内网认证
2. SKILL 工作目录确定
3. 人员/部门基础信息搜索
4. 字段元数据查询
5. 数据查询
6. Python 分析执行
7. 结果输出

**适用场景**: 
- 晋升相关的数据分析与报表
- 职级发展趋势分析
- 人才梯队评估

---

## 完整 Skills 清单

| Skill 名称 | 版本 | 主要功能 | 数据源 | 认证方式 |
|-----------|------|--------|--------|--------|
| **data-analysis-suite** | v4.2.0 | 数据→展示全流程（13 种主题, 37 种图表) | 本地文件 | 无 |
| **data-analyst** | v1.0.0 | 数据分析与洞察生成 | 本地+HR 系统 | 通过子技能 |
| **data-caliber** | v3.5.0 | HR 指标口径查询（在线） | 快手万相系统 | SSO 0Pass |
| **data-caliber-offline** | v1.1.0 | HR 指标口径查询（离线） | 本地缓存 | 无 |
| **hr-data-query** | v1.0.0 | 员工数据查询与统计 | 快手万相系统 | SSO 0Pass |
| **promotion_analysis** | v1.0.0 | 晋升与职级分析 | 快手万相系统 | SSO 0Pass |

---

## Skills 之间的依赖关系

```
data-analysis-suite
├── 用于：可视化任何分析结果
└── 由以下工具提供数据：
    ├── data-analyst（生成分析报告）
    ├── hr-data-query（获取员工数据）
    └── promotion_analysis（获取晋升数据）

data-analyst
├── 依赖：hr-data-query（获取数据）
└── 输出给：data-analysis-suite（生成报告）

data-caliber / data-caliber-offline
└── 用于：确认数据指标的精确定义

hr-data-query
├── 数据源：快手万象系统
└── 输出给：data-analyst, promotion_analysis

promotion_analysis
├── 依赖：hr-data-query（员工基础信息）
└── 核心数据表：27656（晋升数据表）
```

---

## 关键技术栈

- **Python**: pandas, openpyxl, beautifulsoup4, markdown2, python-docx
- **前端**: ECharts 5.4.3（CDN），HTML/CSS/JavaScript
- **认证**: SSO 0Pass（快手内网）
- **浏览器自动化**: agent-browser, Playwright (chromium)
- **数据源**: 快手内网万象数据平台

---

## 使用工作流示例

### 场景 1: 生成 HR 分析报告
```
用户问题 → data-analyst（偏好问卷）
→ hr-data-query（取数）
→ 数据分析（Stage 1-3）
→ data-analysis-suite（调用并生成 HTML/图表）
→ 输出报告给用户
```

### 场景 2: 查询指标口径
```
用户问："T1保留率怎么算？"
→ data-caliber（查询）或 data-caliber-offline（离线）
→ 返回公式、分子分母、统计范围
```

### 场景 3: 晋升数据分析
```
用户问："企业应用部 2025 年春季晋升通过率？"
→ promotion_analysis（晋升数据查询）
→ hr-data-query（员工基础数据补充）
→ 统计分析
→ 输出结果
```

这套 skill 体系形成了一个完整的数据分析生态，从数据获取、指标定义、数据分析到最终可视化展示。
