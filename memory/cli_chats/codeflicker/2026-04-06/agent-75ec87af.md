---
session_id: agent-75ec87af
date: 2026-04-06
time_start: 2026-04-06T14:24:05.011Z
time_end: 2026-04-06T14:25:38.124Z
project: users-lianzimeng-working-stargate
git_branch: master
---

[2026-04-06 22:24:05] [user]: 请探索 /Users/lianzimeng/working/stargate 项目的整体架构，包括：
1. 顶层目录结构
2. 主要模块和功能划分
3. 技术栈（package.json, 主要依赖）
4. 核心代码文件（src 目录结构）
5. 配置文件
请返回详细的目录结构和架构描述。
[2026-04-06 22:24:07] [assistant]: 我来帮你探索这个项目的整体架构。让我从顶层开始逐步深入分析。
[2026-04-06 22:24:10] [assistant]: 好的，这是一个很大的项目。让我继续探索主要的配置文件和核心代码结构。
[2026-04-06 22:24:14] [assistant]: 现在让我进一步探索项目的核心结构和主要模块：
[2026-04-06 22:24:17] [assistant]: 现在让我查看更多关于主要模块和配置信息：
[2026-04-06 22:24:21] [assistant]: 现在让我查看一些主要模块的初始化文件和核心结构：
[2026-04-06 22:24:24] [assistant]: 现在让我获取一些主要的数据库和服务信息：
[2026-04-06 22:24:27] [assistant]: 现在让我查看一些具体的实现文件来理解架构：
[2026-04-06 22:24:31] [assistant]: 现在让我创建一个综合的架构报告：
[2026-04-06 22:25:38] [assistant]: 完美！现在我有足够的信息来生成详细的架构报告。让我整理所有信息：

## 项目架构探索完整报告

### 1. 顶层目录结构

```
/Users/lianzimeng/working/stargate/
├── 核心模块
│   ├── stargate_core/              # 核心框架和公共服务
│   ├── stargate_api/               # REST API 层（FastAPI）
│   ├── stargate_console/           # 控制台服务
│   ├── stargate_task/              # 任务调度服务
│   │
│   └── 业务模块
│       ├── stargate_analytics/     # 数据分析模块
│       ├── stargate_finnace/       # 财务分析模块
│       ├── stargate_hr/            # 人力资源模块
│       ├── stargate_general/       # 通用智能模块（深度研究）
│       ├── stargate_intelligence/  # 智能分析模块
│       ├── stargate_kursor/        # Kursor RAG 模块
│       ├── stargate_travel/        # 出差管理模块
│       └── stargate_wanxiang_search/  # 万向数据搜索模块
│
├── 配置与部署
│   ├── configs/                    # 环境配置文件
│   │   ├── config_development.json
│   │   ├── config_local.json
│   │   ├── config_staging.json
│   │   ├── config_preonline.json
│   │   └── config_production.json
│   ├── Dockerfile                  # Docker 镜像配置
│   ├── start.sh                    # 启动脚本
│   └── build/                      # 构建配置
│
├── 数据和文档
│   ├── sql/                        # 数据库脚本
│   │   └── stargate.sql
│   ├── docs/                       # 开发文档
│   │   └── development/
│   └── examples/                   # 示例代码
│
└── 测试
    ├── tests/                      # 单元测试目录
    ├── pytest.ini                  # Pytest 配置
    └── requirements.txt            # Python 依赖
```

---

### 2. 主要模块和功能划分

