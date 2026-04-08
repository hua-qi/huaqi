---
session_id: agent-55874ed7
date: 2026-04-06
time_start: 2026-04-06T14:30:37.241Z
time_end: 2026-04-06T14:35:27.096Z
project: users-lianzimeng-working-stargate
git_branch: master
---

[2026-04-06 22:30:37] [user]: 请深入探索以下两个模块的完整流程：

1. /Users/lianzimeng/working/stargate/stargate_finnace/ - 财务分析模块
2. /Users/lianzimeng/working/stargate/stargate_analytics/ - 数据分析模块

对每个模块，请详细说明：
- 目录结构（所有文件）
- 核心 workflow 文件的主要逻辑和步骤（读取关键文件内容）
- service 层的主要功能
- prompts 目录下的提示词文件列表和内容摘要
- API 路由如何触发这些模块（查看 stargate_api/routers/ 中相关文件）
- 数据流：用户请求 → 路由 → service → workflow → LLM → 响应
- 各个工作流之间的调用关系
- 关键的 events/steps 定义

请尽量读取核心文件内容，给出详细的流程说明。
[2026-04-06 22:30:40] [assistant]: 我会深入探索这两个模块的完整流程。让我从目录结构开始，然后逐步分析核心文件和数据流。
[2026-04-06 22:30:44] [assistant]: 现在让我深入读取核心文件来了解完整的工作流程。
[2026-04-06 22:30:47] [assistant]: 让我继续读取更多关键文件：
[2026-04-06 22:30:52] [assistant]: 现在让我查看API路由和事件定义：
[2026-04-06 22:30:59] [assistant]: 现在让我查看提示词和服务文件：
[2026-04-06 22:31:03] [assistant]: 现在让我进一步查看完整的数据分析工作流和关键服务：
[2026-04-06 22:31:08] [assistant]: 现在让我查看相关路由文件来了解数据流如何被触发：
[2026-04-06 22:31:14] [assistant]: 现在让我创建一个综合的文档来总结这两个模块的完整流程。让我创建一个todo列表来追踪任务：
[2026-04-06 22:32:22] [assistant]: 现在让我生成完整的分析文档：
[2026-04-06 22:32:25] [assistant]: 让我用另一种方式创建文档：
[2026-04-06 22:32:28] [assistant]: 现在让我直接创建一个更详细的文档：
[2026-04-06 22:34:07] [assistant]: 完美，现在让我生成最终的综合文档。由于篇幅限制，我会将其分成多个部分输出：
[2026-04-06 22:34:10] [assistant]: 让我用更安全的方式输出文档：
[2026-04-06 22:34:17] [assistant]: 让我直接通过文档输出来提供分析结果：
[2026-04-06 22:35:27] [assistant]: 现在让我生成最终的完整分析文档：

## 📊 Stargate 两大核心模块完整深度分析

我已经完成了对两个模块的深入探索。下面是详细的分析报告：

---

## 一、📂 财务分析模块 (stargate_finnace) - 完整架构

### 目录结构概览 (总计 3064 行代码)

```
stargate_finnace/
├── workflows/           # 工作流编排层 (14个工作流)
│   ├── fin_manager_workflow.py     ⭐ 主管理工作流 (627行)
│   ├── fin_mgmt_report_workflow.py (451行)
│   ├── fin_data_interp_workflow.py (225行)
│   ├── fin_ir_workflow.py          (222行)
│   ├── mcp_asset_workflow.py       (391行)
│   └── 其他工作流...
├── service/            # 业务逻辑服务层 (20+ 个服务)
│   ├── finance_route_service.py    ⭐ 意图路由服务 (关键)
│   ├── task_agent_service.py       ⭐ 参数提取服务 (关键)
│   ├── finance_service.py          (核心数据查询)
│   ├── accounting_aggregation_ledger_service.py
│   ├── income_statement_service.py
│   ├── cashflow_statement_service.py
│   └── 其他专业服务...
├── model/
│   └── fin_asset_schema.py         (资产数据模型)
└── config/
    └── finance_config.py           (配置管理)
```

