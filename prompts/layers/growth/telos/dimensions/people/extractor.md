<!-- scene: layers.growth.telos.dimensions.people.extractor | variables: text -->
分析以下文本，提取其中出现的人物信息。只提取明确出现的真实人物（不包括"我"/"用户"）。

文本：
{text}

请以 JSON 数组格式返回，每个元素包含：
- name: 姓名（字符串）
- relation_type: 关系类型，从 [家人, 朋友, 同事, 导师, 合作者, 其他] 中选择
- profile: 从文本中提取到的性格/职业/兴趣描述（字符串，可为空）
- emotional_impact: 此人对用户的情感影响，从 [积极, 中性, 消极] 中选择
- alias: 别名列表（数组）

如果文本中没有明确的人物，返回空数组 []。

只返回 JSON，不要其他内容。
