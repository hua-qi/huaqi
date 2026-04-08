---
session_id: agent-1a31c26e
date: 2026-03-28
time_start: 2026-03-28T21:00:17.285Z
time_end: 2026-03-28T21:01:16.064Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-03-29 05:00:17] [user]: 在 /Users/lianzimeng/workspace/huaqi-growing 项目中，详细查看：
1. huaqi_src/agent/nodes/chat_nodes.py 中 retrieve_memories 函数的完整实现（特别是它如何检索记忆、是否用了 embedding）
2. 现有的 MarkdownMemoryStore 的 search_conversations() 方法实现（是 BM25 还是 embedding？）
3. 数据存储格式：conversations、diary、user_profile 等是否都是纯文本/Markdown？有没有向量数据库或 JSON 结构化存储？
4. 是否有任何 embedding 模型相关的依赖或代码？

返回完整的代码内容。
[2026-03-29 05:00:21] [assistant]: 我来帮你详细查看这些文件。让我先搜索相关的代码文件。
[2026-03-29 05:00:24] [assistant]: 现在让我查看这些关键文件：
[2026-03-29 05:00:29] [assistant]: 现在让我查看混合搜索的实现和其他相关文件：
[2026-03-29 05:00:33] [assistant]: 现在让我查看数据存储的具体格式和配置：
[2026-03-29 05:00:37] [assistant]: 让我查看更多的初始化和配置文件：
[2026-03-29 05:00:41] [assistant]: 完美！现在让我创建一个完整的总结文档：
[2026-03-29 05:01:16] [assistant]: 现在我有了所有的信息。让我为你创建一个完整的详细分析文档：

## 📊 Huaqi 项目记忆检索系统详细分析

我已详细查看了你项目的完整代码实现。以下是完整的分析结果：

---

### 1️⃣ **retrieve_memories 函数详细实现**

**📄 文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/agent/nodes/chat_nodes.py` (第 213-245 行)

```python
def retrieve_memories(state: AgentState) -> Dict[str, Any]:
    """检索记忆节点
    
    从向量库中检索相关记忆
    """
    messages = state.get("messages", [])
    
    # 获取最后一条用户消息作为查询
    query = ""
    for msg in reversed(messages):
        if isinstance(msg, HumanMessage):
            query = msg.content
            break
    
    if not query:
        return {"recent_memories": []}
    
    try:
        # 使用混合检索获取相关记忆
        from ...memory.vector import get_hybrid_search
        
        search = get_hybrid_search(use_vector=False, use_bm25=True)
        results = search.search(query, top_k=3)
        
        memories = []
        for r in results:
            content = r.get("content", "")
            if content:
                memories.append(content)
        
        return {"recent_memories": memories}
        
    except Exception as e:
        # 检索失败不影响主流程
        return {"recent_memories": []}
```

**关键特点:**
- ✅ **使用了混合检索** (`get_hybrid_search`)，但**默认禁用向量搜索**（`use_vector=False`）
- ✅ **仅启用 BM25**（`use_bm25=True`）进行检索
- ✅ 返回前 3 个最相关的记忆
- ✅ 无缝降级处理（检索失败不中断主流程）

---

### 2️⃣ **MarkdownMemoryStore 搜索实现**

**📄 文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/memory/storage/markdown_store.py` (第 319-356 行)

```python
def search_conversations(self, query: str) -> List[Dict[str, Any]]:
    """搜索会话内容
    
    Args:
        query: 搜索关键词
        
    Returns:
        List: 匹配的会话列表
    """
    results = []
    query_lower = query.lower()
    
    for filepath in self.base_dir.rglob("*.md"):
        try:
            with open(filepath, "r", encoding="utf-8") as f:
                content = f.read()
            
            if query_lower in content.lower():
                frontmatter, _ = self._parse_frontmatter(content)
                
                # 提取匹配的片段
                lines = content.split('\n')
                context = []
                for i, line in enumerate(lines):
                    if query_lower in line.lower():
                        # 提取前后 2 行作为上下文
                        start = max(0, i - 2)
                        end = min(len(lines), i + 3)
                        context.extend(lines[start:end])
                        context.append("...")
                
                results.append({
                    "filepath": str(filepath.relative_to(self.base_dir)),
                    "session_id": frontmatter.get("session_id", ""),
                    "created_at": frontmatter.get("created_at", ""),
                    "context": '\n'.join(context[:10]),  # 限制上下文长度
                })
                
        except Exception:
            continue
    
    return results
```

