---
session_id: agent-6ea5421b
date: 2026-03-27
time_start: 2026-03-27T12:10:08.854Z
time_end: 2026-03-27T12:10:55.367Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-03-27 20:10:08] [user]: 探索 XP 项目的经验召回机制实现。请查找：
1. 检索相关的代码文件（search, retrieve, query 等）
2. 混合检索的实现（BM25 + Embedding）
3. 召回流程和排序逻辑
4. 相关的配置文件

重点关注 src/ 目录下的实现，返回关键文件路径和核心逻辑说明。
[2026-03-27 20:10:13] [assistant]: 我来帮你探索 XP 项目的经验召回机制实现。让我从多个维度搜索相关代码。
[2026-03-27 20:10:22] [assistant]: 现在让我读取核心文件来理解召回机制的实现：
[2026-03-27 20:10:55] [assistant]: 根据探索结果，我来总结 XP 项目的经验召回机制实现：

---

## XP 经验召回机制实现分析

### 一、核心文件路径

| 文件 | 职责 |
|------|------|
| `/Users/lianzimeng/workspace/xp/src/knowledge.py` | **核心召回逻辑** - 混合检索入口、A/B 测试、排序策略 |
| `/Users/lianzimeng/workspace/xp/src/storage.py` | **BM25 检索实现** - 分词、索引、纯 BM25 搜索 |
| `/Users/lianzimeng/workspace/xp/src/embeddings.py` | **Embedding 模型封装** - BGE-small-zh 语义向量、缓存 |
| `/Users/lianzimeng/workspace/xp/src/server.py` | **MCP Server** - 暴露 `search_best_practices` 工具 |
| `/Users/lianzimeng/workspace/xp/src/models.py` | **数据模型** - Experience、Metadata 定义 |

---

### 二、混合检索实现（BM25 + Embedding）

#### 权重配置
```
混合分数 = 0.7 * BM25归一化分数 + 0.3 * Embedding余弦相似度
```

#### 核心代码位置

**`knowledge.py` 第 183-285 行** - 混合检索主流程：

```python
async def search(
    self,
    query: str,
    tags: Optional[list[str]] = None,
    top_k: int = 3,
    threshold: float = 0.1,
    session_id: Optional[str] = None,
    enable_ab_test: bool = True,
    enable_strategy_ab_test: bool = True,
) -> tuple[list[Experience], dict]:
```

#### 三种检索策略

| 策略 | 说明 | 触发条件 |
|------|------|---------|
| `bm25_only` | 仅 BM25 排序 | A/B 测试分组或 Embedding 失败时回退 |
| `embedding_only` | 仅 Embedding 排序 | A/B 测试分组 |
| `hybrid` | **默认混合策略** | 非测试场景 |

---

### 三、召回流程详解

```
┌─────────────────────────────────────────────────────────────┐
│                    1. 候选集获取                             │
│   list_active() → 项目过滤 → TTL过期过滤 → stale过滤         │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    2. 标签过滤                               │
│   tags → tech_stack 精确匹配                                 │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    3. A/B 测试分组                           │
│   - 整体 A/B：10% control组(无结果) vs 90% treatment        │
│   - 策略 A/B：50% BM25 vs 50% Embedding                     │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    4. BM25 粗排                              │
│   语料: title + problem + solution + keywords               │
│   分词: 正则提取 [a-z]+ | [\u4e00-\u9fa5]+ | \d+            │
│   阈值: score >= 0.1                                        │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    5. Embedding 精排                         │
│   模型: BAAI/bge-small-zh-v1.5 (512维)                      │
│   计算: cosine_similarity(query_vec, doc_vecs)             │
│   归一化: BM25 分数归一化到 [0,1]                            │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    6. 混合打分 & 排序                        │
│   hybrid_score = 0.7 * bm25_norm + 0.3 * semantic_score    │
│   取 top_k 返回                                              │
└─────────────────────────────────────────────────────────────┘
```

---

### 四、关键实现细节

#### 1. BM25 实现 (`storage.py`)

**分词函数** 第 129-133 行：
```python
def _tokenize(text: str) -> list[str]:
    text = text.lower()
    tokens = re.findall(r'[a-z]+|[\u4e00-\u9fa5]+|\d+', text)
    return tokens
```

**纯 BM25 搜索** 第 205-239 行：
```python
def search(self, query, tech_stack, problem_type, scene, top_k, threshold):
    # 1. 标签过滤
    # 2. BM25 排序
    corpus = [f"{exp.title}\n{exp.problem}\n{exp.solution}\n{' '.join(exp.metadata.keywords)}" ...]
    bm25 = BM25Okapi(corpus)
    scores = bm25.get_scores(query_tokens)
```

#### 2. Embedding 实现 (`embeddings.py`)

**模型配置**：
- 模型: `BAAI/bge-small-zh-v1.5`
- 维度: 512
- 大小: ~100MB
- 本地运行，无需 API Key

**缓存机制**：
```python
class EmbeddingCache:
    - _memory_cache: 内存缓存 dict
    - _cache_dir: 磁盘缓存目录
    - get_or_compute(): 先查缓存，不存在则计算
    - get_or_compute_batch(): 批量获取，避免重复计算
```

**余弦相似度** 第 63-72 行：
```python
def cosine_similarity(query_vec: np.ndarray, doc_vecs: np.ndarray) -> np.ndarray:
    # 向量已归一化，点积即余弦相似度
    return np.dot(doc_vecs, query_vec)
```

#### 3. A/B 测试机制

**两层 A/B 测试**：

1. **整体效果对比** (`knowledge.py` 第 192-199 行)：
   - 10% control 组：不展示经验
   - 90% treatment 组：正常展示

2. **检索策略对比** (`knowledge.py` 第 226-231 行)：
   - 50% BM25 only
   - 50% Embedding only
   - 用于对比两种策略的长期效果

**数据记录**：
- `search_strategy_ab_test` 表：记录每次检索的策略和结果
- `search_strategy_metrics` 表：记录每种策略的累计效果指标

---

### 五、设计文档

**`docs/hybrid-search-design.md`** 记录了混合检索的设计决策：

| 场景 | BM25 | Embedding | 结论 |
|------|------|-----------|------|
| 精确匹配函数名/配置项 | ✅ 精确 | ❌ 语义无意义 | BM25 赢 |
| 语义相似字面不同 | ❌ 匹配不到 | ✅ 理解语义 | Embedding 赢 |
| 通用场景 | ✅ | ✅ | 混合最优 |

---

### 六、配置依赖

**`pyproject.toml`** 关键依赖：
```
rank-bm25>=0.2.2          # BM25 算法
sentence-transformers>=2.2.0  # Embedding 模型
numpy>=1.24.0             # 向量计算
```

---

### 七、总结

XP 的经验召回机制采用了**两层检索架构**：

1. **粗筛层（BM25）**：快速召回，保证专业术语、代码片段不丢失
2. **精排层**：语义理解，提升相似但字面不同的召回

**核心优势**：
- 无需 API Key，本地运行
- 混合检索兼顾精确性和语义理解
- A/B 测试机制持续优化策略
- 向量缓存避免重复计算
