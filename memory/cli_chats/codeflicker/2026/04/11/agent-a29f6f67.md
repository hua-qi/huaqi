---
session_id: agent-a29f6f67
date: 2026-04-11
time_start: 2026-04-11T03:46:55.882Z
time_end: 2026-04-11T03:50:26.804Z
project: users-lianzimeng-working-skill-repository
git_branch: master
---

[2026-04-11 11:46:55] [user]: 分析 /Users/lianzimeng/working/skill-repository 项目的整体结构，重点关注：
1. 项目整体目录结构
2. 有哪些 skill 文件
3. 是否有关于图表（chart）渲染的代码，搜索关键词：chart、echarts、mermaid、图表、render
4. 查看主要的 skill 文件内容，了解 skill 的输出格式
5. 有没有关于 stargate 或 codeflicker 渲染的相关说明或文档

请返回：
- 完整的目录结构
- 所有 skill 文件列表和主要内容
- 图表相关代码或文档的内容
- 任何关于渲染格式的说明
[2026-04-11 11:47:00] [assistant]: 我来帮你全面分析这个项目的结构。让我并行执行多个搜索任务。
[2026-04-11 11:49:17] [assistant]: 很好！现在让我搜索图表相关的代码，并读取主要的 SKILL 文件和 README：
[2026-04-11 11:49:22] [assistant]: 现在让我读取所有的 SKILL.md 文件和 server_api.py 文件来查看详细内容：
[2026-04-11 11:49:28] [assistant]: 现在让我查看关于 stargate 和 codeflicker 的更多细节，以及 server_api.py 文件：
[2026-04-11 11:49:34] [assistant]: 现在让我查看更多 Python 脚本的详情，查看 wanxiang_api.py 来了解输出格式：
[2026-04-11 11:49:39] [assistant]: 现在让我获取完整的项目目录结构和查看一些参考文件：
[2026-04-11 11:49:45] [assistant]: 完美！现在让我查看缓存的 JSON 文件来理解数据格式：
[2026-04-11 11:49:51] [assistant]: 完美！现在让我生成完整的分析报告：
[2026-04-11 11:50:26] [assistant]: 现在我有足够的信息来生成完整的分析报告。让我汇总所有的发现：

## 项目结构分析报告

我已完成对 `/Users/lianzimeng/working/skill-repository` 项目的全面分析。以下是详细的发现：

---

## 1. 项目整体目录结构

```
/Users/lianzimeng/working/skill-repository/
├── .gitignore
├── README.md (GitLab项目模板)
├── data-caliber/                    # Skill 1: 数据口径查询工具（在线版）
│   ├── SKILL.md
│   ├── references/
│   │   ├── cache/
│   │   │   ├── category_tree_cache.json
│   │   │   ├── datasource_map.json
│   │   │   └── static_calibers.json
│   │   └── sandbox_sso.md
│   └── scripts/
│       ├── init_session.py
│       ├── server_api.py
│       ├── sso_login.py
│       └── wanxiang_api.py
├── data-caliber-offline/            # Skill 2: 数据口径查询工具（离线版）
│   ├── SKILL.md
│   ├── references/
│   │   ├── cache/
│   │   │   ├── category_tree_cache.json
│   │   │   ├── datasource_map.json
│   │   │   └── static_calibers.json
│   │   └── sandbox_sso.md
│   └── scripts/
│       ├── init_session.py
│       ├── server_api.py
│       ├── sso_login.py
│       ├── wanxiang_api.py
│       └── wanxiang_offline.py
├── hr-data-query/                   # Skill 3: HR数据查询工具
│   ├── SKILL.md
│   ├── references/
│   │   ├── field_glossary.md       # 字段速查表
│   │   └── sandbox_sso.md
│   └── scripts/
│       ├── init_session.py
│       ├── server_api.py
│       ├── sso_login.py
│       └── wanxiang_api.py
└── promotion_analysis/              # Skill 4: 晋升与职级分析工具
    ├── SKILL.md
    ├── references/
    │   └── sandbox_sso.md
    └── scripts/
        ├── init_session.py
        ├── server_api.py
        ├── sso_login.py
        └── wanxiang_api.py
```

---

## 2. 所有 Skill 文件列表和主要内容

### **Skill 1: data-caliber** (版本 3.5.0)
- **路径**: `/Users/lianzimeng/working/skill-repository/data-caliber/SKILL.md`
- **功能**: 万相数据口径查询工具（在线版本）
- **适用场景**: 查询快手内网万相数据白皮书中的指标定义与数据口径
- **触发场景**: 用户询问指标定义、口径、计算规则、计算公式、统计方式等
- **支持的指标**: 年轻人、年轻人跃迁、晋升年轻人占比、T1/T2/T4、绩优、绩效CD、校招生、名校率、管理浓度、内升占比、保留率、离职率、汰换率等
- **执行流程**: SSO认证 → 会话初始化 → 查询执行
- **数据来源**: 离线缓存 + 在线API