#### **核心层（stargate_core）**
```
stargate_core/
├── ai/                        # AI 和机器学习支持
│   ├── embedding/            # 嵌入模型（BGE）
│   ├── llm/                  # 大模型支持
│   │   ├── claude.py         # Claude LLM
│   │   ├── deepseek.py       # DeepSeek LLM
│   │   ├── llm_factory.py    # LLM 工厂模式
│   │   └── llm_proxy.py      # LLM 代理
│   ├── reranking/            # 重排序模型（BGE、Jina）
│   └── workflows/            # AI 工作流
│
├── core/                      # 核心功能模块
│   ├── client/               # 各类客户端（40+ 个）
│   │   ├── access/          # 访问控制客户端
│   │   ├── ai_knowledge_base/  # AI 知识库客户端
│   │   ├── crawling/        # 网页爬取客户端（Jina、Crawl4AI）
│   │   ├── finance/         # 财务数据客户端
│   │   ├── permission/      # 权限管理客户端
│   │   ├── search/          # 搜索客户端（SearXNG）
│   │   └── 其他集成...
│   ├── database/            # 数据库层
│   │   ├── mysql_connection.py
│   │   ├── sqlalchemy_module.py
│   │   ├── db_executor.py
│   │   └── orm/             # ORM 工具
│   ├── cache/               # 缓存管理
│   │   ├── async_redis_manager.py
│   │   └── redis_manager.py
│   └── es/                  # Elasticsearch 客户端
│
├── dao/                      # 数据访问对象层（30+ DAOs）
│   ├── message_dao.py
│   ├── conversation_dao.py
│   ├── agent_dao.py
│   ├── data_resource_dao.py
│   └── ...其他数据访问层
│
├── services/                 # 业务服务层（50+ services）
│   ├── message_service.py
│   ├── conversation_service.py
│   ├── ai_service.py
│   ├── agent_service.py
│   ├── rag_service.py
│   ├── code_execute_service.py
│   └── ...其他业务服务
│
├── models/                   # 数据模型
│   ├── entities/            # 数据库实体模型（30+ entities）
│   │   ├── message.py
│   │   ├── conversation.py
│   │   ├── agent.py
│   │   └── ...其他实体
│   ├── schemas/             # API 响应模式（80+ schemas）
│   │   ├── message_schema.py
│   │   ├── ai_schema.py
│   │   ├── card_schema.py
│   │   └── ...其他模式
│   └── convertor/           # 数据转换器
│
├── interceptor/             # 请求拦截器
│   ├── authorization_gateway_interceptor.py
│   ├── request_performance_interceptor.py
│   └── ...其他拦截器
│
├── interpreter/             # 代码执行器
│   └── jupyter/             # Jupyter 执行环境
│
├── deepresearch/            # 深度研究模块
│   ├── deepresearch.py      # 主模块
│   ├── model/               # 深度研究数据模型
│   ├── prompts/             # 提示词模板（20+ 文件）
│   ├── reason/              # 推理逻辑
│   ├── search/              # 搜索逻辑
│   └── read/                # 阅读理解
│
├── config/                  # 配置管理
│   ├── env_config.py        # 环境配置
│   ├── llm_config.py        # LLM 配置
│   ├── model_config.py      # 模型配置
│   └── kconf_properties.py  # Kconf 属性
│
├── constants/               # 常量定义
│   ├── constants.py
│   ├── llm_type.py
│   ├── hr_enums.py
│   └── ...其他常量
│
├── enums/                   # 枚举定义（20+ enums）
├── utils/                   # 工具函数（30+ 工具）
├── exception/               # 异常处理
└── mcp/                     # MCP 服务支持
```

#### **API 层（stargate_api）**
```
stargate_api/
├── api_server.py           # FastAPI 应用主入口
│   ├── 定时任务（APScheduler）
│   ├── 路由注册
│   ├── 全局异常处理
│   └── 生命周期管理
│
└── routers/                # 30+ 路由模块
    ├── ai_chat_router.py        # AI 聊天
    ├── message_router.py         # 消息管理
    ├── conversation_router.py    # 对话管理
    ├── agent_router.py           # 智能体
    ├── bot_router.py             # 机器人
    ├── file_router.py            # 文件操作
    ├── knowledge_base_document_router.py  # 知识库
    ├── note_router.py            # 笔记
    ├── batch_test_router.py      # 批量测试
    ├── task_router.py            # 任务管理
    ├── open/                     # 开放 API
    │   ├── file_router_open.py
    │   └── schema_generator_router_open.py
    └── v2/                       # API v2
        └── ai_chat_router_v2.py
```

#### **控制台服务（stargate_console）**
```
stargate_console/
├── service/                 # 业务服务
│   ├── data_resource/      # 数据资源管理
│   ├── data_asset/         # 数据资产管理（包括 protobuf）
│   └── 其他服务
│
├── consumer/               # Kafka 消费者
│   ├── async_kafka_consumer.py
│   ├── data_resource_update_consumer.py
│   └── dictionary_event_consumer.py
│
└── routers/                # 控制台路由
    ├── data_resource_router.py
    ├── data_asset_router.py
    └── open/               # 开放路由
```

---

### 3. 技术栈分析

