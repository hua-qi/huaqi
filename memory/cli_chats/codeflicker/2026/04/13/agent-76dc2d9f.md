---
session_id: agent-76dc2d9f
date: 2026-04-13
time_start: 2026-04-13T03:52:13.099Z
time_end: 2026-04-13T03:55:52.526Z
project: users-lianzimeng-workspace-xp-verify
git_branch: 
---

[2026-04-13 11:52:13] [user]: 请查阅 xp-local 的使用文档或功能说明，了解它是否支持查看全局（不限于某个 git 仓库）的已沉淀经验。请重点查找：1) search_experience 工具的参数说明 2) 是否有不需要 git_remote_url 的查询方式 3) 是否有管理经验的全局命令。返回相关文档内容或结论。
[2026-04-13 11:52:21] [assistant]: 我来帮你查找 xp-local 的相关文档。首先让我获取 codeflicker 的使用文档，查找 MCP 服务器和 xp-local 相关的功能说明。
[2026-04-13 11:52:48] [assistant]: 让我继续搜索更多关于 xp-local 工具功能和 search_experience 的文档：
[2026-04-13 11:53:22] [assistant]: 让我尝试更具体地搜索 xp-local 的工具和命令说明：
[2026-04-13 11:53:49] [assistant]: 让我继续搜索更多关于 xp-local 的功能信息：
[2026-04-13 11:54:33] [assistant]: 让我检查用户当前项目中的 xp-local 相关配置文件，以及查看是否有更多文档：
[2026-04-13 11:54:40] [assistant]: 找到了相关文档。让我查看这些设计文档中关于 search_experience 工具和全局查询功能的说明：
[2026-04-13 11:55:23] [assistant]: 让我查看 MCP server 的 main.py 中 search_experience 工具的具体实现和参数定义：
[2026-04-13 11:55:35] [assistant]: 让我查看 CLI 设计文档中关于全局查询的说明：
[2026-04-13 11:55:52] [assistant]: 根据我对 xp-local 文档的全面查阅，以下是针对你三个问题的结论：

---

## 结论汇总

### 1. search_experience 工具的参数说明

根据 `xp_local_mcp/main.py` 中的定义：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `problem` | `str` | 是 | 当前要解决的问题描述 |
| `task` | `str` | 是 | 用户原始任务指令 |
| `git_remote_url` | `str` | **是** | 当前仓库 remote URL |
| `tags` | `list[str]` | 否 | 技术栈标签，向量软过滤用 |

### 2. 是否有不需要 git_remote_url 的查询方式

**结论：没有。**

`git_remote_url` 是 MCP 工具 `search_experience` 的**必填参数**，设计文档明确说明：

> stdio 模式下 MCP server 进程独立启动，不感知工作目录。通过 tool description 明确要求 Agent 传入当前仓库的 `git_remote_url`，server 入口统一标准化后派生 `project_id`，后续逻辑只使用 `project_id`。

如果无法解析 `git_remote_url`（为空或格式无法识别），会直接返回错误，不执行后续逻辑。

### 3. 是否有管理经验的全局命令

**结论：有，但仅限 CLI 工具。**

根据 CLI 设计文档 (`docs/designs/2026-04-08-cli-design.md`)：

> `--project URL` 可选，不传则自动检测当前目录的 git remote url 作为过滤条件，**找不到则展示所有 project 的数据**。

可用的全局命令：
- `xp stats [--days N]` - 查看使用漏斗和 Top 经验，不传 `--project` 时显示所有项目
- `xp review` - 查看待处理项（冲突、archived、从未被采纳），不传 `--project` 时显示所有项目

---

## 设计背景说明

xp-local-mcp 的定位是**私域经验沉淀**，强调的是"项目特有"的最佳实践（项目特有的架构决策、踩坑、约束），而非通用的编程知识。因此：

- **MCP 工具层面**：search/save 都是基于 `project_id` 隔离的，必须指定仓库
- **CLI 工具层面**：提供了查看全局数据的能力（主要用于管理和 review）

如果你需要实现不限于某个 git 仓库的全局经验查询能力，需要对 MCP 工具进行定制化修改。
