---
session_id: agent-093d67ad
date: 2026-03-30
time_start: 2026-03-30T13:47:26.092Z
time_end: 2026-03-30T13:51:59.262Z
project: users-lianzimeng-working-stargate
git_branch: master
---

[2026-03-30 21:47:26] [user]: 在 /Users/lianzimeng/working/stargate 项目中，详细分析 Excel 文件上传后生成看板（Dashboard）的完整流程：

1. 找 Excel/文件上传相关的 API 路由（搜索 upload, excel, file 相关路由）
2. 找文件处理服务（FileService, ExcelService 等）
3. 找 file_analytics_workflow.py 的完整逻辑（重点关注 Excel 解析、数据处理）
4. 找 Dashboard 生成相关代码（ECharts, dashboard, visualization 等）
5. 找从 Excel 数据 → SQL/DuckDB → 图表 的数据流
6. 找前后端交互的响应格式（流式响应的 XML/JSON 格式）

请返回：
- 完整的数据流程（从上传到展示）
- 关键代码片段（每个环节的核心代码）
- 涉及的所有关键文件路径
[2026-03-30 21:47:31] [assistant]: 我来帮你详细分析 Excel 文件上传后生成看板的完整流程。让我开始系统地搜索和分析。
[2026-03-30 21:47:34] [assistant]: 让我搜索 Excel 上传和文件处理相关的代码：
[2026-03-30 21:47:38] [assistant]: 很好！现在让我搜索 Dashboard 相关的代码：
[2026-03-30 21:47:41] [assistant]: 现在让我查看关键文件：
[2026-03-30 21:47:49] [assistant]: 现在让我查看 Dashboard 生成相关代码：
[2026-03-30 21:47:53] [assistant]: 现在让我查看数据流的关键部分 - SQL/DuckDB 相关代码：
[2026-03-30 21:47:56] [assistant]: 让我查看 CodeSandboxService：
[2026-03-30 21:48:04] [assistant]: 现在让我查看前后端交互的数据流和 API 路由：
[2026-03-30 21:48:07] [assistant]: 让我读取 file_service.py 的完整内容：
[2026-03-30 21:48:15] [assistant]: 让我查看 analyze_and_upload_file2blob 的具体实现：
[2026-03-30 21:48:21] [assistant]: 现在让我查看前后端响应格式相关的代码：
[2026-03-30 21:48:25] [assistant]: 让我查看 AI Chat Router 的响应格式：
[2026-03-30 21:48:28] [assistant]: 现在让我查看流式响应生成器：
[2026-03-30 21:48:36] [assistant]: 现在让我生成完整的分析文档。首先让我查看更多路由和生成文件相关的关键代码：
[2026-03-30 21:48:40] [assistant]: 现在让我查看具体的看板工作流：
[2026-03-30 21:50:23] [assistant]: 现在我有足够的信息来生成完整的分析文档。让我创建一份详细的分析报告：
[2026-03-30 21:50:30] [assistant]: 让我用另一种方式创建分析文档：
[2026-03-30 21:51:35] [assistant]: 现在让我生成一份详细的文本总结：
[2026-03-30 21:51:59] [assistant]: 完美！现在让我创建一个总结文档供用户参考：

## 完整分析总结

我已详细分析了 Stargate 项目中 **Excel 文件上传后生成看板（Dashboard）的完整流程**。以下是核心要点总结：

---

## 📊 **完整数据流程（从上传到展示）**

### **第一步：上传文件**
- **API**: `POST /uploadEncryptedFile?stream=true`
- **文件格式**: xlsx, xls, json, pdf
- **加密方式**: RSA-2048（密钥）+ AES-256-GCM（文件）
- **响应**: Server-Sent Events (SSE) 流式返回

### **第二步：文件处理与存储 (6 个关键步骤)**

1. **解密** → RSA 解密 AES 密钥，AES-GCM 解密文件
2. **上传原始文件** → 加密后上传到 Blob 存储 (S3)
3. **DuckDB 导入** → 自动解析 Excel 表结构
4. **AI 结构分析** → SchemaAnalyticsWorkflow 生成字段元数据
5. **数据库导出** → 将 DuckDB 加密后上传到 Blob
6. **向量嵌入** → Elasticsearch 索引，MySQL 存储元数据

### **第三步：看板生成 (三个阶段)**

**阶段 1: 看板规划**
- LLM 根据用户查询 + 上下文生成看板 XML 规划

**阶段 2: 代码生成与执行**
- LLM 为每个图表生成 Python 代码（并发 ≤4）
- 最多重试 3 次，每次失败都反馈错误给 LLM
- 代码必须包含：DuckDB 查询、pandas 处理、pyecharts 可视化

**阶段 3: 结果汇聚与总结**
- 分离成功/失败图表
- LLM 生成看板总结 Markdown
- 构建最终响应卡片

### **第四步：前后端交互**
- **流式响应**: SSE 格式（progress → delta → end）
- **卡片类型**: MarkdownCard + HtmlCard (ECharts) + CodeAndConsoleCard
- **前端渲染**: 逐步加载并展示看板

---

## 🔑 **关键代码片段**