### 🔄 完整的工作流流程 (FinManagerWorkflow - 627行)

**阶段 1: 初始化 [step: start]**
```
用户请求 (POST /route-send-message)
    ↓
存储请求信息:
  • username (用户名)
  • conversation_id (会话ID)
  • message_id (消息ID)
  • content (用户问题)
  • app_type (应用类型: FINANCE等)
    ↓
检查文件上传状态:
  • 有文件 → data_analysis_on = true
  • 无文件 → 检查会话历史状态
    ↓
触发异步事件: GenerateConversationNameEvent
```

**阶段 2: 特殊指令检查 [step: special]**
```
检查用户指令:
  • /repeat   → 重复上次查询
  • /NOTE     → 触发笔记生成工作流
  • /BOARD    → 触发数据分析工作流
  • 其他      → 进入路由器
```

**阶段 3: 意图识别和资产路由 [step: planner] ⭐ 核心**

```python
关键方法: FinanceRouteService.send_message_with_assets_multi()

1. 获取用户可用资产列表
   await FinanceRouteService.get_user_assets(app_type, username)
   返回: List[FinanceAssetSchema]
   
2. 构建资产上下文
   FinanceRouteService.get_assets_with_group(assets)
   → 按财务分类分组
   
3. 构建LLM提示词
   System Prompt:
     • 财务路由Agent的系统提示
     • 可用资产列表(JSON)
     • 用户角色(accounting/tax/bp)
     
   User Prompt:
     • 用户问题
     • 对话历史(从Redis, max 16轮)
     
4. 调用LLM (Agent: FINANCE_ROUTE_BY_ASSET)
   输出格式 (PlannerResponse):
   {
     "assets": [
       {"name": "IncomeStatement", "question": "用户重新表述的问题"}
     ],
     "entities": ["收入", "成本"],
     "tools": [],
     "response_to_user": null
   }
```

**阶段 4: 处理路由结果 [step: handler_planner_result]**

```
根据 PlannerResponse 判断:

Case A: 有 assets
  └→ [step: retrieve_asset_args]
      • 检查资产是否为MCP类型
      • 调用 TaskAgentService.get_assets_args()
      • 返回: DataQueryEvent(args, asset)

Case B: 有 tools (如data_analytics)
  └→ 调用子工作流 (DataAnalyticsWorkflow等)

Case C: 有 response_to_user
  └→ 直接返回 TaskAgentResultEvent
```

**阶段 5: 数据查询 [step: data_query] ⭐ 关键**

```python
async def data_query(ctx, ev: DataQueryEvent):
    
    # 1. 获取查询Agent和提示词
    agent = await agent_service.get_agent_by_agent_type(
        AgentType.FINANCE_DATA_QUERY
    )
    
    # 2. 构建消息列表
    messages = [
        ChatMessage(role=SYSTEM, content=system_prompt_formatted),
        ...历史消息...,
        ChatMessage(role=USER, content=user_prompt_formatted)
    ]
    
    # 3. args 包含的关键变量
    args = {
        'currDate': '2024-06-04',
        'processedUserQuestion': '查询2024年上半年的利润',
        'currQuarter': 'Q2',
        'asset': FinanceAssetSchema(...),
        'asset_fields': [...],  # 资产相关字段
        'ner_mapping': {...},    # NER实体映射
        'always_notes': [...]    # 始终笔记
    }
    
    # 4. 调用LLM
    llm_response = await llm.achat(messages)
    
    # 5. 解析 AIMultiFunctionResponse
    parser = PydanticOutputParser(AIMultiFunctionResponse)
    
    # 重试逻辑 (最多3次):
    for retry in range(3):
        try:
            ai_functions = parser.parse(llm_response)
            break
        except:
            if retry == 0:
                # 尝试修复JSON
                repaired = repair_json(llm_response)
                ai_functions = parser.parse(repaired)
            else:
                # 询问LLM重试
                messages.append(ChatMessage(role=ASSISTANT, content=llm_response))
                messages.append(ChatMessage(role=USER, content="修复失败，请重试"))
                llm_response = await llm.achat(messages)
    
    # 6. 为每个function发送 GenerateResultEvent
    for function in ai_functions.functions:
        ctx.send_event(GenerateResultEvent(
            function_name=function.function_name,
            parameters=function.parameters,
            asset=asset,
            ...
        ))
```

