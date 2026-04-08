---
session_id: agent-cad49fec
date: 2026-03-31
time_start: 2026-03-31T13:45:25.961Z
time_end: 2026-03-31T13:46:58.820Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-03-31 21:45:25] [user]: 在 /Users/lianzimeng/workspace/huaqi-growing 项目中，找出所有用户数据模块（如 diary、people、world、learning、collectors 等），以及每个模块存储了什么数据、数据格式、存储路径。重点关注 huaqi_src/ 下各子目录的数据结构。返回一个清单，包含：模块名、数据类型、存储路径、数据格式。不需要写代码，只需要探索和总结。
[2026-03-31 21:45:33] [assistant]: 我将帮你探索项目中的所有用户数据模块，并整理成清单。让我从项目结构开始系统地分析。
[2026-03-31 21:45:38] [assistant]: 现在让我深入探索各个数据模块，查看具体的数据结构和存储实现：
[2026-03-31 21:45:42] [assistant]: 继续探索各个模块的数据存储：
[2026-03-31 21:45:45] [assistant]: 继续探索微信、日记和其他数据模块：
[2026-03-31 21:45:50] [assistant]: 现在让我查看其他关键数据模块：
[2026-03-31 21:45:55] [assistant]: 让我查看管道、报告、调度器等模块：
[2026-03-31 21:45:59] [assistant]: 现在让我查看.test_data目录以了解实际存储结构，以及人关系图谱模块：
[2026-03-31 21:46:04] [assistant]: 让我查看scheduler模块和一些存储相关的文件：
[2026-03-31 21:46:08] [assistant]: 现在让我检查一些示例文件和config_manager来了解更多存储细节：
[2026-03-31 21:46:12] [assistant]: 完美！现在我有足够的信息来编写综合清单。让我整理所有发现：
[2026-03-31 21:46:58] [assistant]: 现在我有足够的信息。让我创建一个详细的清单文档：

## 项目用户数据模块清单

基于对 `/Users/lianzimeng/workspace/huaqi-growing` 项目的全面探索，以下是所有用户数据模块的完整清单：

---

### 1. **日记模块 (Diary)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/core/diary_simple.py` |
| **数据类型** | DiaryEntry（日期、内容、心情、标签） |
| **存储路径** | `{data_dir}/memory/diary/YYYY-MM/YYYY-MM-DD[-suffix].md` |
| **存储格式** | Markdown 格式，包含 YAML Frontmatter |
| **关键字段** | date, content, mood, tags, created_at, updated_at |
| **特点** | - 按年月组织目录结构<br> - YAML Frontmatter 存储元数据<br> - 支持同日多篇日记（使用 suffix）<br> - 支持批量导入与搜索 |

---

### 2. **人关系图谱 (People Graph)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/people/graph.py` + `models.py` |
| **数据类型** | Person（人物信息）、Relation（关系） |
| **存储路径** | `{data_dir}/people/{person_name}.md` |
| **存储格式** | Markdown 格式，HTML 注释存储 JSON 数据 |
| **关键字段** | person_id, name, relation_type, alias[], profile, emotional_impact, interaction_frequency, notes, created_at, updated_at |
| **特点** | - 每个人物一个 Markdown 文件<br> - 支持别名列表<br> - 记录情感倾向和互动频率<br> - 关系强度可配置 |

---

### 3. **学习进度模块 (Learning)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/learning/progress_store.py` + `models.py` |
| **数据类型** | CourseOutline（课程大纲）、LessonOutline（课时）、学习会话 |
| **存储路径** | - 课程元数据：`{data_dir}/learning/courses/{slug}/outline.yaml`<br> - 学习会话：`{data_dir}/learning/sessions/{YYYYMMDD}_{skill_slug}.md` |
| **存储格式** | YAML（大纲）+ Markdown（会话记录） |
| **关键字段** | skill_name, slug, lessons[], current_lesson, status, completed_at, content, quiz, user_answer, feedback |
| **特点** | - 按技能组织课程<br> - 跟踪完成进度<br> - 保存每次学习会话详情<br> - 支持课程大纲 YAML 管理 |

---

### 4. **世界感知模块 (World News)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/world/storage.py` + `fetcher.py` |
| **数据类型** | HuaqiDocument（文档/新闻条目） |
| **存储路径** | `{data_dir}/world/{YYYY-MM-DD}.md` |
| **存储格式** | Markdown 格式，按日期组织 |
| **关键字段** | doc_id, doc_type, source, content, timestamp, summary, people[], metadata |
| **特点** | - 按日期存储世界热点摘要<br> - 支持 RSS 源采集<br> - 包含人物提及标记<br> - 支持内容搜索（7天内） |