#### **核心框架和库**
```
Web Framework:
  - FastAPI 0.115.5          # 异步 Web 框架
  - Uvicorn 0.32.0           # ASGI 应用服务器
  - Pydantic 2.11.7          # 数据验证和序列化

Database & ORM:
  - SQLAlchemy 2.0.41 (asyncio支持)
  - SQLModel 0.0.24          # SQL ORM（Pydantic + SQLAlchemy）
  - MySQL Connector 8.0.29
  - asyncmy 0.2.10           # 异步 MySQL 驱动
  - Neo4j 5.28.2             # 图数据库
  - DuckDB 1.3.2             # 分析数据库

AI/LLM Stack:
  - LLaMA Index Core 0.12.43 # 向量存储和 RAG 框架
  - LLaMA Index Workflows 1.3.0  # 工作流框架
  - OpenAI 1.99.6            # OpenAI API
  - Anthropic 0.49.0         # Claude API
  - LLaMA Index - Bedrock (多个版本)  # AWS Bedrock
  - LLaMA Index - OpenAI/Azure OpenAI  # 多个 LLM 支持
  - LLaMA Index - Perplexity 0.3.2
  - LLaMA Index - Groq 0.3.1

Embedding & Reranking:
  - LLaMA Index - Embeddings OpenAI 0.3.1
  - BGE Embeddings / Reranker  # 开源嵌入和重排
  - Jina Reranker            # Jina 重排服务

Vector & Search:
  - Elasticsearch (向量搜索)
  - LLaMA Index - Elasticsearch Vector Store 0.4.2
  - Chroma Vector Store
  - SearXNG (元搜索引擎)

Data Processing:
  - Pandas 2.3.0             # 数据分析
  - NumPy 2.2.6              # 数值计算
  - PyArrow 20.0.0           # 列式数据格式
  - DuckDB 1.3.2             # 分析数据库
  - OpenPyXL 3.1.2           # Excel 处理
  - PyExcel 0.7.3 + Plugins  # Excel 工具
  - Pillow 11.2.1            # 图片处理
  - Beautiful Soup 4.12.3    # HTML 解析
  - Markdownify 1.1.0        # HTML to Markdown

Distributed & Async:
  - APScheduler              # 定时任务调度
  - Redis (asyncio)          # 缓存和消息队列
  - Kafka (通过消费者)        # 消息队列
  - Aiobotocore 2.18.0       # 异步 AWS SDK
  - AIOHttp 3.11.18          # 异步 HTTP 客户端
  - Aiostream 0.7.1          # 异步流处理
  - Aiocache 0.12.3          # 异步缓存
  - AsyncIO (内置)           # 异步编程

Document Processing:
  - Unstructured 0.18.15     # 文档处理
  - PyDocX 0.9.10            # Docx 处理
  - Python-Docx 1.2.0        # Word 处理
  - PyPDFium2 5.0.0          # PDF 处理
  - PyPandoc 1.15            # Pandoc 集成
  - Chardet 5.2.0            # 字符集检测

Utilities:
  - Loguru 0.7.3             # 日志库
  - PyYAML 6.0.2             # YAML 解析
  - Jinja2 3.1.6             # 模板引擎
  - Injector 0.22.0          # 依赖注入
  - Markdown 3.9             # Markdown 处理
  - Transformers 4.50.0      # Hugging Face 模型
  - Tiktoken 0.11.0          # Token 计数
  - Tabulate 0.9.0           # 表格格式化
  - Cryptography             # 加密
  - PyPinyin 0.55.0          # 汉字转拼音
  - Boto3/Botocore 1.36.0    # AWS SDK

Testing:
  - Pytest 8.3.5             # 测试框架
  - Pytest-asyncio 0.25.3    # 异步测试支持
  - Pyfakefs 5.9.2           # 文件系统模拟

Observability:
  - Arize Phoenix 10.15.0    # LLM 监控
  - OpenInference Instrumentation 4.3.0

Other:
  - Marisa-Trie 1.2.1        # 字符串匹配
  - SQLGlot 26.25.3          # SQL 解析
  - Diskcache 5.6.3          # 磁盘缓存
  - Httpx 0.27.2             # HTTP 客户端
  - PyMinizip 0.2.6          # 压缩工具
  - PyZipper 0.3.6           # ZIP 工具
```

**重要的自定义依赖：**
- `kuaishou-ea-ai-chat-sdk==0.1.0` - 快手 AI Chat SDK
- `mcp==1.22.0` - MCP（模型上下文协议）
- `ks-ee-excel-tool~=1.0.0` - 快手 Excel 工具

