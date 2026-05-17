<!-- scene: layers.growth.telos.dimensions.people.pipeline | variables: content, known_people, mentioned_names -->
分析以下信号文本，提取其中出现的人物互动信息。

信号文本：
{content}

已知人物列表（摘要）：
{known_people}

本次信号中提到的人名：{mentioned_names}

对每个提到的人物，提取：
- interaction_type: 从 [合作, 冲突, 日常, 初识, 久未联系] 中选择
- emotional_score: 此次互动对用户情感的影响，-1.0（极负面）到 1.0（极正面）
- summary: 一句话描述此次互动
- new_profile: 若发现新的画像信息（职位/性格/兴趣），填写；否则 null
- new_relation_type: 若关系类型发生变化，填写；否则 null

只返回 JSON 数组，不要其他内容。