**AI函数响应结构**:
```python
class AIMultiFunctionResponse(BaseModel):
    functions: List[
        {
            'function_name': 'IncomeStatement',  # 查询类型
            'parameters': {
                'company': '快手',
                'period': '2024Q1',
                'dimensions': ['成本', '收入']
            }
        }
    ]

class FinanceQueryParameter(BaseModel):
    company: str              # 公司
    period: str              # 期间
    dimensions: List[str]    # 查询维度
    filters: Dict[str, Any]  # 过滤条件
```

**阶段 6: 后处理 [step: post_process]**

```python
# 调用后处理服务进行:
• 数据聚合 (如多个维度求和)
• 时间戳转换 (FinRiskReport)
• 枚举值转换 (字段值映射)
• 单位转换

# 构建卡片
card = ProductCard(
    card_type='IncomeStatement',
    card_content=parameters.model_dump_json(),  # 完整JSON
    display_position=DisplayPosition.IN_LAYER,
    title='利润表'
)

# 流式发送进度信息
await _send_step(ctx, 3, '输出查询结果', '输出文字结果及表格')
```

**阶段 7: 持久化 [step: persistent]**

```python
# 收集所有 TaskAgentResultEvent (可能多个)
results = ctx.collect_events(ev, [TaskAgentResultEvent] * card_count)

# 保存到数据库
message_id = await memory_service.save_response(
    agent_id=agent.id,
    request=request,
    processed_user_question=processed_user_question,  # 格式化的问题
    response_message=ChatMessage(...),  # LLM完整输出
    response_content=json.dumps(cards),  # 卡片JSON
    card_type=CardType.MIXED_CARD,
    citations=citations,
    username=username
)

# 返回 PersistenceCompletedEvent(persistence_id=message_id)
```

**阶段 8: 返回响应 [step: end]**

```python
return SendMessageResponse(
    messageId=message_id,
    cardContents=agent_result,  # 所有卡片
    cardType=CardType.MIXED_CARD,
    role=MessageRole.ASSISTANT,
    conversationId=conversation_id
)
```

### 🎯 关键Service详解

#### 1. FinanceRouteService (路由服务)

**职责**: 识别用户意图，选择合适的资产

**关键方法**:
```python
async def send_message_with_assets_multi(
    username: str,
    request: SendMessageRequest,
    assets: List[FinanceAssetSchema],
    agent_type: AgentType
) -> Tuple[PlannerResponse, ChatMessage, dict]
```

**提示词包含**:
- 可用资产列表(JSON) → 让LLM知道有哪些查询选项
- 用户角色(accounting/tax/bp) → 不同角色关注不同数据
- 对话历史(Redis) → 维持上下文连贯性

**路由历史管理** (Redis):
```
Key: route_history:{conversation_id}
Value: [
  {"role": "user", "content": "查询..."},
  {"role": "assistant", "content": "{...}"},
  ...
]
TTL: 24小时
Max轮数: 16轮 (超过则删除最早的)
重新发送: 删除最后2条消息
```

#### 2. TaskAgentService (参数提取)

**职责**: 为查询提取必要参数，进行NER实体映射

```python
async def get_assets_args(
    assets: List[FinanceAssetSchema],
    query: str,
    entities: List[str],  # 用户提到的实体
    rephrased_question: str
) -> Dict[str, Any]
```

