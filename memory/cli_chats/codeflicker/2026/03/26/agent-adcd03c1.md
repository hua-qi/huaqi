---
session_id: agent-adcd03c1
date: 2026-03-26
time_start: 2026-03-26T13:46:27.480Z
time_end: 2026-03-26T13:47:46.217Z
project: users-lianzimeng-workspace-huaqi
git_branch: main
---

[2026-03-26 21:46:27] [user]: 请探索 /Users/lianzimeng/workspace/huaqi 目录下的项目结构，了解这是一个什么类型的项目，主要代码文件在哪里，以及 CLI 的入口点在哪里。我需要了解项目整体架构来查找输入处理相关的代码。
[2026-03-26 21:46:30] [assistant]: 我来帮你探索这个项目的结构。让我从多个角度进行分析。
[2026-03-26 21:46:35] [assistant]: 没有找到 TypeScript 文件，让我继续探索其他可能的代码文件类型：
[2026-03-26 21:46:39] [assistant]: 让我查看配置文件和检查是否有其他隐藏的项目文件：
[2026-03-26 21:46:58] [assistant]: 让我继续读取更多配置文件来了解这个工具：
[2026-03-26 21:47:09] [assistant]: 让我进一步检查这个项目的 git 远程仓库信息，以及是否是一个 CLI 工具的数据目录：
[2026-03-26 21:47:18] [assistant]: 发现了！这是一个 Python 包，源代码在 `/Users/lianzimeng/workspace/huaqi-growing`。让我探索源代码目录：
[2026-03-26 21:47:22] [assistant]: 现在让我查看 CLI 入口点和关键文件：
[2026-03-26 21:47:25] [assistant]: 现在让我查看核心模块中的输入处理相关代码：
[2026-03-26 21:47:46] [assistant]: 现在我已经全面了解了项目结构。以下是完整的项目分析：

---

## 项目概述

**Huaqi (花旗)** 是一个 **个人 AI 同伴系统**，用 Python 开发。

### 核心理念
- 不是"使用 AI"，而是"养育 AI"
- 让 AI 越来越像用户，越来越懂用户
- Human 3.0: 通过 AI 基础设施实现人类能力的大规模增强

---

## 项目结构

### 1. 数据目录 (`/Users/lianzimeng/workspace/huaqi`)
这是 **用户数据存储目录**，不是源代码目录：

```
/Users/lianzimeng/workspace/huaqi/
├── config/              # 全局配置
├── memory/              # AI 记忆系统
│   ├── config.yaml      # LLM 配置
│   ├── personality.yaml  # AI 个性配置
│   ├── hooks.yaml       # 自动化钩子
│   └── growth.yaml      # 成长追踪
├── users/               # 用户账户数据
└── users_data/          # 用户隔离数据
    └── {user_id}/       # 各用户的独立数据
```

### 2. 源代码目录 (`/Users/lianzimeng/workspace/huaqi-growing`)
这是 **实际的源代码仓库**：

```
/Users/lianzimeng/workspace/huaqi-growing/
├── cli.py               # ⭐ CLI 入口点
├── huaqi/               # 核心包
│   ├── __init__.py
│   ├── core/            # 核心模块
│   │   ├── config_simple.py     # 配置管理
│   │   ├── personality_simple.py # 个性引擎
│   │   ├── hooks_simple.py      # 钩子系统
│   │   ├── growth_simple.py     # 成长追踪
│   │   ├── llm.py               # ⭐ LLM 接口层
│   │   └── git_auto_commit.py   # Git 同步
│   └── memory/          # 记忆系统
│       ├── storage/     # 存储层
│       └── search/      # 搜索层
├── pyproject.toml       # 项目配置
└── requirements.txt
```

---

## CLI 入口点分析

### 主入口: `/Users/lianzimeng/workspace/huaqi-growing/cli.py`

```python
# pyproject.toml 中定义:
[project.scripts]
huaqi = "cli:app"
```

**关键函数:**

| 函数 | 行号 | 功能 |
|------|------|------|
| `main()` | 669 | 主回调，无子命令时进入对话模式 |
| `chat_mode()` | 407 | 交互式对话模式主循环 |
| `_prompt_input()` | 45 | **用户输入处理** - 使用 prompt_toolkit |
| `_handle_slash_command()` | 231 | 斜杠命令处理 |
| `_generate_response()` | 111 | 调用 LLM 生成回复 |

---

## 输入处理流程

### 1. 用户输入入口 (`cli.py:45-52`)
```python
def _prompt_input() -> str:
    """使用 prompt_toolkit 获取输入，支持多字节字符"""
    session = _get_prompt_session()
    return session.prompt()  # 用户输入，支持中文删除等操作
```

### 2. 主对话循环 (`cli.py:407-505`)
```python
def chat_mode():
    while True:
        user_input = _prompt_input().strip()
        
        # 处理不同类型的输入:
        if user_input.lower() in ("exit", "quit", "退出"): ...
        if user_input.lower() == "help": ...
        if user_input.lower().startswith("/"): ...  # 斜杠命令
        # 否则发送给 LLM
```

### 3. 斜杠命令处理 (`cli.py:231-290`)
支持的命令:
- `/skill <名称>` - 添加技能
- `/log <技能> <小时>` - 记录练习
- `/goal <标题>` - 添加目标
- `/status` - 查看状态
- `/skills`, `/goals`, `/help`

---

## 架构层次

```
┌─────────────────────────────────────────────────────────────────┐
│              交互层 (CLI) - cli.py                                │
│              使用 Typer + Rich + prompt_toolkit                  │
└─────────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────────┐
│              编排层 - ConversationManager                         │
│              PersonalityEngine + Hooks + Growth + Memory         │
└─────────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────────┐
│              核心层 - huaqi/core/                                 │
│              LLMManager + ConfigManager + Memory                 │
└─────────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────────┐
│              基础设施层                                           │
│              Markdown 存储 + BM25 搜索 + Git 同步                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 关键文件路径汇总

| 用途 | 文件路径 |
|------|----------|
| **CLI 入口** | `/Users/lianzimeng/workspace/huaqi-growing/cli.py` |
| **LLM 接口** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi/core/llm.py` |
| **配置管理** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi/core/config_simple.py` |
| **个性引擎** | `/Users/lianzimeng/workspace/huaqi-growing/huaqi/core/personality_simple.py` |
| **项目配置** | `/Users/lianzimeng/workspace/huaqi-growing/pyproject.toml` |
| **用户数据** | `/Users/lianzimeng/workspace/huaqi/` |
| **架构文档** | `/Users/lianzimeng/workspace/huaqi-growing/ARCHITECTURE.md` |

---

## 技术栈

- **语言**: Python 3.9+
- **CLI 框架**: Typer + Rich + prompt_toolkit
- **数据验证**: Pydantic
- **存储**: Markdown + YAML
- **搜索**: BM25
- **LLM**: OpenAI API 兼容 (OpenAI, Claude, DeepSeek)