---

### 5. **心理特征模块 (Personality)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/core/personality_simple.py` |
| **数据类型** | PersonalityProfile（心理特征档案） |
| **存储路径** | `{data_dir}/memory/personality.yaml` |
| **存储格式** | YAML |
| **关键字段** | name, tone, openness, conscientiousness, extraversion, agreeableness, neuroticism, values[], role, relationship, language_style, use_emoji, use_markdown, proactivity, challenge_user, custom_traits |
| **特点** | - 定义 AI 同伴的个性<br> - Big5 人格维度<br> - 支持预设个性（companion/mentor/friend/assistant）<br> - 自定义特征扩展 |

---

### 6. **成长追踪模块 (Growth Tracker)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/core/growth_simple.py` |
| **数据类型** | Skill（技能）、Goal（目标） |
| **存储路径** | `{data_dir}/memory/growth.yaml` |
| **存储格式** | YAML |
| **关键字段** | 技能：name, category, current_level, target_level, total_hours, last_practice, notes<br> 目标：id, title, description, status, progress, created_at |
| **特点** | - 统一的 YAML 存储<br> - 跟踪技能练习时间<br> - 目标进度管理<br> - 支持目标状态变更 |

---

### 7. **Hook 系统 (Triggers & Actions)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/core/hooks_simple.py` |
| **数据类型** | Hook（触发器定义） |
| **存储路径** | `{data_dir}/memory/hooks.yaml` |
| **存储格式** | YAML |
| **关键字段** | id, name, description, trigger_type (schedule/event/condition), trigger_config, actions[], enabled, last_triggered, trigger_count |
| **特点** | - 定义自动化流程<br> - 支持定时（cron）、事件、条件触发<br> - 可执行多个动作<br> - 默认内置晨间问候、每日总结等 |

---

### 8. **用户画像模块 (User Profile)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/core/profile_models.py` + `profile_manager.py` |
| **数据类型** | UserProfile（身份+偏好+背景） |
| **存储路径** | `{data_dir}/memory/profile.yaml` |
| **存储格式** | YAML |
| **关键字段** | identity (name, nickname, occupation, location, email)<br> preferences (communication_style, topics_of_interest)<br> background (education, skills, hobbies, life_goals, values)<br> extraction_history, version, created_at, updated_at |
| **特点** | - 从对话自动提取信息<br> - 版本控制与提取历史<br> - 支持画像摘要生成（用于 Prompt） |

---

### 9. **事件存储 (Event Storage)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/core/event.py` + `db_storage.py` |
| **数据类型** | Event（事件记录） |
| **存储路径** | `{data_dir}/events.db` (SQLite) |
| **存储格式** | SQLite3 数据库 |
| **关键字段** | id, timestamp, source, actor, content, context_id |
| **特点** | - 统一事件数据结构<br> - 自动红敏信息脱敏<br> - 支持事件搜索与检索<br> - 可用于事件流分析 |

---

### 10. **向量数据库 (Vector Memory)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/memory/vector/chroma_client.py` |
| **数据类型** | 向量化的文本数据（embeddings） |
| **存储路径** | `{data_dir}/vector_db/` (Chroma 持久化目录) |
| **存储格式** | Chroma 专有格式（SQLite + 向量索引） |
| **关键字段** | doc_id, content, metadata{}, embedding[] |
| **特点** | - 基于 Chroma 向量库<br> - 支持余弦相似度搜索<br> - 用于记忆检索<br> - 可自定义 embedding 模型 |

---

### 11. **Markdown 记忆存储 (Conversation Memory)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/memory/storage/markdown_store.py` |
| **数据类型** | 对话会话（对话轮次） |
| **存储路径** | `{data_dir}/memory/YYYY/MM/{timestamp}_{session_id}.md` |
| **存储格式** | Markdown + YAML Frontmatter |
| **关键字段** | session_id, created_at, turns (user_message, assistant_response, metadata) |
| **特点** | - 人类可读的会话历史<br> - 按日期月份组织<br> - 支持导出为 JSON/HTML/TXT<br> - 支持搜索与删除 |

---

