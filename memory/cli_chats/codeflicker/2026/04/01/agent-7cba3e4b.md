---
session_id: agent-7cba3e4b
date: 2026-04-01
time_start: 2026-04-01T03:46:37.408Z
time_end: 2026-04-01T03:58:39.110Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-04-01 11:46:37] [user]: 请对 /Users/lianzimeng/workspace/xp 项目进行非常彻底的探索分析，我需要了解：

1. 项目整体目录结构（所有文件和目录）
2. 每个模块的职责和实现细节（读取所有源码文件）
3. 模块间的依赖关系和调用关系
4. 数据流转过程（数据如何在各模块间流动）
5. 外部依赖和接口
6. 配置文件内容
7. 入口文件和主要命令

请读取所有关键文件的完整内容，包括：
- package.json / cargo.toml 等依赖配置
- 所有源码文件（src/ 目录下所有文件）
- README 等文档
- 配置文件

返回尽可能详细的信息，包括每个文件的核心逻辑、函数签名、数据结构定义等。
[2026-04-01 11:46:40] [assistant]: 我来为你进行非常彻底的项目探索分析。让我从多个维度逐步深入了解这个项目。
[2026-04-01 11:46:46] [assistant]: 现在让我探索项目配置和源码结构：
[2026-04-01 11:46:52] [assistant]: 现在让我读取所有源码文件：
[2026-04-01 11:46:56] [assistant]: 现在读取 CLI、服务器和知识管理相关文件：
[2026-04-01 11:47:00] [assistant]: 继续读取后端和配置相关文件：
[2026-04-01 11:47:09] [assistant]: 现在让我读取云服务商和其他关键文件：
[2026-04-01 11:47:16] [assistant]: 现在让我检查测试文件和后端实现：
[2026-04-01 11:47:24] [assistant]: 最后让我检查 PostgreSQL 后端实现：
[2026-04-01 11:47:32] [assistant]: 现在让我检查文档设计文件来完整理解项目规划：
[2026-04-01 11:47:40] [assistant]: 现在让我生成一份完整的项目分析报告。让我创建一个详细的总结：
[2026-04-01 11:57:48] [assistant]: 现在让我创建一份完整的项目分析文档：
[2026-04-01 11:57:52] [assistant]: 让我用另一种方法创建分析报告：
[2026-04-01 11:58:39] [assistant]: 现在让我为你创建一份详细的项目分析总结：

## 项目详细分析总结

我已完成对 XP 项目的非常彻底的分析。以下是完整的结构和技术细节：

---

## **一、项目核心概览**

### **定位与价值**
XP 是一个 **AI Agent 经验沉淀与复用工具**，通过 MCP 协议集成到 IDE（Cursor/Claude），自动沉淀任务经验，下次遇到相似问题时直接检索复用。

**核心价值链**:
```
Agent 完成任务 → 自动提取经验 → 人工审核 → 智能检索注入 → 下次直接复用
```

---

## **二、完整项目结构**

### **源码统计**
- **总代码量**: 4,321 行 Python
- **14 个核心模块**
- **6 个关键数据文件**（JSON + SQLite）

### **模块职责清单**

| 模块 | 行数 | 核心职责 |
|------|------|---------|
| **knowledge.py** | 1176 | 经验提取、检索、反馈、质量分析 |
| **storage.py** | 646 | 三层存储（经验JSON+向量SQLite+指标SQLite） |
| **cloud_providers.py** | 505 | Supabase/Weaviate/Elasticsearch 云同步 |
| **cli.py** | 714 | review/add/edit/stats/analyze CLI 命令 |
| **server.py** | 458 | MCP Server，4+3 个工具导出 |
| **backends/postgres.py** | 298 | PostgreSQL 异步后端（Phase 3） |
| **models.py** | 110 | 5 个数据类（Experience/Session/Feedback 等） |
| **embeddings.py** | 86 | BGE-small-zh + OpenAI Embedding Provider |
| **其他** | 328 | file_watcher、project_config、cloud_providers 等 |

---

## **三、核心数据模型**

