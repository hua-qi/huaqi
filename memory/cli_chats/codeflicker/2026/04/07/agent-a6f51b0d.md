---
session_id: agent-a6f51b0d
date: 2026-04-07
time_start: 2026-04-07T14:04:14.523Z
time_end: 2026-04-07T14:25:19.470Z
project: users-lianzimeng
git_branch: 
---

[2026-04-07 22:04:14] [user]: 用户想知道 codeflicker（当前这个 CLI 工具）的 code agent 是基于什么框架构建的。请查阅相关文档，找出其技术架构、使用的框架或底层实现方式，并返回详细信息。
[2026-04-07 22:05:47] [assistant]: 根据 codeflicker 官方文档，我已找到关于其 code agent 技术架构的详细信息：

## CodeFlicker Code Agent 技术架构

### 核心架构概览

CodeFlicker 的 code agent 是一个**企业级多代理协同框架**，采用以下架构层级：

| 层级 | 说明 |
|------|------|
| 用户界面层 | CLI/前端 UI，触发任务、查看执行状态 |
| 任务编排层 | 解析请求，生成任务树/计划 |
| 子代理执行层 | 执行具体子任务 |
| 模型提供者层 | 统一封装对不同大模型的调用 |
| 工具与执行层 | 文件读取、glob、grep、Bash 等内置工具 |
| 技能与插件层 | 模块化能力扩展 |
| 观测与运维层 | 日志、追踪、指标、审计 |

### 技术栈实现

**核心语言**：TypeScript / Node.js

**关键组件**：

1. **数据模型**
   - `Task`：任务
   - `Plan`：计划
   - `Subtask`：子任务
   - `Tool`：工具
   - `ModelProvider`：模型提供者

2. **子代理机制**
   - 内置子代理：`Explore`、`GeneralPurpose`、`UsageGuide` 等
   - 自定义子代理：通过 `AGENTS.md` 文件定义
   - 加载优先级：内置 > 插件 > 全局 `~/.codeflicker/agents/` > 项目级 `.codeflicker/agents/`（项目级最高）

3. **模型提供者**
   - 支持多种模型：wanqing/kat-coder、OpenAI、本地模型等
   - 统一接口封装，便于切换

4. **工具系统**
   - 读取/搜索/执行 Bash 命令
   - 文件读取、glob 匹配、grep 搜索
   - 统一中间件调度、超时管理、错误处理

### AGENTS.md 格式示例

```yaml
---
name: code-reviewer
description: Review code changes for security and style issues.
tools: read, glob, grep
model: wanqing/glm-4.7
forkContext: false
color: blue
---
You are an expert code reviewer...
```

### 核心特性

- **并行执行**：使用 Promise.all 并行执行多个子代理
- **Frontmatter 驱动**：子代理以 Markdown + YAML 格式定义
- **插件系统**：支持 Skills/Plugins 扩展
- **安全性**：审批模式、审计日志、敏感数据遮蔽
- **可观测性**：结构化日志、分布式追踪、指标监控

### 目录结构

```
src/
  core/
    engine.ts          # 任务引擎与编排
    planner.ts         # 计划/任务拆分
    subagent.ts        # 子代理抽象
    model-provider.ts # 模型提供者抽象
    tool-manager.ts    # 工具管理
  agents/
    builtin/           # 内置子代理
    custom/            # 自定义子代理
  plugins/             # 插件与技能
  loaders/
    agents-loader.ts   # AGENTS.md 加载器
  interfaces.ts        # 数据模型
```

---

**总结**：CodeFlicker 的 code agent 是基于 **TypeScript/Node.js** 构建的企业级多代理协同框架，核心采用**任务编排 + 子代理并行执行**的架构，通过统一的模型提供者抽象和工具系统实现能力扩展，并支持 AGENTS.md 声明式配置自定义子代理。
[2026-04-07 22:24:44] [user]: 用户想知道 codeflicker 内置了多少个子 agent（built-in sub-agents），请列出所有内置子 agent 的名称和描述。
[2026-04-07 22:25:19] [assistant]: 根据 codeflicker 官方文档，**内置子代理（built-in subagents）共有 3 个**：

## 内置子代理列表

| 子代理名称 | 描述 |
|-----------|------|
| **Explore（探索）** | 专注于探索代码库，快速查找文件、搜索代码，并回答关于项目结构的问题 |
| **GeneralPurpose（通用）** | 多功能代理，用于研究复杂问题或执行路径不明确的多步骤任务 |
| **UsageGuide（使用指南）** | 回答关于 CodeFlicker CLI 功能和用法的问题（帮助文档、用法查询等） |

---

## 使用示例

```
use Explore, 我想分析项目的目录结构
use Explore, 请列出根目录下的 packages 目录及其子包

use GeneralPurpose, 我需要分解一个不确定路径的任务并给出执行计划

use UsageGuide, CodeFlicker CLI 的插件要如何开发？
use UsageGuide, 如何导出会话历史？
```

---

## 加载优先级说明

| 优先级 | 来源 |
|--------|------|
| 1（最高） | 项目级代理 `.codeflicker/agents/` |
| 2 | 全局个人代理 `~/.codeflicker/agents/` |
| 3 | 插件 |
| 4（最低） | **内置代理** |

> 注意：内置代理虽然优先级最低，但在无冲突情况下是默认使用的核心代理。若有其他来源定义了同名代理，后续来源会覆盖内置配置。
