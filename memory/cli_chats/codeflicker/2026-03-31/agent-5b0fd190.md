---
session_id: agent-5b0fd190
date: 2026-03-31
time_start: 2026-03-31T08:28:59.854Z
time_end: 2026-03-31T08:30:01.541Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-03-31 16:28:59] [user]: 请探索 /Users/lianzimeng/workspace/huaqi-growing 项目中与学习计划相关的代码和文档，包括：
1. docs/ 目录下的学习相关文档
2. huaqi_src/learning/ 目录的结构和主要功能
3. huaqi_src/cli/commands/study.py 的命令
4. 任何学习计划相关的 README 或使用说明

请返回：用户如何使用该项目制定学习计划的完整说明，包括 CLI 命令和主要功能。
[2026-03-31 16:29:02] [assistant]: 我来帮你探索项目中与学习计划相关的代码和文档。让我按步骤逐一分析。
[2026-03-31 16:29:05] [assistant]: 现在让我读取学习相关的文档和代码文件：
[2026-03-31 16:29:08] [assistant]: 现在让我查看学习模块的其他文件：
[2026-03-31 16:29:12] [assistant]: 现在让我查看 README 和快速开始文档：
[2026-03-31 16:29:16] [assistant]: 现在让我查看测试文件以获取更详细的使用示例：
[2026-03-31 16:29:19] [assistant]: 完美！现在我已经有了全面的信息。让我生成完整的使用说明文档：
[2026-03-31 16:30:01] [assistant]: 现在我有了完整的信息。让我生成详细的说明文档：

## 花旗学习助手完整使用说明

基于对项目的探索，我现在为你提供**用户如何使用该项目制定学习计划的完整说明**。

---