### 12. **CLI 聊天采集 (CLI Chat Collector)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/collectors/cli_chat_parser.py` + `cli_chat_watcher.py` |
| **数据类型** | CLIChatMessage（CLI 聊天消息） |
| **存储路径** | `{data_dir}/memory/cli_chats/{YYYY-MM}/` |
| **存储格式** | Markdown（自动采集外部 CLI 工具的对话） |
| **关键字段** | role (user/assistant), content |
| **特点** | - 自动采集 Codeflicker/Claude 对话<br> - 支持 Markdown 和 JSON 两种格式<br> - 按月组织归档<br> - 监听文件变化自动同步 |

---

### 13. **微信数据采集 (WeChat)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/collectors/wechat_reader.py` + `wechat_state.py` |
| **数据类型** | WeChatMessage（微信消息） |
| **存储路径** | 原始数据：微信本地数据库<br> 同步状态：`{data_dir}/wechat/{state.json}` |
| **存储格式** | - 采集状态：JSON<br> - 消息数据：已封存（需要访问微信 SQLite 数据库）|
| **关键字段** | rowid, contact, timestamp, content, is_self |
| **特点** | **已封存**：因 macOS SIP 保护和 SQLCipher 加密，不再支持微信采集<br> - 保留技术参考<br> - 支持增量同步（last_rowid 跟踪） |

---

### 14. **内容流水线 (Content Pipeline)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/pipeline/core.py` + `models.py` |
| **数据类型** | ContentItem（内容条目）、PipelineResult（处理结果）|
| **存储路径** | 流水线处理结果存储在内存/临时目录，最终可发布到小红书等平台 |
| **存储格式** | Python 对象（通常序列化为 JSON） |
| **关键字段** | id, source, title, content, status (raw/summarized/translated/ready/published), summary, translated_content, xiaohongshu_content, xiaohongshu_tags, metadata, created_at, updated_at |
| **特点** | - 支持 X/Twitter、RSS 等来源<br> - 多阶段处理：摘要→翻译→格式化<br> - 支持小红书格式发布<br> - 跟踪处理状态 |

---

### 15. **报告模块 (Reports)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/reports/daily_report.py` + `weekly_report.py` + `quarterly_report.py` |
| **数据类型** | 生成的报告文本（日报、周报、季报） |
| **存储路径** | - 日报：`{data_dir}/reports/daily/YYYY-MM-DD.md`<br> - 周报：`{data_dir}/reports/weekly/YYYY-W{week}.md`<br> - 季报：`{data_dir}/reports/quarterly/YYYY-Q{quarter}.md` |
| **存储格式** | Markdown |
| **关键字段** | 报告内容（由 LLM 生成的摘要、亮点、建议等） |
| **特点** | - 定期自动生成（通过 Scheduler）<br> - 聚合日记、世界热点、关系网络数据<br> - 支持 LLM 生成复盘与建议<br> - 用于长期反思与成长追踪 |

---

### 16. **调度器数据库 (Scheduler)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/scheduler/manager.py` |
| **数据类型** | APScheduler 任务定义 |
| **存储路径** | `{data_dir}/scheduler.db` (SQLite) |
| **存储格式** | SQLite3 数据库（APScheduler 专有表结构） |
| **关键字段** | job_id, trigger_type, trigger_config, function_name, args, kwargs, enabled, next_run_time |
| **特点** | - 持久化定时任务<br> - 支持 Cron/Interval/Date 触发器<br> - 跟踪任务执行日志<br> - 支持任务重试和异常处理 |

---

### 17. **灵活存储层 (Flexible Store)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/core/flexible_store.py` |
| **数据类型** | ContentInteractionEvent（内容交互）、分析结果 |
| **存储路径** | - 交互数据：`{data_dir}/memory/content_interactions/`<br> - 分析结果：`{data_dir}/memory/analysis_results/` |
| **存储格式** | JSON |
| **关键字段** | event_id, content_id, source, title, url, topics[], actions[], user_rating, user_comment, timestamp |
| **特点** | - 跟踪用户与内容的交互<br> - 记录用户反馈（评分、评论）<br> - 支持动态扩展字段<br> - 用于个性化推荐 |

---

### 18. **配置管理 (Configuration)**

| 项目 | 详情 |
|------|------|
| **模块路径** | `huaqi_src/core/config_manager.py` + `config_simple.py` |
| **数据类型** | AppConfig（应用配置） |
| **存储路径** | - 本地配置：`{data_dir}/memory/config.yaml`<br> - 全局配置：`~/.huaqi/config.yaml` |
| **存储格式** | YAML |
| **关键字段** | version, data_dir, llm_default_provider, llm_providers{}, memory{}, interface_theme, interface_language, modules{}, custom{} |
| **特点** | - 支持多个 LLM 提供商配置<br> - 记忆系统参数（搜索算法、top_k）<br> - 模块开关管理<br> - 全局与本地配置分离 |

