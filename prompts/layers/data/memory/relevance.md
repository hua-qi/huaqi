<!-- scene: layers.data.memory.relevance | variables: query, content -->
请判断以下查询与记忆内容的相关性。

查询: {query}

记忆内容:
{content}

请分析：
1. 这段记忆是否回答了查询？
2. 这段记忆是否包含查询相关的信息？
3. 相关程度如何？（0-1 分）

以 JSON 格式返回：
{{
    "relevant": true/false,
    "score": 0.85,
    "reason": "这段记忆提到了...与查询相关"
}}