### **Skill 2: data-caliber-offline** (版本 1.1.0)
- **路径**: `/Users/lianzimeng/working/skill-repository/data-caliber-offline/SKILL.md`
- **功能**: 万相数据口径查询工具（离线版本）
- **与在线版的区别**: 
  - 禁止执行登录动作
  - 禁止刷新或重抓数据
  - 完全使用本地缓存
- **适用场景**: 无登录权限或需要快速查询时使用
- **执行流程**: 直接使用本地查询入口（无需SSO认证）

### **Skill 3: hr-data-query** (版本 1.0.0)
- **路径**: `/Users/lianzimeng/working/skill-repository/hr-data-query/SKILL.md`
- **功能**: HR数据查询与统计分析
- **支持的查询**: 
  - 按姓名/工号/部门查询员工信息
  - 在离职状态查询
  - 组织架构查询
  - 按性质/职级/学历/性别/工作地点等维度分组统计
- **数据源**: 
  - 员工统计大表 (tableId: 27076) - 当前最新数据
  - 员工切片表 (tableId: 16183) - 历史快照数据
- **关键约束**: 两表字段命名不同，严禁混用

### **Skill 4: promotion_analysis** (版本 1.0.0)
- **路径**: `/Users/lianzimeng/working/skill-repository/promotion_analysis/SKILL.md`
- **功能**: 晋升与职级分析工具
- **支持的分析**: 
  - 晋升名单、晋升通过率、破格/直通晋升
  - 职级停留、职级变动、职级分布
  - 晋升历史、通道转换、变动原因
- **数据源**: 
  - 晋升数据表 (tableId: 27656) - 核心表
  - 员工统计大表 (tableId: 27076)
  - 员工切片表 (tableId: 16183)
  - 职级历史表 (tableId: 27793)

---

## 3. 图表（Chart）渲染相关代码

### **搜索结果**：🔍 **未发现** Chart、ECharts、Mermaid 等图表渲染代码

**搜索关键词**: chart、echarts、mermaid、图表、render

**结论**: 
- 项目中**不包含**任何图表渲染库或代码
- 所有 Skill 专注于**数据查询和统计分析**，而非可视化
- 输出格式为**JSON、Markdown、Excel**，不涉及图表渲染

---

## 4. 主要 Skill 文件的输出格式

### **4.1 SKILL.md 的结构化说明**

每个 SKILL.md 都遵循以下元数据格式：

```yaml
---
name: <skill名称>
description: <功能描述，包含触发词>
version: <版本号>
---
```

### **4.2 Python 脚本输出格式**

#### **server_api.py** - 人员和部门搜索
```json
// 人员搜索结果示例
[
  {
    "username": "zhangsan",              // 邮箱前缀
    "orgDisplayName": "企业应用部/服务端组",  // 部门全路径
    "name": "张三"                        // 中文名
  }
]

// 部门搜索结果示例
[
  {
    "code": "D998333",          // 部门编码
    "name": "企业应用部",       // 部门名称
    "en_name": "企业应用部",    // 部门英文名
    "dept_level": "1级部门"     // 部门层级
  }
]
```

#### **wanxiang_api.py** - 数据口径查询
```json
// 分类树查询结果示例
[
  {
    "id": 72,
    "categoryName": "在岗",
    "proList": [
      {
        "id": 3,
        "name": "在岗人数"
      },
      // ...
    ]
  }
]

// 口径详情示例
{
  "name": "大盘T1保留率（大盘绩优保留）",
  "category": "大盘观测指标",
  "definition": "年初绩效标签为T1的员工，在年底仍在职的比例",
  "formula": "年底在职人数 / 年初T1人数",
  "numerator": "分母人群中截止年底在职的人数",
  "denominator": "年初绩效标签为T1的人数",
  "note": "关于T1/2/3/4的口径说明..."
}
```

#### **wanxiang_api.py** - 数据查询 (HR/晋升)
```json
// dataList 返回结果示例
[
  {
    "t68__effective_start_date": 20260325,
    "t68__effective_end_date": 20260325,
    "t68__person_id": "36267",
    "t68__person_name": "张三",
    "t68__person_number": "zhangsan"
  }
]
```

#### **Python 数据分析输出**

```markdown
# 统计结果 (Markdown格式)
共 XX 条记录
分布：{分类1: 数量1, 分类2: 数量2}
详细数据已保存到 /path/to/data.xlsx

# Excel 导出
DataFrame.to_excel(file_path, index=False)
```

---

## 5. Stargate 和 Codeflicker 渲染相关说明

### **发现的 Stargate 引用**：

在 `server_api.py` 文件中，发现对 **Stargate** 的调用：