### **Experience（经验）**
```python
Experience:
├── id: UUID
├── type: ExperienceType (bugfix/feature/pattern)
├── level: ExperienceLevel (L1 架构/L2 场景/L3 修复)
├── title: str (自动生成或 LLM 生成)
├── problem: str (问题描述)
├── solution: str (解决方案，20+ 字要求)
├── key_decisions: str (关键决策/踩坑点，10+ 字要求，质量关键)
├── confidence: float (0.0-1.0，动态调整，< 0.1 自动归档)
├── status: ExperienceStatus (pending/active/archived)
├── source: ExperienceSource (agent/manual)
├── metadata: ExperienceMetadata
│   ├── tech_stack: ["react", "typescript"] (自动推断)
│   ├── problem_type: str
│   └── scene: ["表单", "异步"] (40+ 个预定义标签，自动推断)
├── tags: list (向后兼容)
├── related_files: list (Phase 3：用于失效检测)
├── file_hashes: dict (Phase 3：SHA256 前 16 位)
├── project: str (Phase 3：项目隔离)
├── last_hit_at: str (Phase 3：TTL 管理)
└── stale_reason: str (Phase 3：失效原因)
```

### **Session（会话数据）**
```python
Session:
├── session_id: str (会话唯一标识)
├── task_description: str
├── experience_ids_injected: list (注入的经验 ID)
├── iteration_count: int (对话轮数)
├── had_error_correction: bool
├── user_accepted: bool
├── ab_test_group: str ("control"/10% 或 "treatment"/90%)
└── ab_test_result_shown: bool
```

---

## **四、关键技术方案**

### **4.1 纯向量检索（核心创新）**

**Embedding 方案**:
- 模型: **BGE-small-zh-v1.5**（推荐）
- 维度: 384 维
- 本地推理: 首次下载 ~90MB，后续离线可用
- 归一化: 支持快速相似度计算

**检索流程**:
```
1. 标签过滤 (metadata.tech_stack, scene, problem_type)
2. Embedding 计算 (query, 所有 active 经验)
3. 余弦相似度排序
4. 阈值过滤 (>= 0.5)
5. 返回 TopK
```

**存储优化**:
- 向量压缩: `struct.pack(f"{len(vector)}f", *vector)` → binary blob
- 384 维向量 = 1.5KB
- 10 万条经验 = 150MB（SQLite）

### **4.2 质量评分系统**

**评分权重**:
```
key_decisions >= 50 字:      40 分
solution_summary >= 50 字:    20 分
related_files 非空:          15 分
tech_stack 推断成功:          15 分
重复距离 <= 0.6:             10 分
──────────────────────────────
总分 >= 80:  自动 active (confidence 0.65)
总分 40-79:  pending (人工 review)
总分 < 40:   拒绝
```

### **4.3 Confidence 动态更新机制**

```
每次反馈后:
new_confidence = old_confidence × 0.9 + adoption_rate × 0.1

若 confidence < 0.1:
  → 自动归档（停止参与检索）
```

### **4.4 A/B 测试框架**

```
根据 session_id hash 分组:
hash(session_id) % 10 == 0  → 对照组（10%，无经验注入）
其他                        → 实验组（90%，注入经验）

效果对比:
                有注入   无注入
平均轮数        3.2      5.7
报错率         18%      41%
接受率         84%      67%
```

### **4.5 自动推断系统**

**技术栈推断** (10+ 种):
- react/vue/typescript/python/docker/git 等
- 通过关键词匹配自动识别

**场景推断** (40+ 种):
- 前端: 表单、列表、异步、状态管理、路由、性能优化等
- 后端: 数据库、API、认证、错误处理、日志等
- 工程化: 依赖管理、构建、类型系统、代码规范等
- 运维: 容器化、K8s、云服务、CI/CD 等

---

## **五、数据流转过程**

### **完整生命周期**