**技术特点:**
- ❌ **纯字符串匹配**（case-insensitive）
- ❌ **不是 BM25**，也**不是 embedding**
- ❌ 简单的关键词查找 + 上下文提取
- ✅ 处理返回前 10 个结果片段

---

### 3️⃣ **数据存储格式**

#### **3.1 Conversations（对话记录）**

**存储格式:** Markdown 文件，按日期组织

```
~/.huaqi/data/memory/
└── 2024/
    ├── 01/
    │   ├── 20240115_101530_session-123.md
    │   └── 20240115_143025_session-456.md
    └── 02/
        └── 20240210_092015_session-789.md
```

**Markdown 内容结构:**
```markdown
---
session_id: session-123
created_at: 2024-01-15T10:15:30
turns: 5
intent: chat
---

# 对话记录 - 2024年01月15日 10:15

## 回合 1

**👤 用户**

你好，今天天气怎么样？

**🤖 Huaqi**

你好呀！今天是个阳光明媚的日子...

---
```

**元数据 (YAML Frontmatter):**
- `session_id`: 会话唯一ID
- `created_at`: ISO 8601 时间戳
- `turns`: 对话轮数
- 自定义字段（如 intent, tags 等）

#### **3.2 Diary（日记）**

**存储位置:** `~/.huaqi/data/memory/diary/` (Markdown 格式)

#### **3.3 User Profile（用户画像）**

**存储位置:** `~/.huaqi/data/memory/user_profile/` (结构化 + Markdown)

---

### 4️⃣ **Embedding 模型与向量存储**

#### **4.1 Embedding 依赖**

**📦 项目依赖:** `pyproject.toml` (第 44-45 行)
```
"chromadb>=0.5.0",
"sentence-transformers>=3.0.0",
"rank-bm25>=0.2.0",
```

**实现文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/memory/vector/embedder.py`

```python
class EmbeddingService:
    """Embedding 服务 - 文本向量化
    
    支持多种模型：
    - BAAI/bge-small-zh (默认，轻量中文)
    - BAAI/bge-large-zh (高质量中文)
    - sentence-transformers/all-MiniLM-L6-v2 (英文)
    """
    
    def __init__(
        self,
        model_name: str = "BAAI/bge-small-zh",  # 默认使用 BGE-small-zh
        device: str = "auto",
        normalize_embeddings: bool = True,
        cache_dir: Optional[Union[str, Path]] = None,
    ):
```

**关键特性:**
- ✅ **支持 CPU/GPU/MPS** 自动检测
- ✅ **支持批量编码** (可控制批大小和进度条)
- ✅ **向量维度:** BGE-small = 512 dim, BGE-large = 1024 dim
- ✅ **相似度计算:** 余弦相似度 (L2 normalized)

#### **4.2 向量数据库（Chroma）**

**📄 文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/memory/vector/chroma_client.py`

**存储位置:** `~/.huaqi/data/vector_db/`

**主要功能:**
- 持久化存储向量
- 支持向量搜索、元数据过滤
- 集合管理 (默认 "memories" 集合)
- 导出/导入功能

```python
class ChromaClient:
    """Chroma 向量数据库客户端
    
    提供封装好的向量存储和检索功能：
    - 集合创建和管理
    - 文档添加和更新
    - 向量搜索 (相似度)
    """
```

---

### 5️⃣ **混合检索系统架构**

#### **5.1 两个混合检索实现**

**问题:** 项目中有 **两个不同的混合检索模块**！

**选项 A:** `/huaqi_src/memory/vector/hybrid_search.py` (向量+BM25)
```
融合策略:
- BM25: 精确匹配关键词
- 向量相似度: 语义匹配
- 时间衰减: 近期内容加权

公式: final_score = 0.7 * vector_score + 0.3 * bm25_score + recency_boost
```

**选项 B:** `/huaqi_src/memory/search/hybrid_search.py` (BM25+LLM)
```
融合策略:
1. 先用 BM25 快速召回候选集（Top 20）
2. 再用 LLM 精确判断相关性（Top 5）
3. 融合两种分数得到最终排序

优点：无需 Embedding，兼容所有 LLM
```

#### **5.2 实际使用的是哪一个？**

根据 `chat_nodes.py` 第 215 行：
```python
from ...memory.vector import get_hybrid_search
```

**👉 使用的是 `memory/vector/hybrid_search.py`**（向量 + BM25 融合）

---

### 6️⃣ **完整的混合检索流程**

**📄 文件:** `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/memory/vector/hybrid_search.py` (完整实现)