---

### 19. **待审核与草稿目录**

| 项目 | 详情 |
|------|------|
| **模块路径** | - 草稿：`{data_dir}/drafts/`<br> - 待审核：`{data_dir}/pending_reviews/` |
| **数据类型** | 未发布的内容草稿与待审核内容 |
| **存储路径** | 自定义文件组织 |
| **存储格式** | 通常为 Markdown 或其他文本格式 |
| **特点** | - 发布前的暂存区<br> - 人机协同审核<br> - 支持版本管理 |

---

### 20. **模型缓存目录**

| 项目 | 详情 |
|------|------|
| **存储路径** | `{data_dir}/models/` |
| **用途** | 缓存下载的 embedding 模型、LLM 权重等 |
| **格式** | 模型文件（如 ONNX、PyTorch、HuggingFace 格式） |

---

## 存储路径层级总览

```
{data_dir}/
├── memory/
│   ├── config.yaml                      # 应用配置
│   ├── personality.yaml                 # 心理特征档案
│   ├── growth.yaml                      # 成长目标与技能
│   ├── hooks.yaml                       # 自动化 Hook 定义
│   ├── profile.yaml                     # 用户画像
│   ├── diary/
│   │   └── YYYY-MM/
│   │       └── YYYY-MM-DD[-suffix].md  # 日记条目
│   ├── cli_chats/
│   │   └── YYYY-MM/
│   │       └── {timestamp}_{tool}.md   # CLI 聊天记录
│   ├── conversations/
│   │   └── YYYY/MM/
│   │       └── {timestamp}_{session_id}.md  # 对话记录
│   ├── content_interactions/
│   │   └── *.json                       # 内容交互事件
│   ├── analysis_results/
│   │   └── *.json                       # 分析结果
│   └── work_docs/                       # 工作文档
│
├── people/
│   └── {person_name}.md                # 人物关系图谱
│
├── world/
│   └── YYYY-MM-DD.md                   # 世界热点摘要
│
├── learning/
│   ├── courses/
│   │   └── {skill_slug}/
│   │       ├── outline.yaml            # 课程大纲
│   │       └── lessons/                # 课时内容
│   └── sessions/
│       └── YYYYMMDD_{skill_slug}.md    # 学习会话记录
│
├── reports/
│   ├── daily/
│   │   └── YYYY-MM-DD.md              # 日报
│   ├── weekly/
│   │   └── YYYY-W{week}.md            # 周报
│   └── quarterly/
│       └── YYYY-Q{quarter}.md         # 季报
│
├── drafts/                              # 草稿
├── pending_reviews/                     # 待审核
├── vector_db/                           # Chroma 向量库
├── models/                              # 模型缓存
│
├── events.db                            # 事件数据库（SQLite）
├── scheduler.db                         # 调度器数据库（SQLite）
└── wechat_db/                           # 微信数据库（已封存）
```

---

## 数据格式总结

| 格式 | 用途 | 模块示例 |
|------|------|--------|
| **Markdown** | 人类可读的长文本 | 日记、世界热点、对话记录、报告、人物档案 |
| **YAML** | 配置与结构化数据 | 配置、个性档案、成长数据、Hook 定义、学习大纲 |
| **JSON** | 结构化数据交互 | 事件、交互数据、分析结果 |
| **SQLite** | 关系型数据 | 事件存储、调度器任务 |
| **Chroma** | 向量数据库 | 向量化记忆检索 |

---

## 关键特点总结

1. **单用户设计**：所有数据以单个用户为单位，配置在 `{data_dir}` 下
2. **Git 友好**：核心数据采用 Markdown/YAML 格式，便于 Git 版本控制
3. **模块化存储**：不同功能独立的存储目录，低耦合
4. **自动采集**：支持从外部工具（CLI、天气、新闻源）自动采集数据
5. **LLM 驱动**：大量数据由 LLM 分析生成（总结、翻译、提取等）
6. **人机协同**：支持用户审核、反馈、修改 AI 生成内容
7. **长期追踪**：完整的时间序列数据，支持成长分析

---

这就是 huaqi-growing 项目的完整用户数据模块清单！每个模块都有明确的职责和存储方案。