```
┌─ 任务开始
│  ├─ Agent 调用 search_best_practices(query)
│  ├─ 系统返回历史经验（相似度 >= 0.5）
│  └─ Agent 参考历史经验执行任务
│
├─ 任务执行
│  └─ Agent 完成代码实现
│
├─ 任务完成
│  ├─ Agent 调用 extract_experience(...)
│  ├─ 系统验证字段长度
│  ├─ 系统检查重复（相似度 >= 0.85 则拒绝）
│  ├─ 系统评分（40-100）
│  ├─ 系统自动推断技术栈、场景、类型
│  ├─ 系统生成 Embedding 向量并存储
│  ├─ 若评分 >= 80，自动 active (confidence 0.65)
│  ├─ 否则标记 pending，等人工 review
│  └─ 返回 Experience 对象
│
├─ 人工审核
│  ├─ 工程师运行 xp review
│  ├─ 逐条显示 pending 经验详情
│  └─ 选择操作:
│      [y] 确认 → active, confidence 0.8
│      [e] 编辑后确认 → 打开编辑器，confidence 1.0
│      [n] 拒绝 → archived
│      [s] 跳过
│
├─ 知识库可用
│  └─ active 经验参与检索
│
└─ 反馈记录
   ├─ Agent 调用 record_feedback(exp_id, adopted=true/false)
   ├─ 系统更新 experience_stats (hit_count, adopted_count, adoption_rate)
   ├─ 系统更新 confidence
   └─ 多次低反馈 → 自动归档
```

### **数据库操作**

**写入**:
```
extract_experience()
├─ ExperienceStore.add() → ~/.xp/knowledge.json
├─ VectorStore.save_vector() → metrics.db[experience_vectors]
└─ (首次 review 时记录)

record_feedback()
├─ MetricsStore.record_feedback() → metrics.db[feedback]
└─ MetricsStore.update_stats() → metrics.db[experience_stats]
```

**读取**:
```
search()
├─ ExperienceStore.list_active() → 从 JSON 过滤
├─ VectorStore.get_all_vectors() → 从 SQLite 加载
├─ NumPy cosine_similarity() → 本地计算
└─ MetricsStore.record_search() → 记录搜索事件
```

---

## **六、三层存储架构**

### **Layer 1: 经验数据**
- **位置**: `~/.xp/knowledge.json`
- **格式**: JSON 结构化（Experience 对象列表）
- **操作类**: `ExperienceStore`
- **方法**: add/get/update/delete/list_by_status

### **Layer 2: 向量索引**
- **位置**: `~/.xp/metrics.db[experience_vectors]`
- **格式**: SQLite BLOB（二进制压缩）
- **操作类**: `VectorStore`
- **方法**: save_vector/get_vector/get_all_vectors/delete_vector

### **Layer 3: 指标和事件**
- **位置**: `~/.xp/metrics.db`
- **表结构**:
  ```sql
  sessions              -- 会话记录（含 A/B 分组）
  search_events        -- 搜索事件（查询+结果数）
  review_events        -- 审核事件（action+拒绝原因）
  feedback             -- 反馈记录（experience_id+adopted+reason）
  experience_stats     -- 经验统计（hit_count/adopted_count/adoption_rate）
  experience_vectors   -- 向量索引（BLOB）
  ```

---

## **七、MCP Server 与 CLI 接口**

### **MCP Tools（7 个）**

**Phase 1（已实现）**:
1. `search_best_practices(query, tags, top_k, session_id)` - 检索
2. `extract_experience(...)` - 提取
3. `record_session(...)` - 记录会话
4. `record_feedback(experience_id, adopted, reason)` - 记录反馈

**Phase 2（规划）**:
5. `finalize_task(...)` - 一键完成（合并 1-3）
6. `infer_adoption(...)` - 自动推断采纳

**其他**:
7. 各类 CLI 命令

### **CLI 命令（14 个）**

```bash
xp add                  # 交互式添加
xp review              # 逐条审核 pending
xp edit <ID>           # 编辑并重新激活
xp archive <ID>        # 归档
xp delete <ID>         # 永久删除
xp import <文件>       # Markdown 导入
xp stats               # 统计数据
xp stats --since 7d    # 近 7 天统计
xp analyze             # 质量分析报告（7 类 pattern）
xp project list        # 列出项目
xp project switch <名> # 切换项目
xp watch               # 文件监听（Phase 3）
xp sync up/down        # 云同步（Phase 3）
```