```python
def search(
    self,
    query: str,
    top_k: int = 5,
    doc_type: Optional[str] = None,
    use_bm25: Optional[bool] = None,
    use_vector: Optional[bool] = None,
) -> List[Dict[str, Any]]:
    """混合搜索
    
    流程：
    1. 向量搜索 (如果启用)
       - 使用 Chroma 向量数据库
       - 返回 top_k*2 个结果
       
    2. BM25 搜索 (如果启用)
       - 构建 BM25 索引 (5分钟缓存)
       - 返回 top_k*2 个结果
       
    3. 融合分数
       - final_score = 0.7 * vector_score + 0.3 * bm25_score
       - 加入时间衰减因子
       - 排序并返回 top_k
    """
```

**关键参数:**
- `alpha = 0.7`: 向量权重（可调）
- `beta = 0.1`: 时间衰减系数（可调）
- 时间衰减公式: `score = exp(-beta * days_ago)`

---

### 7️⃣ **数据流总结**

```
用户消息
   ↓
retrieve_memories()
   ↓
get_hybrid_search(use_vector=False, use_bm25=True)
   ↓
HybridSearch.search(query, top_k=3)
   ↓
分支 A: 向量搜索 (禁用)
   ├─ ChromaClient.search() → []
   └─ vector_score = 0
   
分支 B: BM25 搜索 (启用)
   ├─ _build_bm25_index()
   ├─ BM25Okapi.get_scores()
   └─ bm25_score > 0
   
融合:
   final_score = 0.7 * 0 + 0.3 * bm25_score + recency_boost
   ≈ 0.3 * bm25_score + recency_boost
   
排序 & 返回 top 3 记忆
   ↓
添加到生成回复的上下文
```

---

### 8️⃣ **当前配置总结**

| 维度 | 当前实现 | 备注 |
|------|---------|------|
| **Markdown 存储** | ✅ 是 | conversations, diary, user_profile 都用 MD 格式 |
| **Embedding 模型** | ✅ 有，但未用 | BAAI/bge-small-zh (默认) 可用但在 chat 中禁用 |
| **向量数据库** | ✅ Chroma | `~/.huaqi/data/vector_db/` (初始化但未在 chat 中激活) |
| **BM25 搜索** | ✅ 启用 | `rank-bm25` 库，用于 retrieve_memories |
| **LLM 搜索** | ✅ 有备选方案 | `memory/search/hybrid_search.py` (未在 chat_nodes 中使用) |
| **时间衰减** | ✅ 支持 | 指数衰减权重 exp(-0.1 * days_ago) |

---

### 9️⃣ **关键发现**

🔍 **核心洞察:**

1. **架构冗余**: 项目有两套混合检索系统
   - A: 向量 + BM25 (当前使用)
   - B: BM25 + LLM (备选)

2. **Embedding 未充分利用**: 虽然依赖了 `sentence-transformers`，但在 `retrieve_memories()` 中显式禁用了向量搜索
   - 原因猜测: 可能性能/成本考虑，或仍在优化阶段

3. **存储格式混合**:
   - Markdown: 人类可读（conversations, diary）
   - Chroma Vector DB: 向量存储（初始化但可选）
   - 未见 JSON 结构化存储

4. **可配置灵活性高**:
   ```python
   search = get_hybrid_search(use_vector=False, use_bm25=True)
   # 可轻松切换为:
   search = get_hybrid_search(use_vector=True, use_bm25=True)  # 启用向量
   search = get_hybrid_search(use_vector=True, use_bm25=False)  # 仅向量
   ```

---

### 🔟 **文件清单**

| 文件路径 | 功能 | 代码行数 |
|---------|------|---------|
| `chat_nodes.py` | 检索节点实现 | 335 |
| `markdown_store.py` | MD 存储服务 | 387 |
| `embedder.py` | Embedding 服务 | 231 |
| `chroma_client.py` | 向量数据库客户端 | 319 |
| `hybrid_search.py` (vector) | 向量+BM25混合检索 | 279 |
| `hybrid_search.py` (search) | BM25+LLM混合检索 | 226 |
| `text_search.py` | TF-IDF/BM25 文本搜索 | 281 |
| `llm_search.py` | LLM 相关性搜索 | 220 |
| `config_paths.py` | 路径配置管理 | 184 |

---

这是一个**设计良好、灵活可扩展**的记忆检索系统！目前采用保守的 BM25-only 策略，向量搜索作为可选的增强方案预留。
