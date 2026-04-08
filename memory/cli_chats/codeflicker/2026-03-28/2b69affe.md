---
session_id: 2b69affe
date: 2026-03-28
time_start: 2026-03-28T19:02:38.614Z
time_end: 2026-03-28T19:06:38.514Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-03-29 03:02:38] [user]: [Image #1] 如图所示，如何配置？
[2026-03-29 03:02:51] [assistant]: 运行以下命令配置 LLM：

```bash
huaqi config set-llm openai --api-key YOUR_API_KEY
```

可选参数：
- `--api-base` 自定义 API 地址（如使用代理/国内镜像）
- `--model` 指定模型名（默认 `gpt-3.5-turbo`）

例如使用 DeepSeek：
```bash
huaqi config set-llm deepseek --api-key YOUR_API_KEY --model deepseek-chat
```

支持的提供商：`openai` / `claude` / `deepseek` / `dummy`
[2026-03-29 03:06:31] [user]: 我的 LLM 使用示例 import os
from openai import OpenAI

# 请确保您已将 API Key 存储在环境变量 WQ_API_KEY 中
# 初始化 OpenAI 客户端，从环境变量中读取您的 API Key
client = OpenAI(
    # 如需办公网调用，请使用：https://wanqing-api.corp.kuaishou.com/api/gateway/v1/endpoints
    base_url="http://wanqing.internal/api/gateway/v1/endpoints",
    # 从环境变量中获取您的 API Key
    api_key=os.environ.get("WQ_API_KEY")
)

# 流式输出示例
print("----- 流式输出 -----")
stream = client.chat.completions.create(
    model="ep-czo1x9-1774507780045283571",  # ep-czo1x9-1774507780045283571 为您当前的推理接入点ID
    messages=[
        {"role": "system", "content": "你是一个 AI 人工智能助手"},
        {"role": "user", "content": "请介绍一下太阳系的八大行星"},
    ],
    stream=True,
)
for chunk in stream:
    if not chunk.choices:
        continue
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
print() ，我应该如何配置？
[2026-03-29 03:06:38] [assistant]: ```bash
huaqi config set-llm openai \
  --api-key $WQ_API_KEY \
  --api-base "http://wanqing.internal/api/gateway/v1/endpoints" \
  --model "ep-czo1x9-1774507780045283571"
```
