<!-- scene: layers.capabilities.personality.updater | variables: openness, conscientiousness, extraversion, agreeableness, neuroticism, interests, entries_text -->
分析以下日记内容，识别用户画像的潜在变化。

当前画像：
- 性格开放度: {openness}
- 责任心: {conscientiousness}
- 外向性: {extraversion}
- 宜人性: {agreeableness}
- 情绪稳定性: {neuroticism}
- 兴趣: {interests}

日记内容：
{entries_text}

请分析是否有以下变化：
1. 新的兴趣爱好
2. 价值观变化
3. 行为模式变化
4. 目标变化

以 JSON 格式返回。如果没有明显变化，返回空数组。