---

### 4. 核心代码结构详解

#### **4.1 API 层启动过程**

```python
# stargate_api/api_server.py
app = FastAPI(...)

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动时初始化
    await initialize()
    # 设置定时任务
    scheduler.add_job(scheduled_dashboard_task, CronTrigger(...))
    scheduler.add_job(scheduled_refresh_cache_task, CronTrigger(...))
    scheduler.start()
    yield
    # 关闭时清理
    await dispose()
    scheduler.shutdown()

# 注册所有路由
app.include_router(ai_chat_router.router)
app.include_router(message_router.router)
app.include_router(conversation_router.router)
... # 30+ 个路由
```

**启动配置：**
- 默认 8 worker 进程（开发/测试环境 4 个）
- 异步循环（asyncio）
- 环境变量控制：`RUN_ENV` (development/staging/preonline/production)

#### **4.2 依赖注入框架**

```python
# stargate_core/core/__init__.py
class StargateModule(Module):
    # 单例注册
    @singleton
    def async_redis(self) -> Redis:
        return KConfDiscoverRedis('ihrDataFlashAi')
    
    @singleton
    def crawl_client(self) -> BaseCrawlClient:
        return JinaClient(cache)
    
    @singleton
    def async_neo4j_client(self) -> AsyncDriver:
        # Neo4j 图数据库连接
        ...

injector = Injector([StargateModule(), DatabaseModule(), ConsoleModule()])
```

**关键服务：**
- Redis（缓存、消息队列）
- MySQL（主数据存储）
- Elasticsearch（文本和向量搜索）
- Neo4j（知识图谱）
- AWS S3（文件存储）

#### **4.3 数据库模型分层**

```
Entity (ORM Model) 
    ↓
DAO (Data Access Object)
    ↓
Service (Business Logic)
    ↓
Schema (API Response)
    ↓
Router (HTTP Endpoint)
```

**例如消息模块：**
```
Message Entity → MessageDAO → MessageService → MessageSchema → message_router
```

#### **4.4 业务工作流架构**

每个业务模块都遵循类似的结构：

```
stargate_{业务}/
├── service/           # 业务服务实现
├── workflows/         # LLaMA Index 工作流
├── prompts/          # LLM 提示词
├── events/           # 事件定义
└── agent/            # 智能体定义
```

**工作流特点：**
- 基于 LLaMA Index Workflows（支持异步）
- 集成 LLM（Claude、DeepSeek、Bedrock）
- 支持多步骤编排和条件分支

---

### 5. 配置文件说明

#### **5.1 环境配置**

```json
// configs/config_development.json
{
  "logging": { /* 日志配置 */ },
  "openapi": { /* OpenAPI gateway */ },
  "server": { /* 服务器信息 */ },
  "wanxiang": { /* 万向数据配置 */ },
  "permission": { /* 权限管理配置 */ },
  "infra": { /* 基础设施配置 */ },
  "fulcrum": { /* Fulcrum 服务配置 */ },
  "blobstore": { /* 对象存储配置 */ },
  "flashdata": { /* Flash Data 配置 */ },
  "aikonwledgebase": { /* AI 知识库 ID 映射（30+） */ }
}
```

**多环境支持：**
- `config_local.json` - 本地开发
- `config_development.json` - 开发环境
- `config_staging.json` - 测试环境
- `config_preonline.json` - 预发环境
- `config_production.json` - 生产环境

#### **5.2 启动脚本 (start.sh)**

```bash
#!/usr/bin/env bash
# 配置日志目录
LOG_DIR="/home/web_server/kuaishou-runner/logs/$MY_POD_NAME"

# 获取端口（从 PORT_INFO.json 或默认 8000）
PORT=8000

# Worker 数根据环境决定（生产 16，测试 4）
WORKER_NUM=16

# 启动 FastAPI
uvicorn stargate_api.api_server:app \
  --host 0.0.0.0 \
  --port $PORT \
  --workers $WORKER_NUM \
  --loop=asyncio
```

#### **5.3 Docker 配置**

```dockerfile
FROM registry.corp.kuaishou.com/kks/cicd/kci/stargate-base-image:cid-bf3fa899-390305368

# 安装额外依赖
RUN pip install kuaishou-ea-ai-chat-sdk==0.1.0
RUN pip install mcp==1.22.0

# 复制应用代码
COPY . /app
WORKDIR /app

# 复制启动脚本
COPY ./start.sh /start.sh
RUN chmod +x /start.sh

ENV START_COMMAND "/start.sh"
CMD ["/usr/local/bin/kcsize", "api"]
```