**核心逻辑**:
1. **NER实体扩展**:
   - 对每个实体进行embedding向量检索
   - 从业务词库中召回相关的枚举字段
   - 返回: 实体→可选值的映射

2. **字段分组**:
   - 按 dictionary_code 分组
   - 为每个字段列出可选的枚举值

3. **参数组装**:
   - 返回完整的args字典供Agent使用

#### 3. FinanceService (数据查询)

**支持的查询类型** (20+):
- IncomeStatement (利润表)
- CashFlowStatement (现金流)
- AssetsLiabilities (资产负债表)
- AccountingAggregation (核算汇总)
- TrialBalance (试算平衡)
- ArAging/ApAging (应收/应付账龄)
- FaAsset (固定资产)
- 等...

### 📊 Agent映射表

| Agent Type | 功能描述 | 提示词 |
|-----------|--------|------|
| FINANCE_ROUTE_BY_ASSET | **意图识别** - 决定查询哪个资产 | 路由提示词 |
| FINANCE_DATA_QUERY | **参数解析** - 解析查询参数并执行 | Agent系统/用户提示 |
| ACCOUNTING_AGGREGATION_LEDGER | 查询财务核算汇总 | 专业提示词 |
| INCOME_STATEMENT | 查询财报利润表 | 专业提示词 |
| 等20+ | 其他财务查询 | 对应专业提示词 |

---

## 二、📂 数据分析模块 (stargate_analytics) - 通用引擎

### 目录结构 (总计 2193 行核心工作流)

```
stargate_analytics/
├── workflows/
│   └── data_analytics_workflow.py   ⭐ 主Agent工作流 (2193行)
├── service/
│   ├── code_sandbox_service.py      ⭐ 代码沙箱
│   ├── data_schema_service.py       (Schema管理)
│   ├── database_rag_service.py      (知识检索)
│   └── graph_service.py             (图数据库)
├── tools/                           ⭐ 工具实现 (6个)
│   ├── code_execute.py              (执行代码)
│   ├── get_table_details.py         (查询表结构)
│   ├── retrieve_knowledge.py        (召回知识)
│   ├── generate_dashboard.py        (生成看板)
│   └── 其他工具...
└── prompts/
    ├── agents/
    │   ├── main_agent_prompt.md     ⭐ 主Agent提示词
    │   ├── finmi_agent_prompt.md    (财务Agent)
    │   └── ...
    └── tools_definition.json        ⭐ 工具定义
```

### 🤖 核心工作流：DataAnalyticsWorkflow (2193行)

采用**单Agent + 多Tool循环** (ReAct模式)

**关键特点**:
- Single Agent: Qwen Coder 480B (多能力大模型)
- Multiple Tools: 6+个工具
- Loop: 最多30次迭代
- Context: 智能压缩 + 硬截断兜底

#### 初始化阶段 [step: start]

```python
1. 初始化请求上下文
   • username, conversation_id, app_type
   • compression_strategy (压缩策略选择)

2. 处理用户文件
   • 上传的Excel转存为SQL表
   • 构建file_map和file_schema

3. 初始化数据服务
   • DataSchemaService: 获取所有表结构
   • DatabaseRagService: 初始化RAG引擎
   • CodeSandboxService: 启动代码执行会话

4. 加载历史对话
   • 从agent_message_service获取历史
   • 处理Redis中的工具结果缓存

5. 初始化全局变量
   • depth = 0 (循环深度)
   • code_execute_history = [] (代码执行历史)
   • variable_registry = {} (变量注册表)
```

#### 特殊指令检查 [step: check_special_command]

```
• /NOTE   → 触发笔记生成工作流
• /BOARD  → 启用看板生成工具，进入main_agent_loop
• 其他    → 直接进入main_agent_loop
```

#### 主Agent循环 [step: main_agent_loop] ⭐ **核心**