### **1. 文件上传API**
```python
# /stargate_api/routers/file_router.py
@router.post("/uploadEncryptedFile")
async def upload_encrypted_file(
    encrypted_files: List[UploadFile],
    encryptedKeys: List[str],
    stream: bool = True
) -> StreamingResponse:
    file_ids_task = asyncio.create_task(
        file_service.analyze_and_upload_file2blob(...)
    )
    # 返回 SSE 流式响应
    return StreamingResponse(
        content=event_stream_generator(),
        media_type="text/event-stream"
    )
```

### **2. 文件分析核心处理**
```python
# /stargate_core/services/file_service.py
async def analyze_and_upload_file2blob(...):
    # Step 1: 解密
    key = decrypt_rsa(private_key, encrypted_key)
    
    # Step 2: 上传到 Blob
    await blob_store.upload_file(file, file_id)
    
    # Step 3: DuckDB 导入
    con, table_to_sheet_map = await load_file_to_duckdb(path, fs)
    
    # Step 4: AI 分析
    await analyze_file_info_with_filesystem(...)
    
    # Step 5: DuckDB 导出 & 加密
    con.execute(f"ATTACH '{db_file}' AS file")
    con.execute('COPY FROM DATABASE memory TO file')
    await encrypt_aes(db_file, encrypted_path, key)
    
    # Step 6: 向量嵌入 & 存储
    await embedding_and_insert_file(...)
```

### **3. 看板生成核心逻辑**
```python
# /stargate_analytics/tools/generate_dashboard.py
async def generate_dashboard(ctx, code_sandbox_service):
    # 阶段 1: 规划
    plan = await _generate_dashboard_plan(ctx, user_query)
    charts = plan["charts"]  # [(title, desc), ...]
    
    # 阶段 2: 并行生成代码 & 执行
    async def process_chart(title, desc):
        for attempt in range(1, 4):
            # 2.1 LLM 生成代码
            llm_response = await llm.astream_chat(messages)
            
            # 2.2 提取代码块
            code = re.findall(r"<python_code>(.*?)</python_code>", ...)[0]
            
            # 2.3 执行代码
            result = await code_sandbox_service.board_run_code(code)
            
            # 2.4 验证结果
            if result.artifacts and "<dataframe>" in result.stdout:
                return {"success": True, "code": code, ...}
            
            # 重试 (反馈错误给 LLM)
        return {"success": False, "error": last_error}
    
    # 阶段 3: 汇聚结果
    results = await asyncio.gather(*[process_chart(...)])
    summary = await generate_summary_markdown(successful_charts)
    return summary, board_json
```

### **4. 代码沙箱执行**
```python
# /stargate_analytics/service/code_sandbox_service.py
async def board_run_code(username, app_type, code):
    # 使用 Jinja2 模板包装用户代码
    template = self._jinja_env.get_template('board_sandbox_template')
    formatted_code = template.render(
        username=username,
        app_type=app_type,
        code=code
    )
    
    # 调用沙箱服务 REST API
    response = await session.post(
        'http://sandbox:8080/v1/sandbox/run',
        json={'language': 'python3', 'code': formatted_code}
    )
    
    # 返回执行结果 (JSON)
    return CodeExecuteResult.model_validate_json(response.stdout)
```

---

## 📁 **涉及的所有关键文件路径**

### **API 路由**
- `/stargate_api/routers/file_router.py` → 文件上传/下载/预览
- `/stargate_api/routers/v2/ai_chat_router_v2.py` → 聊天/看板生成

### **核心服务**
- `/stargate_core/services/file_service.py` → 文件处理主服务
- `/stargate_core/services/blob_store_service.py` → 对象存储
- `/stargate_core/utils/file_tool.py` → 文件工具（加密/解密/DuckDB）

### **工作流**
- `/stargate_analytics/workflows/data_analytics_workflow.py` → 主 Agent 循环
- `/stargate_analytics/workflows/file_analytics_workflow.py` → 文件分析工作流
- `/stargate_analytics/workflows/schema_analytics_workflow.py` → 表结构分析

### **看板生成**
- `/stargate_analytics/tools/generate_dashboard.py` → 看板规划/代码生成/汇聚
- `/stargate_analytics/service/code_sandbox_service.py` → 代码沙箱执行

### **数据模型**
- `/stargate_core/models/entities/file_info.py` → FileInfo 实体
- `/stargate_core/models/schemas/file_schema.py` → ExcelSheetSchema
- `/stargate_core/models/schemas/card_schema.py` → 卡片模型

### **Prompt & 模板**
- `/stargate_analytics/prompts/dashboard/plan_v1.md` → 看板规划 Prompt
- `/stargate_analytics/prompts/dashboard/codegen_v1.md` → 代码生成 Prompt
- `/stargate_analytics/service/board_sandbox_prescript.py.jinja` → 沙箱预设脚本

---

## ⚙️ **核心特点**

✅ **全链路加密**: RSA + AES-GCM，支持客户端加密  
✅ **流式响应**: SSE 实时反馈进度  
✅ **异步并发**: 最多 4 个图表并行生成  
✅ **容错重试**: 代码失败自动重试 3 次，LLM 基于错误调整  
✅ **虚拟文件系统**: FakeFS 减少磁盘 I/O  
✅ **DuckDB 快速查询**: 内存数据库，极速处理  
✅ **代码沙箱隔离**: 独立服务，支持水平扩展  

希望这份详细分析对你有帮助！如有任何问题，欢迎继续提问 😊
