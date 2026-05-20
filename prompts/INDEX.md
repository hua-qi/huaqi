# 提示词索引

共 40 个提示词文件。

| 文件 | Scene ID | 影响的功能 |
|------|----------|-----------|
| `agent/chat.md` | `agent.chat` | ChatAgent 对话 (`huaqi chat`) |
| `base.md` | `base` | 所有场景的角色基线 |
| `cli/chat.md` | `cli.chat` | CLI 交互模式 |
| `layers/capabilities/learning/feedback.md` | `layers.capabilities.learning.feedback` | layers.capabilities.learning.feedback 场景 |
| `layers/capabilities/learning/lesson.md` | `layers.capabilities.learning.lesson` | layers.capabilities.learning.lesson 场景 |
| `layers/capabilities/learning/outline.md` | `layers.capabilities.learning.outline` | layers.capabilities.learning.outline 场景 |
| `layers/capabilities/learning/quiz.md` | `layers.capabilities.learning.quiz` | layers.capabilities.learning.quiz 场景 |
| `layers/capabilities/onboarding/telos_generator.md` | `layers.capabilities.onboarding.telos_generator` | 引导式 TELOS 初始化 |
| `layers/capabilities/personality/engine.md` | `layers.capabilities.personality.engine` | 个性引擎系统提示词 |
| `layers/capabilities/personality/updater.md` | `layers.capabilities.personality.updater` | 人格画像自动更新 |
| `layers/capabilities/reports/daily.md` | `layers.capabilities.reports.daily` | 日终复盘 |
| `layers/capabilities/reports/growth.md` | `layers.capabilities.reports.growth` | 成长报告 |
| `layers/capabilities/reports/morning.md` | `layers.capabilities.reports.morning` | 晨间简报 |
| `layers/capabilities/reports/quarterly.md` | `layers.capabilities.reports.quarterly` | 季报 |
| `layers/capabilities/reports/weekly.md` | `layers.capabilities.reports.weekly` | 周报 |
| `layers/capabilities/world_news_enricher.md` | `layers.capabilities.world_news_enricher` | 世界新闻富化与翻译（旧版，已废弃） |
| `layers/capabilities/world_news_enricher_source.md` | `layers.capabilities.world_news_enricher_source` | 世界新闻按源增强（per-source） |
| `layers/data/memory/relevance.md` | `layers.data.memory.relevance` | 记忆相关性评估 |
| `layers/data/profile/extract.md` | `layers.data.profile.extract` | 用户信息提取 |
| `layers/data/profile/narrative.md` | `layers.data.profile.narrative` | 用户画像叙事生成 |
| `layers/growth/telos/context/chat.md` | `layers.growth.telos.context.chat` | layers.growth.telos.context.chat 场景 |
| `layers/growth/telos/context/distill.md` | `layers.growth.telos.context.distill` | layers.growth.telos.context.distill 场景 |
| `layers/growth/telos/context/onboarding.md` | `layers.growth.telos.context.onboarding` | layers.growth.telos.context.onboarding 场景 |
| `layers/growth/telos/context/report.md` | `layers.growth.telos.context.report` | layers.growth.telos.context.report 场景 |
| `layers/growth/telos/dimensions/people/extractor.md` | `layers.growth.telos.dimensions.people.extractor` | 人物信息提取 |
| `layers/growth/telos/dimensions/people/pipeline.md` | `layers.growth.telos.dimensions.people.pipeline` | 人物互动管道 |
| `layers/growth/telos/engine/review_stale.md` | `layers.growth.telos.engine.review_stale` | layers.growth.telos.engine.review_stale 场景 |
| `layers/growth/telos/engine/step1.md` | `layers.growth.telos.engine.step1` | layers.growth.telos.engine.step1 场景 |
| `layers/growth/telos/engine/step3.md` | `layers.growth.telos.engine.step3` | layers.growth.telos.engine.step3 场景 |
| `layers/growth/telos/engine/step345.md` | `layers.growth.telos.engine.step345` | layers.growth.telos.engine.step345 场景 |
| `layers/growth/telos/engine/step4.md` | `layers.growth.telos.engine.step4` | layers.growth.telos.engine.step4 场景 |
| `layers/growth/telos/engine/step5.md` | `layers.growth.telos.engine.step5` | layers.growth.telos.engine.step5 场景 |
| `scheduler/job_runner/learning.md` | `scheduler.job_runner.learning` | scheduler.job_runner.learning 场景 |
| `scheduler/job_runner.md` | `scheduler.job_runner` | 定时任务执行时的系统提示词 |
| `scheduler/jobs/daily_report.md` | `scheduler.jobs.daily_report` | scheduler.jobs.daily_report 场景 |
| `scheduler/jobs/learning_daily_push.md` | `scheduler.jobs.learning_daily_push` | scheduler.jobs.learning_daily_push 场景 |
| `scheduler/jobs/morning_brief.md` | `scheduler.jobs.morning_brief` | scheduler.jobs.morning_brief 场景 |
| `scheduler/jobs/quarterly_report.md` | `scheduler.jobs.quarterly_report` | scheduler.jobs.quarterly_report 场景 |
| `scheduler/jobs/weekly_report.md` | `scheduler.jobs.weekly_report` | scheduler.jobs.weekly_report 场景 |
| `scheduler/jobs/world_fetch.md` | `scheduler.jobs.world_fetch` | scheduler.jobs.world_fetch 场景 |

## 使用说明

- 每个 `.md` 文件对应一个场景的提示词
- 文件第一行 `<!-- scene: xxx | variables: a, b -->` 声明场景标识和模板变量
- `---` 水平线分隔 system prompt（上）和 user prompt（下）
- 修改文件后**立即生效**，无需重启
- 删除文件后系统使用内置默认值
- `base.md` 是所有场景共享的角色基线，修改它会影响所有场景