```python
async def main_agent_loop(ctx, ev: MainAgentLoopEvent):
    depth = await ctx.get('depth', 0)
    
    while depth < 30:  # 最多30次循环
        depth += 1
        
        # ===== 思考阶段 (Reasoning) =====
        
        # 1. 收集上下文
        database_info = await ctx.get('database_info', {})
        slang_info = await ctx.get('slang_info', [])
        code_history = await ctx.get('code_execute_history', [])
        excel_info = await ctx.get('excel_info', {})
        user_behaviour = await ctx.get('user_behaviour', '')
        
        context_str = f"""
        【数据库信息】
        {json.dumps(database_info)}
        
        【业务术语】
        {json.dumps(slang_info)}
        
        【代码执行历史】
        {code_history}
        
        【Excel信息】
        {json.dumps(excel_info)}
        """
        
        # 2. 构建消息
        messages = [
            ChatMessage(role=SYSTEM, content="""
                你是资深数据分析师...
                # 行为准则 (ReAct)
                1. 制定单步目标
                2. 选择行动: 调用工具或输出最终答案
                3. 处理错误: 分析并纠正
                ...
                # 可用工具列表
                {tools_definition}
                
                # 输出格式
                <reason>分析现状，制定下一步</reason>
                <tool>
                  <name>tool_name</name>
                  <arguments><![CDATA[{...}]]></arguments>
                </tool>
                
                或
                
                <final_result>
                Markdown格式的最终结论
                <artifact_work>chart_id</artifact_work>
                </final_result>
            """),
            ...历史消息...,
            ChatMessage(role=USER, content=f"""
                {user_behaviour}
                
                用户问题: {ev.query}
                
                上下文:
                {context_str}
            """)
        ]
        
        # 3. 检查Token是否溢出
        token_count = len(tokenizer.encode(messages))
        if token_count > 131072:  # 阈值
            if compression_strategy == SMART_WITH_FALLBACK:
                messages = await compress_context(messages)
        
        # 4. 调用LLM
        response = await llm.achat(messages)  # 使用Qwen Coder
        llm_output = response.message.content
        
        # ===== 行动阶段 (Acting) =====
        
        # 5. 解析LLM输出 (XML)
        parser = StreamingXMLParser()
        reason = extract_xml_tag(llm_output, 'reason')
        tool_name = extract_xml_tag(llm_output, 'tool.name')
        tool_args = extract_xml_tag(llm_output, 'tool.arguments')
        final_answer = extract_xml_tag(llm_output, 'final_result')
        
        # 发送进度事件
        ctx.write_event_to_stream(ProgressEvent(
            thinking=ThinkingStepItem(
                msgTag=f"思考步骤{depth}",
                steps=[ThinkingDetail(message=reason)]
            )
        ))
        
        # 6. 判定最终答案
        if final_answer:
            return FinalAnswerEvent(
                content=final_answer,
                response_cards=await ctx.get('response_cards', [])
            )
        
        # 7. 执行工具
        if tool_name:
            tool_result = await execute_tool(
                tool_name=tool_name,
                args=json.loads(tool_args),
                ctx=ctx
            )
            
            # 追加消息历史
            ev.messages.append(ChatMessage(
                role=ASSISTANT,
                content=llm_output
            ))
            ev.messages.append(ChatMessage(
                role=USER,
                content=f"Tool {tool_name} 执行结果:\n{tool_result}"
            ))
            
            # 继续循环
            return MainAgentLoopEvent(query=ev.query, messages=ev.messages)
        
        else:
            return GiveUpEvent(error_message="无法解析工具调用")
    
    return GiveUpEvent(error_message="超过最大循环次数")
```

### 🛠️ 工具详解

#### Tool 1: code_execute (执行Python代码)

**调用格式**:
```xml
<tool>
  <name>code_execute</name>
  <arguments><![CDATA[
  {
    "purpose": "数据分析",
    "code": "import pandas as pd\ndf = pd.read_sql(...)\nprint(df.head())"
  }
  ]]></arguments>
</tool>
```