#### **data-caliber/scripts/server_api.py**
```python
BASE_URL = "https://stargate.corp.kuaishou.com"

API_MAP = {
    "employee_username": "/magic-ai/api/search/employee_username",
    "dept": "/magic-ai/api/search/dept",
}
```

#### **hr-data-query/scripts/server_api.py** 和 **promotion_analysis/scripts/server_api.py**
```python
BASE_URL = "https://stargate-home.corp.kuaishou.com"

API_MAP = {
    "employee_username": "/magic-ai/api/search/employee_username",
    "dept": "/magic-ai/api/search/dept",
}
```

#### **data-caliber-offline/scripts/server_api.py**
```python
BASE_URL = "https://stargate.corp.kuaishou.com"
```

### **Stargate 的实现方式**：

所有 API 调用通过 `agent-browser` 发送：

```python
def browser_post_json(url: str, body: dict) -> dict:
    # 使用 JavaScript 在浏览器环境中发送 POST 请求
    js = f"""
(async () => {{
  try {{
    const resp = await fetch('{url}', {{
      method: 'POST',
      credentials: 'include',
      headers: {{
        'Accept': 'application/json',
        'Content-Type': 'application/json',
        'Apptype': 'STARGATE_X'  // 关键头标识
      }},
      body: JSON.stringify({body_json})
    }});
    const text = await resp.text();
    return JSON.stringify({{ok: resp.ok, status: resp.status, body: text}});
  }} catch(e) {{
    return JSON.stringify({{error: e.message}});
  }}
}})()
"""
    # agent-browser eval 执行 JavaScript
    raw = ab(["eval", js], timeout=60)
    # 解析结果...
```

### **关于 Codeflicker**：

**搜索结果**: 🔍 **未发现** Codeflicker 的任何引用

项目中没有直接的 Codeflicker 相关代码或文档。基于代码结构推测：
- **Codeflicker** 可能是运行这些 Skills 的整体框架/平台
- Skills 通过输出 Markdown、JSON、Excel 等标准格式与 Codeflicker 交互
- 无需在代码中显式引用 Codeflicker

---

## 6. 关键技术栈与依赖

| 工具/库 | 用途 | 说明 |
|--------|------|------|
| `agent-browser` | 浏览器自动化 | npm install -g agent-browser |
| Python 3 | 数据处理脚本 | 所有 .py 脚本基础 |
| pandas | 数据分析 | 用于 DataFrame 操作和 Excel 导出 |
| requests/fetch | HTTP 请求 | 通过 agent-browser 或原生 JS |
| json | 数据序列化 | JSON 解析和序列化 |

---

## 7. 核心工作流程总结

```
用户输入
    ↓
选择对应 Skill
    ↓
SSO 认证 (如需要)
    ↓
初始化会话 (TMP_DIR)
    ↓
搜索基础信息 (人员/部门)
    ↓
查询元数据 (metadata)
    ↓
执行数据查询 (dataList via Stargate API)
    ↓
Python 数据分析处理
    ↓
生成输出:
  - Markdown (终端打印)
  - Excel (文件导出)
  - JSON (原始数据)
```

---

## 8. 重要约束和规范

| 约束项 | 规范说明 |
|--------|---------|
| **SSO认证** | 禁止推断当前登录用户，禁止强制切换账号 |
| **字段使用** | 严禁猜测字段别名，必须从 metadata 查询确认 |
| **两表混用** | 员工统计大表和切片表字段命名不同，严禁混用 |
| **权限处理** | 无权限时立即终止，禁止变通方式绕过 |
| **完整性要求** | 多命中时必须全部返回，不能只返回最相似的 |
| **时间计算** | 所有相对时间必须动态计算，禁止硬编码 |
| **数据输出** | 必须用 --output 存文件，不直接打印到终端 |

---

## 9. 缓存数据文件说明

### **category_tree_cache.json**
- 数据口径的分类树结构
- 包含所有分类及其对应的指标 ID 和名称
- 用于快速查询分类信息

### **datasource_map.json**
- 口径 ID 到数据源的映射
- 包含 JDBC 连接、数据库、表名、字段名等信息
- 每条口径可能对应多个数据源

### **static_calibers.json**
- 静态口径定义（非 API 数据）
- 包含 name、category、definition、formula、numerator、denominator、note 等完整元数据
- 用于离线查询和参考

---

## 总结

这是一个**企业级 HR 数据分析平台**，包含 4 个相互配合的 Skills：

1. **数据口径查询** - 查阅指标定义和计算公式
2. **HR 员工数据查询** - 查询具体员工信息和组织数据
3. **晋升职级分析** - 分析晋升和职级变动
4. **离线口径查询** - 无登录权限时的备用方案

**核心特点**：
- ✓ 无图表渲染（只输出数据）
- ✓ 使用 Stargate API 进行内网数据交互
- ✓ 严格的权限和数据完整性约束
- ✓ 支持本地缓存和离线使用
- ✓ 输出格式多样化（Markdown、Excel、JSON）