---

### 6. 主要业务模块深度分析

#### **6.1 数据分析模块（stargate_analytics）**

```
功能：文件分析、数据库 Schema 分析、业务概念提取

核心工具：
  - Analyze User Requirements：用户需求分析
  - Data Process：数据处理
  - Generate Dashboard：仪表板生成
  - SQL 生成：自动 SQL 编写
  - Execute Code：代码执行（Jupyter）

RAG 系统：
  - 数据库 RAG（database_rag_service.py）
  - 图数据库（graph_database_init_service.py）
  - Schema 分析（data_schema_service.py）

Prompts（20+ 个提示词）：
  - file_analytics_prompt.md
  - schema_analytics.md
  - business_concept_analysis.md
  - 等等...
```

#### **6.2 人力资源模块（stargate_hr）**

```
核心功能：
  - 员工管理（emp_detail_workflow、emp_list_workflow）
  - 部门分析（dept_basic_info_workflow、dept_metrics_workflow）
  - 人才管理（talent_mobility_workflow）
  - 员工关怀（employee_care_workflow）
  - 报告生成（report_workflow）
  - HC 看板（hc_board_workflow）

关键服务：
  - hr_workflow_service.py / hr_workflow_v2_service.py
  - employee_care_service.py
  - report_service.py
  - user_preference_service.py
  - wanxiang_role_service.py（万向角色管理）

特色：
  - 支持偏好聚合（preference_aggregation_prompt.md）
  - 深度思考工作流（deep_think_workflow.py）
  - V2 版本更新（轻量级分析）

支持的视图：
  - 组织结构
  - 员工个档
  - 部门扫描（部门分析）
  - 人才盘点
  - Luopan（洛盘）数据
  - SSC（共享服务中心）视图
```

#### **6.3 财务分析模块（stargate_finnace）**

```
核心功能：
  - 财务报表分析（income_statement、balance_sheet、cashflow）
  - 成本分析（assets_liabilities、trial_balance）
  - 收入税务分析（revenue_and_tax_service）
  - 竞争对手分析（competitor_analysis）
  - 预算管理（finance_budget_split_manage_service）
  - 任务智能体（task_agent_service、biz_agent_service）

工作流：
  - fin_manager_workflow.py（财务经理工作流）
  - fin_mgmt_report_workflow.py/v2（管理报告）
  - accounting_task_agent_flow.py（会计任务流）
  - fin_data_interp_workflow.py（数据解释）

数据来源：
  - Web Query（web_query_service）
  - Knowledge Base（finance_knowledge_service）
  - MCP Asset（mcp_asset_workflow）

知识库：
  - 财务知识库 ID：f848d0b1-ef57-460d-b4db-21c0aa0a04cf
```

#### **6.4 通用智能模块（stargate_general）**

```
核心功能：
  - Deep Research（深度研究）v1 & v2
  - 一般对话（general_chat_workflow）
  - 笔记助手（general_note_workflow）
  - 数据解释（general_data_interp_workflow）

Deep Research 工作流步骤：
  1. Intent Detection（意图检测）
  2. Research（研究收集）
  3. Reason（推理分析）
  4. Report（生成报告）
  5. Filter & Cleanup（过滤清理）

特色：
  - 多轮对话支持
  - 上下文压缩（compress_context_workflow）
  - 图片过滤（image_filter.py）
  - 代码片段选择（snippet_selector.py）

提示词模板（20+ 个）：
  - intent_detection.md
  - researcher.md / researcher_v2.0.md
  - reasoner.md
  - reporter.md / reporter_v2.0.md
  - 等等...
```

#### **6.5 深度研究模块详解（stargate_core/deepresearch）**

```
架构：
  deepresearch.py（主协调器）
    ↓
  ├── Intent Detector（意图检测）
  ├── Researcher（研究者）
  ├── Reasoner（推理者）
  ├── Reporter（报告生成器）
  └── Filter & Cleanup（过滤清理）

特点：
  - 异步任务编排
  - 支持用户偏好个性化
  - 自适应查询重写
  - 多源信息融合

集成：
  - Web Search（SearXNG）
  - Web Crawling（Jina）
  - LLM 调用（Claude/DeepSeek）
  - Redis 缓存

使用场景：
  - 研究性问题解答
  - 信息综合分析
  - 长期上下文保存
```