**执行流程**:
```python
1. 使用Jinja2模板渲染代码
   template = sandbox_prescript.py.jinja
   formatted_code = template.render(session_id=session_id, code=code)

2. 调用远程沙箱API
   url = KConfProperties.code_sandbox_client_config()['base_url']
   response = await http_client.post(url, code=formatted_code)

3. 解析执行结果 (CodeExecuteResult)
   {
     "variables": {"df": "DataFrame", "result": "float"},
     "debug_log": ["中间调试输出"],
     "user_log": ["用户看得到的输出"],
     "stdout": "标准输出",
     "stderr": "错误信息",
     "artifacts": [
       {
         "artifact_type": "html",  // 或 csv
         "artifact_file_id": "stargate_code_sandbox/xxx.html",
         "display_title": "销售趋势图"
       }
     ]
   }

4. 变量持久化
   在沙箱中:
   _saved = load_session('{session_id}')  # 加载之前的变量
   globals().update(_saved)              # 注入当前作用域
   ... 执行用户代码 ...
   _, _saved = save_session(locals().copy(), '{session_id}')
```

#### Tool 2: get_table_details (查询表结构)

**用途**: 获取SQL表或Excel的完整Schema

**返回格式**:
```json
{
  "table_name": "orders",
  "table_description": "订单表",
  "fields": [
    {
      "field_name": "order_id",
      "field_description": "订单ID",
      "field_type": "BIGINT",
      "enum_values": null,
      "is_partition_key": false
    },
    {
      "field_name": "status",
      "field_description": "订单状态",
      "field_type": "VARCHAR",
      "enum_values": ["待支付", "已支付", "已发货", "已收货"],
      "is_partition_key": false
    }
  ],
  "sample_data": [...]
}
```

**使用时机**: 仅在代码执行失败(字段错误)时调用

#### Tool 3: retrieve_knowledge (召回知识)

**功能**: 从数据库和知识库中检索相关的表和字段

**DatabaseRagService**:
```python
async def retrieve_relevant_tables_and_fields(
    query: str  # 用户问题
) -> Dict:
    # 1. 向量编码
    query_embedding = embedding_model.encode(query)
    
    # 2. Neo4j向量搜索
    similar_tables = neo4j_driver.search(query_embedding)
    
    # 3. Jina重排
    reranked = reranker.rerank(similar_tables)
    
    return {
        "relevant_tables": [...],
        "business_concepts": [...]
    }
```

#### Tool 4: generate_dashboard (生成看板)

**功能**: 生成可视化看板

**执行流程**:
1. Agent生成看板规划 (JSON)
2. generate_dashboard 工具处理
3. 在代码沙箱中使用 pyecharts 生成HTML
4. 返回 artifact_file_id

### 📝 主Agent提示词 (main_agent_prompt.md)

**核心内容**:
```markdown
# 角色与使命
你是互联网公司一名资深数据分析师
• 精通Python、SQL编程
• 擅长数据分析、可视化
• 能够撰写分析报告

# 行为准则 (ReAct循环)
1. 制定单步目标: 明确下一步的唯一、最小目标
2. 选择行动: 
   - 调用工具 (code_execute / get_table_details等)
   - 或输出最终答案
3. 处理错误: 分析错误原因并制定纠错计划
4. 保持简洁: 聚焦决策，避免冗余思考
5. 一图胜千言: 每个观点都应有可视化支撑

# 输出格式

## ReAct循环中
<reason>
分析现状，制定下一步目标。
选择哪个工具？为什么选这个？
</reason>
<tool>
  <name>tool_name</name>
  <arguments><![CDATA[
  {json参数}
  ]]></arguments>
</tool>

## 最终答案
<final_result>
## 分析结论

关键发现:
- ...

支撑数据:
<artifact_work>chart_id</artifact_work>
<data_file>csv_id</data_file>
</final_result>
```

