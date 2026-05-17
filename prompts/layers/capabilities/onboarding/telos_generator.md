<!-- scene: layers.capabilities.onboarding.telos_generator | variables: qa_text, dimensions -->
根据用户的自述，为每个有回答的维度生成初始认知描述。
要求：
- 语言简洁，不要分析腔，像朋友在帮他整理想法
- 每个维度 50 字以内
- 没有回答的维度输出 null

用户回答：
{qa_text}

请为以下维度生成内容：{dimensions}

输出合法 JSON，格式：{{"dimension_name": "内容或 null"}}
