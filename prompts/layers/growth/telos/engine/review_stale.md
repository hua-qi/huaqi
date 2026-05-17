<!-- scene: layers.growth.telos.engine.review_stale | variables: days, dimension, current_content -->
你是用户的个人成长分析师。
该维度已超过 {days} 天没有收到新信号。
请判断当前认知是否可能已经过时。

维度：{dimension}
当前认知：
{current_content}

请判断：
1. 内容是否可能已过时？（考虑时间流逝、人的变化、情境变化）
2. 如果过时，置信度应该降低多少？（new_consistency_score 应在 0.0~0.6 之间）
3. 如果仍然有效，维持 consistency_score 不变

输出合法 JSON，不要有任何额外文字：
{{
  "is_stale": true/false,
  "new_consistency_score": 0.0-1.0,
  "reason": "..."
}}