---
session_id: agent-315ba071
date: 2026-04-02
time_start: 2026-04-02T11:55:13.845Z
time_end: 2026-04-02T11:56:14.578Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-04-02 19:55:13] [user]: Analyze the codebase to explain in detail the generation flow of the "Morning Brief" (早报) in the Huaqi project.
Please look at:
1. `huaqi_src/layers/capabilities/reports/morning_brief.py`
2. Relevant data providers in `huaqi_src/layers/capabilities/reports/providers/`
3. How it's triggered (is there a scheduler job? CLI command?)
4. The general step-by-step process (Data gathering -> Prompting -> LLM Generation -> Output).

Return a clear, step-by-step technical explanation of the flow.
[2026-04-02 19:55:17] [assistant]: I'll analyze the Morning Brief generation flow systematically. Let me start by reading the key files and exploring the relevant directories.
[2026-04-02 19:55:22] [assistant]: Now let me search for how the Morning Brief is triggered - looking for scheduler, CLI commands, or agent integration.
[2026-04-02 19:55:41] [assistant]: Let me also check the LLM configuration context and the data directories:
[2026-04-02 19:56:14] [assistant]: Now I have a complete picture. Let me provide a detailed technical explanation of the Morning Brief generation flow.

---

## Morning Brief (早报) Generation Flow - Technical Analysis

### Overview

The Morning Brief is an automated daily report generated at 8:00 AM that provides the user with:
1. World news highlights (今日世界热点)
2. Recent diary snippets (近期日记片段)
3. Active relationship overview (近期活跃关系人)
4. An encouraging message

---

### Step-by-Step Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TRIGGER (08:00 Daily)                                │
│                     APScheduler Cron: "0 8 * * *"                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    1. SCHEDULER INVOCATION                                   │
│         huaqi_src/scheduler/jobs.py::_run_morning_brief()                   │
│                                                                              │
│     def _run_morning_brief():                                               │
│         agent = MorningBriefAgent()                                         │
│         agent.run()                                                          │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    2. PROVIDER REGISTRATION                                  │
│        MorningBriefAgent._register_providers()                              │
│                                                                              │
│     Clears existing world/diary/people providers                            │
│     Re-registers fresh instances:                                           │
│       - WorldProvider(data_dir)                                             │
│       - DiaryProvider(data_dir)                                             │
│       - PeopleProvider(data_dir)                                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    3. CONTEXT BUILDING                                      │
│        MorningBriefAgent._build_context()                                   │
│                                                                              │
│     DateRange: start=today, end=today                                       │
│     Calls: build_context("morning", date_range)                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│               4. DATA PROVIDER ORCHESTRATION                                 │
│          context_builder.build_context()                                    │
│                                                                              │
│     Iterates through registered providers (sorted by priority):             │
│                                                                              │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │ WorldProvider (priority=10)                                      │     │
│     │ File: data_dir/world/{today}.md                                  │     │
│     │ Output: "## 今日世界热点\n{content[:1000]}"                        │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                         │                                                    │
│                         ▼                                                    │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │ DiaryProvider (priority=20)                                      │     │
│     │ Files: data_dir/memory/diary/{last_7_days}.md                    │     │
│     │ Output: "## 近期日记片段\n{snippets}"                              │     │
│     │ Note: Only for "morning" report, aggregates up to 7 days         │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                         │                                                    │
│                         ▼                                                    │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │ PeopleProvider (priority=40)                                      │     │
│     │ Source: PeopleGraph.list_people()                                │     │
│     │ Output: "## 近期活跃关系人\n- {name}（{relation}）{notes}"         │     │
│     │ Note: Top 3 by interaction_frequency                             │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│     Final context = joined with "\n\n" or "暂无上下文数据。"                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    5. LLM CONFIGURATION                                     │
│         MorningBriefAgent._generate_brief()                                 │
│                                                                              │
│     Uses: build_llm_manager(temperature=0.7, max_tokens=500)                │
│     Creates: ChatOpenAI instance from config                                 │
│                                                                              │
│     System Prompt:                                                           │
│       "你是 huaqi，用户的 AI 同伴。请根据以下背景信息，生成一份简洁温暖的    │
│        晨间简报，包含：1）今日世界热点摘要（如有），2）对用户近期状态的       │
│        简短观察，3）一句鼓励的话。简报应简短，不超过 300 字。"               │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    6. LLM GENERATION                                        │
│                                                                              │
│     Messages:                                                                │
│       - SystemMessage(content=system_prompt)                                │
│       - HumanMessage(content=f"背景信息：\n{context}")                      │
│                                                                              │
│     Response: LLM-generated brief text (max 300 Chinese chars)             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    7. OUTPUT STORAGE                                         │
│                                                                              │
│     Path: {data_dir}/reports/daily/{today}-morning.md                       │
│     Format:                                                                  │
│       # 晨间简报 {YYYY-MM-DD}                                                │
│                                                                              │
│       {brief_content}                                                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Key Files and Their Roles