---

## **八、Phase 规划与当前进度**

### **Phase 1（MVP，已完成）**
- ✅ 经验提取、检索、反馈、会话记录
- ✅ 本地 JSON + SQLite 存储
- ✅ BGE Embedding 向量检索
- ✅ 基础 A/B 测试框架
- ✅ CLI 命令行工具

### **Phase 2（进行中）**
- ⏳ 质量评分细化
- ⏳ 自动激活策略（评分 >= 80）
- ⏳ xp analyze 质量分析（7 类 pattern）
- ⏳ Confidence 动态更新

### **Phase 3（计划中）**
- 📋 文件监听 + 失效检测
- 📋 PostgreSQL + pgvector 存储迁移
- 📋 云端同步（Supabase/Weaviate/Elasticsearch）
- 📋 权限管理（Admin/Writer 角色）
- 📋 多项目隔离支持
- 📋 FastAPI 中心化服务

---

## **九、关键设计原则**

1. **Agent 提取，人工把关** - 自动化降低负担，但保留最终质量控制
2. **纯向量检索** - 无需 BM25/全文搜索，支持语义理解
3. **去重保证** - 提取前检查重复，避免知识库噪音
4. **质量飞轮** - 反馈数据驱动 confidence 更新，库越用越好
5. **渐进式架构** - 本地 MVP → 质量优化 → 服务化协作

---

## **十、技术栈与依赖**

| 模块 | 技术 | 用途 |
|------|------|------|
| Embedding | sentence-transformers | BGE-small-zh-v1.5 向量化 |
| 向量相似度 | NumPy | 余弦相似度计算 |
| 存储 | SQLite | 向量索引和指标 |
| CLI | argparse | 命令行交互 |
| MCP | mcp-sdk | IDE 集成协议 |
| LLM | OpenAI API | 可选：生成标题 |
| 异步 | asyncio | 异步操作支持 |
| 云端 | Supabase/Weaviate/ES | Phase 3：多云支持 |

---

## **十一、核心创新与亮点**

1. **纯向量检索方案** - 无需复杂全文搜索，基于 Embedding 的语义理解
2. **智能质量评分** - 四维度评分，自动分级（拒绝/pending/auto-active）
3. **完整 A/B 测试框架** - 10% 对照组，严格效果对比
4. **Confidence 动态更新** - 反馈驱动，自动淘汰低质量经验
5. **自动推断系统** - 技术栈、场景标签自动识别（40+ 个标签）
6. **MCP 无缝集成** - IDE 原生支持，无需额外配置
7. **渐进式迁移** - 支持从本地 JSON 无缝迁移到 PostgreSQL + 云端

---

## **十二、数据卷估算**

```
小团队（10 人，6 个月）:
  经验数: 200-500 条
  数据量: ~10MB （knowledge.json 2MB + metrics.db 8MB）
  向量存储: ~750KB (500 × 384维 × 4字节)

中等团队（50 人，1 年）:
  经验数: 1000-3000 条
  数据量: ~50-150MB
  向量存储: ~1.5-4.5MB

大型团队（100+ 人，多项目）:
  推荐迁移至 PostgreSQL + pgvector
  支持百万级经验，检索 < 50ms
```

---

## **总结**

XP 是一个**设计精良的经验沉淀工具**，具有以下特点：

- **技术选型合理**: 本地 Embedding + SQLite，无依赖，开箱即用
- **流程完整闭环**: 提取 → 审核 → 检索 → 反馈 → 优化
- **质量保障机制**: 去重检测、质量评分、Confidence 动态调整
- **可观测性强**: 完整的统计指标、A/B 测试、质量分析报告
- **可扩展性好**: 三个阶段递进升级，支持从本地到云端的无缝迁移
- **实用性高**: 与主流 IDE（Cursor/Claude）原生集成，零学习成本

适合**任何使用 AI Agent 完成任务的团队**，特别是工程和产品团队积累最佳实践的场景。