---

### 7. 数据流和交互模式

#### **7.1 典型请求流程**

```
HTTP Request
    ↓
Router (FastAPI)
    ↓
Interceptor (授权、性能监控)
    ↓
Service (业务逻辑)
    ↓
├─ DAO (数据库查询)
├─ LLM Call (AI 推理)
├─ RAG (知识检索)
└─ External Clients (爬虫、搜索等)
    ↓
Schema (转换为 API 响应)
    ↓
HTTP Response
```

#### **7.2 异步工作流执行**

```
用户请求
    ↓
Workflow (LLaMA Index)
    ↓
┌─────────────────────────────────────┐
│ 多步骤并行/串行执行                  │
├─────────────────────────────────────┤
│ Step 1: Data Preparation            │
│ Step 2: LLM Analysis (并行搜索)      │
│ Step 3: Result Aggregation          │
│ Step 4: Post-processing             │
└─────────────────────────────────────┘
    ↓
Redis Cache (结果缓存)
    ↓
WebSocket/Streaming Response
```

#### **7.3 事件驱动流程**

```
Application Event (消息发送、数据变更)
    ↓
Event Publisher
    ↓
Kafka/Redis
    ↓
Consumer (async_kafka_consumer.py)
    ↓
处理器逻辑
    ↓
数据库更新 / Cache 更新
```

---

### 8. 性能优化特性

1. **异步支持：** 全异步 I/O，使用 asyncio
2. **缓存策略：** 
   - Redis（会话、缓存）
   - Elasticsearch（向量缓存）
   - 磁盘缓存（DiskCache）
3. **流式处理：** WebSocket 实时流式返回
4. **批量操作：** 批量测试、批量导出
5. **CDN 加速：** 文件存储到 S3
6. **负载均衡：** 16 worker 进程（生产）

---

### 9. 安全特性

1. **认证授权：**
   - KSAP Token 验证
   - OpenAPI Gateway 集成
   - 权限中心（AMC）

2. **加密：**
   - 消息加密存储（whether_encrypt 字段）
   - TLS/HTTPS
   - 凭证管理

3. **数据安全：**
   - SQL 注入防护（SQLAlchemy 参数化）
   - 敏感信息过滤
   - 审计日志

---

### 10. 监控与可观测性

1. **Logging：** Loguru（结构化日志）
2. **Tracing：** Arize Phoenix（LLM 调用追踪）
3. **Metrics：** 请求性能拦截器
4. **Dashboard：** StabilityDashboard（定时构建）
5. **Health Check：** ping_router（健康检查）

---

### 11. 关键文件索引

| 文件 | 位置 | 功能 |
|------|------|------|
| **启动入口** | stargate_api/api_server.py | FastAPI 应用初始化 |
| **依赖注入** | stargate_core/core/__init__.py | 所有服务单例注册 |
| **环境配置** | stargate_core/config/env_config.py | 多环境配置管理 |
| **核心服务** | stargate_core/services/ | 50+ 业务服务实现 |
| **数据访问** | stargate_core/dao/ | 30+ 数据访问对象 |
| **深度研究** | stargate_core/deepresearch/ | 高级研究工作流 |
| **HR 模块** | stargate_hr/workflows/ | 20+ HR 相关工作流 |
| **财务模块** | stargate_finnace/workflows/ | 18+ 财务工作流 |
| **API 路由** | stargate_api/routers/ | 30+ API 端点 |
| **数据库** | sql/stargate.sql | 数据库架构 |

---

### 12. 项目统计

- **顶级模块数：** 9 个（core、api、console、task + 5 业务模块）
- **代码文件总数：** 1000+ 个 Python 文件
- **Services 总数：** 50+ 个
- **DAOs 总数：** 30+ 个
- **数据实体：** 30+ 个
- **API Schemas：** 80+ 个
- **API 路由器：** 30+ 个
- **工作流：** 50+ 个
- **提示词模板：** 100+ 个
- **依赖库：** 92 个（requirements.txt）
- **测试套件：** tests 目录包含完整单元测试和集成测试

---

这就是 Stargate 项目的完整架构图。这是一个企业级的智能分析平台，集成了多个 AI 服务、复杂的数据处理流程和丰富的业务模块，具有高度的模块化和扩展性。