### 目录
1. [核心概念](#核心概念)
2. [CLI 命令完全指南](#cli-命令完全指南)
3. [Agent 对话示例](#agent-对话示例)
4. [数据存储结构](#数据存储结构)
5. [工作流程](#工作流程)
6. [高级功能](#高级功能)

---

## 核心概念

**学习助手（Learning Assistant）** 是 Huaqi 的智能学习系统，让用户通过 CLI 或 Agent 对话触发系统性学习：

| 组件 | 功能 |
|------|------|
| **大纲生成** | LLM 自动生成 6-10 章由浅入深的课程大纲，并存储为 YAML |
| **按章讲解** | 每次学习只推进一章，降低认知负担，生成讲解内容+考题 |
| **进度持久化** | 每章状态（pending/in_progress/completed）保存到本地 |
| **复习提醒** | 每晚 21:00 自动推送进行中课程的复习题 |
| **会话归档** | 每次学习的讲解、考题、回答、反馈记录在 Markdown 中 |

---

## CLI 命令完全指南

### 1. 列出所有课程与进度

```bash
huaqi study --list
# 或
huaqi study -l
```

**输出示例：**
```
学习课程进度
┏━━━━━━━┳━━━━━━━━━━━━━━━━━━━━┳━━━━━━━┳━━━━━━━┓
┃ 技术   ┃ 当前章节           ┃ 进度   ┃ 状态  ┃
┣━━━━━━━╋━━━━━━━━━━━━━━━━━━━━╋━━━━━━━╋━━━━━━━┫
┃ Rust   ┃ 第 2 章 借用        ┃ 1/3    ┃ ▶️ 学习中 ┃
┃ Python ┃ 第 1 章 基础语法    ┃ 0/4    ┃ ▶️ 学习中 ┃
┗━━━━━━━┻━━━━━━━━━━━━━━━━━━━━┻━━━━━━━┻━━━━━━━┛
```

**状态说明：**
- `✅` = 已完成
- `▶️` = 学习中
- `⬜` = 未开始

---

### 2. 开始或继续学习

```bash
# 开始学习新技术（首次自动生成大纲）
huaqi study rust
huaqi study "Python 3"
huaqi study Go
```

**首次运行效果：**
1. LLM 生成 6-10 章大纲
2. 显示第一章讲解内容（限 300 字）
3. 给出一道练习题
4. 本地保存课程大纲（YAML）和会话记录（Markdown）

**输出示例：**
```
📚 启动学习：Rust

## 📚 Rust · 第1章：所有权（Ownership）

所有权是 Rust 内存管理的核心机制。与其他编程语言不同，Rust 通过所有权系统在
编译时（而非运行时）确保内存安全，避免垃圾回收的开销。

Rust 有三个关键规则：
1. 每个值都有一个变量作为其所有者
2. 同时只能有一个所有者
3. 当所有者超出作用域，值将被 drop

总结：所有权是 Rust 确保内存安全的核心，通过所有权规则在编译时检查，无运行时开销。

---

### 🧠 练习题

以下这段代码，哪个选项会导致编译错误？

```rust
let s1 = String::from("hello");
let s2 = s1;          // s1 的所有权转移给 s2
println!("{}", s1);   // 会报错吗？
```

A. 不会报错，s1 和 s2 都可以使用
B. 会报错，s1 已失去所有权
C. 会报错，String 类型不支持复制

（回答后我会给你反馈，说「完成本章」可标记此章完成）
```

---

### 3. 重置课程进度

```bash
# 重置某个课程，删除本地大纲和进度
huaqi study rust --reset
huaqi study python --reset
```

**效果：**
- 删除 `~/.huaqi/memory/learning/courses/<slug>/` 目录
- 会话记录（Markdown）保留，便于回顾
- 下次运行 `huaqi study rust` 时重新生成大纲

---

### 4. 命令行语法参考

```bash
# 查看帮助
huaqi study --help

# 位置参数
huaqi study <skill>              # 要学的技术名称（可选）

# 选项
huaqi study --list              # 列出所有课程
huaqi study --reset             # 重置课程（与 skill 一起使用）
```

---

## Agent 对话示例

在 `huaqi chat` 对话模式下，Agent 会自动调用学习工具处理与学习相关的问题。

### 示例 1：查询学习进度

**用户输入：**
```
我 Rust 学到哪了？
```

**Agent 调用：** `get_learning_progress_tool(skill="Rust")`

**输出：**
```
📚 Rust 学习进度

当前章节：第 2 章（共 3 章）
已完成：1/3 章

章节列表：
✅ 第1章：所有权
▶️ 第2章：借用
⬜ 第3章：生命周期
```

---

### 示例 2：查看课程大纲

**用户输入：**
```
Rust 课程有哪些章节？
给我看一下 Python 的学习计划
```

**Agent 调用：** `get_course_outline_tool(skill="Rust")`

**输出：**
```
📖 Rust 课程大纲（共 3 章）

✅ 第1章：所有权
▶️ 第2章：借用
⬜ 第3章：生命周期
```

---

### 示例 3：开始学习

**用户输入：**
```
帮我开始学 Go
继续学 Python
出道题考我
```

**Agent 调用：** `start_lesson_tool(skill="Go")`

**输出：** 讲解内容 + 考题（同 CLI 示例）

---

### 示例 4：标记章节完成

**用户输入：**
```
我完成本章了
第一章完成了
我学完了
```

**Agent 识别关键词** → 调用进度更新逻辑 → **输出：**
```
✅ 已标记第 1 章完成
下一章：第 2 章 借用
准备好继续学习吗？
```

---

## 数据存储结构

### 目录树

```
~/.huaqi/memory/learning/
├── courses/                      # 课程大纲和进度
│   └── <slug>/                   # 技术 slug（e.g., rust, python-3）
│       ├── outline.yaml          # 课程大纲 + 进度
│       └── lessons/              # 预留（扩展用）
│
└── sessions/                     # 学习会话记录
    ├── 20260331_rust.md          # YYYYMMDD_<slug>.md 格式
    ├── 20260401_python.md
    └── ...
```

### outline.yaml 结构

**示例文件：** `~/.huaqi/memory/learning/courses/rust/outline.yaml`

```yaml
skill_name: Rust
slug: rust
created_at: '2026-03-31T14:30:00.123456'
current_lesson: 2                 # 下一个要学的章节
total_lessons: 3
lessons:
  - index: 1
    title: 所有权（Ownership）
    status: completed              # pending / in_progress / completed
    completed_at: '2026-03-31T15:20:00'
  - index: 2
    title: 借用（Borrowing）
    status: in_progress
    completed_at: null
  - index: 3
    title: 生命周期（Lifetimes）
    status: pending
    completed_at: null
```

### 会话记录 Markdown 格式

**示例文件：** `~/.huaqi/memory/learning/sessions/20260331_rust.md`

```markdown
# 学习会话：rust - 第1章 所有权（Ownership）

**时间：** 2026-03-31 15:10

## 讲解内容

所有权是 Rust 内存管理的核心机制。...

## 考题

以下代码会报错吗？
```rust
let s1 = String::from("hello");
let s2 = s1;
println!("{}", s1);
```

## 你的回答

选项 B，会报错

## 反馈

正确！这是所有权转移的经典例子。当 `let s2 = s1` 执行时，所有权从 s1 转移到 s2，
s1 变量不再有效。因此 `println!("{}", s1)` 会报编译错误。...

---

# 学习会话：rust - 第2章 借用（Borrowing）

...（后续会话追加在此）
```

---

## 工作流程

### 完整学习循环（推荐方式）

#### 步骤 1：启动学习

```bash
huaqi study rust
```

输出：讲解 + 考题

#### 步骤 2：回答问题

在对话中回答考题（通过 `huaqi chat` 或继续 CLI）

#### 步骤 3：查看反馈

Agent 会根据你的回答生成 100-150 字的反馈：
- 肯定正确部分
- 指出错误/补充遗漏
- 鼓励继续学习

#### 步骤 4：标记完成

输入「完成本章」或「第一章完成」→ 自动推进到第 2 章

#### 步骤 5：复习提醒

每晚 21:00，Scheduler 自动生成复习题（最多 2 门课程）：

```bash
huaqi daemon start  # 启动后台任务
```

输出（21:00 自动打印）：
```
[LearningPush] 📚 Rust 每日复习题：
以下代码的输出是什么？
fn main() {
    let x = 5;
    let x = x + 1;
    println!("{}", x);
}
```

---

### 三种使用方式对比

| 使用方式 | 命令 | 优点 | 缺点 |
|---------|------|------|------|
| **纯 CLI** | `huaqi study rust` | 快速启动，无需对话 | 无法追问，反馈有限 |
| **Agent 对话** | `huaqi chat` 后输入学习相关问题 | 可追问，反馈详细 | 需要维持对话会话 |
| **混合模式** | CLI 启动 + 对话提问 | 兼具两者优点 | 操作稍复杂 |

---

## 高级功能

### 1. 自动化定时推送

**前提：** 已启动 Scheduler

```bash
huaqi daemon start
```

**配置：** 每晚 21:00 自动：
- 查询所有「进行中」的课程
- 选择最多 2 门课程
- 为当前章节生成复习题
- 打印到终端（可通过日志重定向保存）

**查看推送历史：**
```bash
tail -f ~/.huaqi/logs/scheduler.log  # 需自行配置日志文件
```

---

### 2. 自定义课程

若要跳过自动大纲生成，手动创建：

**创建 YAML 文件：** `~/.huaqi/memory/learning/courses/custom/outline.yaml`

```yaml
skill_name: 我的自定义课程
slug: custom
created_at: '2026-03-31T00:00:00'
current_lesson: 1
lessons:
  - index: 1
    title: 第一节
    status: pending
    completed_at: null
  - index: 2
    title: 第二节
    status: pending
    completed_at: null
```

之后可通过 `huaqi study custom` 启动学习。

---

### 3. 技能名称规范化

系统自动将技能名称转换为 slug（URL 友好格式）：

```python
slugify("Rust")        # → "rust"
slugify("Python 3")    # → "python-3"
slugify("C++")         # → "c"
slugify("Golang")      # → "golang"
```

因此这些输入等价：
```bash
huaqi study Rust
huaqi study RUST
huaqi study rust        # 都会加载同一个课程
```

---

### 4. 数据迁移

若更换数据目录，所有课程和会话自动迁移：

```bash
huaqi config set data_dir /新/路径
# 系统自动复制 ~/.huaqi 到 /新/路径
```

---

### 5. 与其他功能集成

**与日记系统的协同：**
- 在日记中提及「学了 Rust」→ Agent 自动关联课程进度

**与技能追踪的协同：**
- 完成课程章节 → 自动更新技能等级（需手动 `/log` 记录）

**与目标管理的协同：**
- 为学习设定长期目标：`/goal 3个月掌握 Rust`
- Agent 定期提醒学习进度

---

## 核心文件位置

### 源代码

| 文件 | 功能 |
|------|------|
| `/huaqi_src/learning/models.py` | 数据模型（LessonOutline、CourseOutline） |
| `/huaqi_src/learning/progress_store.py` | YAML 持久化、会话记录 |
| `/huaqi_src/learning/course_generator.py` | LLM 生成（大纲、讲解、考题、反馈） |
| `/huaqi_src/learning/learning_tools.py` | 3 个 Agent 工具 |
| `/huaqi_src/cli/commands/study.py` | CLI 命令实现 |

### 测试

| 文件 | 测试用例数 |
|------|----------|
| `/tests/learning/test_models.py` | 5 |
| `/tests/learning/test_progress_store.py` | 7 |
| `/tests/learning/test_learning_tools.py` | 3 |
| `/tests/cli/test_study_cli.py` | 3 |
| `/tests/scheduler/test_learning_push.py` | 2 |

---

## 常见问题 (FAQ)

### Q1: 如何离线使用？

**A:** 需要 LLM API 才能生成讲解和考题。生成后的讲解、考题、反馈可离线查看。

### Q2: 能否改变每章内容？

**A:** 目前无编辑 UI，但可直接编辑 `outline.yaml` 修改章节标题（重启后生效）。

### Q3: 如何导出学习记录？

**A:** 所有会话记录已保存为 Markdown，位置：
```
~/.huaqi/memory/learning/sessions/
```
可直接复制或转换为 PDF。

### Q4: 能否同时学多个课程？

**A:** 完全支持。每门课程独立大纲和进度，`huaqi study --list` 可同时显示。

### Q5: 反馈生成有问题怎么办？

**A:** 反馈由 LLM 生成，质量取决于模型。若不满意，可：
1. 切换 LLM 提供商：`huaqi config set llm`
2. 手动补充反馈（编辑 Markdown 会话文件）

---

## 相关文档

| 文档 | 位置 | 说明 |
|------|------|------|
| 功能设计 | `/docs/features/learning-assistant.md` | 完整功能规范 |
| 实现计划 | `/docs/plans/2026-03-31-learning-assistant.md` | 开发细节 |
| 单元测试 | `/tests/learning/` | 28 个测试用例 |
| API 文档 | `huaqi_src/learning/__init__.py` | 导出的公共接口 |

---

**版本：** v1.0  
**最后更新：** 2026-03-31  
**维护者：** Huaqi 项目组