---

## 三、🌐 API集成与数据流

### API 路由 (/route-send-message)

**位置**: `stargate_api/routers/v2/ai_chat_router_v2.py`

```python
@router.post("/route-send-message")
async def send_message_by_route_v2(request: SendMessageRequest):
    username = get_general_context().username
    app_type = get_general_context().app_type
    
    # 应用类型路由
    if app_type == AppType.FINANCE or app_type == AppType.FINANCE_IR:
        workflow = injector.get(FinManagerWorkflow)
    
    elif app_type in (AppType.DEEP_ANALYTICS, AppType.FIN_DEEP_ANALYTICS):
        workflow = injector.get(DataAnalyticsWorkflow)
    
    elif app_type == AppType.HR or app_type == AppType.HR_ANALYTICS:
        workflow = HRManagerV2Workflow()
    
    # ... 其他应用 ...
    
    # 执行工作流
    handler = workflow.run(
        username=username,
        request=request,
        user_files=user_files,
        app_type=app_type,
        compression_strategy=request.compression_strategy  # 仅数据分析模块
    )
    
    if request.stream:
        # 流式响应
        return StreamingResponse(
            content=event_stream_generator(handler),
            media_type="text/event-stream"
        )
    else:
        # 非流式响应
        result = await handler
        return SendMessageResponse(...)
```

### 📡 流式事件

```
StartEvent
    ↓
ProgressEvent (处理步骤/思考)
ProgressEvent
...
CardResponseEvent (卡片响应)
...
TaskAgentResultEvent (任务结果)
    ↓
PersistenceCompletedEvent (持久化完成)
    ↓
StopEvent (流结束)
```

---

## 四、🔗 工作流间的调用关系

```
FinManagerWorkflow (财务主流)
    │
    ├─→ FinanceKnowledgeAgentFlow    (知识问答)
    ├─→ WebQueryFlow                 (网络查询)
    ├─→ DataAnalyticsWorkflow ⭐    (数据分析)
    ├─→ FileAnalyticsWorkflow ⭐    (文件分析)
    ├─→ DeepResearchWorkflow        (深度研究)
    ├─→ McpAssetWorkflow            (MCP资产)
    └─→ GeneralNoteWorkflow         (笔记生成)

DataAnalyticsWorkflow (数据分析主流)
    │
    └─→ Tools: [code_execute, get_table_details, ...]
```

---

## 五、🎯 核心对比

| 维度 | 财务模块 | 数据分析模块 |
|------|--------|-----------|
| **工作流代码** | 627行 | 2193行 |
| **Agent数量** | 多个专门Agent | 单一通用Agent |
| **执行模式** | 链式 (路由→查询→返回) | 循环式 (ReAct) |
| **循环次数** | 1-2次 | 最多30次 |
| **工具数量** | 0 (调用服务) | 6+个工具 |
| **适用场景** | 结构化财务数据 | 通用数据分析 |
| **响应速度** | 快 (秒级) | 慢 (分钟级) |
| **Token管理** | 简单 | 智能压缩+硬截断 |

---

## 六、💾 关键数据结构

### Redis缓存
```
route_history:{conversation_id}
  → 财务路由历史 (max 16轮, TTL 24h)

main_agent_workflow:{session_id}:database_info
  → 数据库Schema缓存

main_agent_workflow:{session_id}:tool_result_before_ask_user
  → 工具结果缓存 (TTL 7days)
```

### 数据库持久化
```
message表:
  - message_id
  - conversation_id
  - agent_type
  - processed_user_question
  - response_message (LLM输出)
  - response_content (JSON卡片)
  - citations
  - username
```

---

这就是 Stargate 两大核心模块的完整流程分析！两个模块采用不同的设计：

✅ **财务模块**: 专门化 - 使用多个专业Agent，快速高效
✅ **数据分析模块**: 通用化 - 使用单一强大Agent + 多工具，适应性强
