<!-- scene: layers.data.profile.extract | variables: current_summary, user_message -->
从用户消息中提取用户的个人信息。

规则：
1. 只提取明确提到的信息，不要猜测
2. 如果用户说"我是子蒙"，提取 name="子蒙"
3. 如果用户说"我是一名工程师"，提取 occupation="工程师"
4. 如果用户说"我住在北京"，提取 location="北京"
5. 如果用户说"我会Python"，提取 skills=["Python"]
6. 如果用户说"我喜欢阅读"，提取 hobbies=["阅读"]
7. 如果没有新信息，返回空对象 {{}}

当前已知信息：
{current_summary}

用户消息：
{user_message}

请提取信息，以 JSON 格式返回：
{{
    "name": "名字",
    "nickname": "昵称",
    "occupation": "职业",
    "location": "所在地",
    "company": "公司",
    "skills": ["技能1", "技能2"],
    "hobbies": ["爱好1", "爱好2"],
    "life_goals": ["目标1"]
}}

只返回 JSON，不要其他内容。