| File | Role |
|------|------|
| `huaqi_src/scheduler/jobs.py` | Defines `_run_morning_brief()` scheduler job, registered at `"0 8 * * *"` |
| `huaqi_src/scheduler/manager.py` | `SchedulerManager` using APScheduler with SQLite persistence |
| `huaqi_src/layers/capabilities/reports/morning_brief.py` | `MorningBriefAgent` - main orchestrator class |
| `huaqi_src/layers/capabilities/reports/context_builder.py` | `build_context()` - aggregates data from providers |
| `huaqi_src/layers/capabilities/reports/providers/__init__.py` | `DataProvider` abstract base class and registry |
| `huaqi_src/layers/capabilities/reports/providers/world.py` | `WorldProvider` - reads world news from markdown files |
| `huaqi_src/layers/capabilities/reports/providers/diary.py` | `DiaryProvider` - reads diary entries from markdown files |
| `huaqi_src/layers/capabilities/reports/providers/people.py` | `PeopleProvider` - reads relationship data via `PeopleGraph` |

---

### Data Provider Registry Pattern

The system uses a **pluggable provider registry** for extensibility:

```python
# providers/__init__.py
_registry: list[DataProvider] = []

def register(provider: DataProvider) -> None:
    _registry.append(provider)

def get_providers(report_type: str) -> list[DataProvider]:
    return sorted(
        [p for p in _registry if report_type in p.supported_reports or "*" in p.supported_reports],
        key=lambda p: p.priority,
    )
```

Each provider:
- Has a `name`, `priority`, and `supported_reports`
- Implements `get_context(report_type, date_range) -> str | None`
- Self-registers on import (if data_dir is available)

---

### Trigger Mechanisms

The Morning Brief can be triggered in **two ways**:

1. **Automatic (Daemon Mode)**:
   ```bash
   huaqi daemon start          # Start scheduler daemon
   huaqi daemon install        # Install as macOS LaunchAgent (auto-start on boot)
   ```
   The daemon runs `register_default_jobs()` which adds the cron job `"0 8 * * *"`.

2. **Manual (via CLI - implicit)**:
   Although there's no dedicated CLI command for morning brief specifically, the test shows direct instantiation:
   ```python
   from huaqi_src.layers.capabilities.reports.morning_brief import MorningBriefAgent
   agent = MorningBriefAgent()
   brief = agent.run()
   ```

---

### Data Sources

| Provider | Data Source | File Path |
|----------|-------------|-----------|
| WorldProvider | RSS/news fetcher output | `{data_dir}/world/{YYYY-MM-DD}.md` |
| DiaryProvider | User diary entries | `{data_dir}/memory/diary/{YYYY-MM-DD}.md` |
| PeopleProvider | PeopleGraph (relationship DB) | `{data_dir}/people/{name}.md` |

---

### Configuration

- **LLM Settings**: Retrieved from `ConfigManager`, uses the `llm_default_provider` setting
- **Temperature**: 0.7 (for natural, varied output)
- **Max Tokens**: 500 (limit to keep brief concise)
- **Output**: Stored as markdown in `{data_dir}/reports/daily/`

---

### Related Reports

The same architecture powers:
- **Daily Report** (23:00) - more comprehensive end-of-day summary
- **Weekly Report** (Sunday 21:00) - week-level analysis
- **Quarterly Report** - quarter-level retrospective
