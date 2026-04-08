---
session_id: agent-23c8b0c5
date: 2026-03-27
time_start: 2026-03-27T03:41:32.651Z
time_end: 2026-03-27T03:44:42.271Z
project: users-lianzimeng-workspace-huaqi
git_branch: main
---

[2026-03-27 11:41:32] [user]: Explore the Huaqi codebase structure to understand:
1. The overall architecture and main files
2. How cli.py works as the entry point
3. The core modules (diary, personality, llm, etc.)
4. The memory system

Focus on understanding the current implementation so we can discuss refactoring to a code agent architecture. Provide a comprehensive summary of the file structure and key components.
[2026-03-27 11:42:04] [assistant]: I'll thoroughly explore the Huaqi codebase to understand its architecture. Let me start by examining the overall structure.
[2026-03-27 11:42:13] [assistant]: Let me search for other file types to understand what language/framework this project uses:
[2026-03-27 11:42:17] [assistant]: Let me check for all files in the root and look for package.json, pyproject.toml, or other config files:
[2026-03-27 11:42:22] [assistant]: Let me explore the configuration files to understand the architecture:
[2026-03-27 11:42:33] [assistant]: Let me check the git branches and history to find the source code:
[2026-03-27 11:42:49] [assistant]: Let me check the gh-pages branch to see if there's source code there:
[2026-03-27 11:43:14] [assistant]: I see that this repository (`gh-pages` branch) contains a VuePress static site. Let me check if there's source code in the `config` directory or if we need to look elsewhere:
[2026-03-27 11:43:21] [assistant]: I notice this repository doesn't contain Python source code - it's primarily a VuePress static site on `gh-pages` and memory/config data on `main`. Let me check if the Huaqi source code might be elsewhere:
[2026-03-27 11:43:26] [assistant]: I found there's a `huaqi-growing` directory. Let me check if that contains the source code:
[2026-03-27 11:43:30] [assistant]: Excellent! I found the actual source code in `huaqi-growing`. Let me thoroughly explore this codebase:
[2026-03-27 11:43:34] [assistant]: Now let me explore the core modules in depth:
[2026-03-27 11:43:40] [assistant]: Let me continue exploring the remaining core modules:
[2026-03-27 11:43:43] [assistant]: Now let me explore the memory system in detail:
[2026-03-27 11:43:50] [assistant]: Let me check for additional documentation and explore remaining directories:
[2026-03-27 11:43:58] [assistant]: Let me check the empty directories and the test files:
[2026-03-27 11:44:42] [assistant]: Now I have a comprehensive understanding of the Huaqi codebase. Let me provide a detailed summary:

---

## Huaqi Codebase Architecture Summary

### Overview

**Huaqi (花旗)** is a personal AI companion system built with Python. The project follows a vision of "raising AI" rather than just "using AI" - creating a digital peer that grows with the user.

**Important Note:** There are two directories:
- `/Users/lianzimeng/workspace/huaqi` - Data storage directory (memory, config, user data)
- `/Users/lianzimeng/workspace/huaqi-growing` - Source code repository

---

### Project Structure

```
huaqi-growing/
├── cli.py                    # Main CLI entry point (~982 lines)
├── huaqi/                    # Core package
│   ├── __init__.py
│   ├── core/                 # Core business logic
│   │   ├── config_simple.py    # Configuration management
│   │   ├── diary_simple.py     # Diary/journal system
│   │   ├── git_auto_commit.py  # Auto Git sync
│   │   ├── growth_simple.py    # Skills & goals tracking
│   │   ├── hooks_simple.py     # Automation hooks
│   │   ├── llm.py              # LLM abstraction layer
│   │   └── personality_simple.py # AI personality engine
│   ├── memory/               # Memory system
│   │   ├── layers/             # (Empty - planned)
│   │   ├── search/             # Search algorithms
│   │   │   ├── hybrid_search.py  # BM25 + LLM hybrid
│   │   │   ├── llm_search.py     # LLM-based relevance
│   │   │   └── text_search.py    # TF-IDF + BM25
│   │   ├── storage/            # Data persistence
│   │   │   └── markdown_store.py # Markdown storage
│   │   └── sync/               # (Empty - planned)
│   ├── orchestration/         # (Empty - planned)
│   ├── security/              # (Empty - planned)
│   └── skills/                # (Empty - planned)
├── spec/                     # Documentation
│   ├── decisions/             # Architecture Decision Records
│   └── roadmap/
├── tests/
├── templates/
├── pyproject.toml
├── ARCHITECTURE.md
├── SPEC.md
└── ROADMAP.md
```

---

### Key Components Analysis

#### 1. CLI Entry Point (`cli.py`)

The CLI is built with **Typer + Rich + prompt_toolkit**:

```python
# Key structure:
app = typer.Typer(name="huaqi")

# Main command groups:
- chat_mode()         # Interactive conversation
- config_app          # Configuration management
- show_status()       # System status

# Slash commands in chat:
/skill <name>         # Add skill
/log <skill> <hours>  # Log practice time
/goal <title>         # Add goal
/diary                # Write diary
/history              # View conversation history
```

**Key functions:**
- `ensure_initialized()` - Lazy initialization of core components
- `_build_system_prompt()` - Constructs AI system prompt from personality + growth + diary context
- `_generate_response()` / `_generate_streaming_response()` - LLM API calls
- `_handle_slash_command()` - Process in-chat commands

---

#### 2. Core Modules (`huaqi/core/`)

##### a. Configuration (`config_simple.py`)
```python
class ConfigManager:
    - Stores config in memory/config.yaml
    - Pydantic models: AppConfig, LLMProviderConfig, MemoryConfig
    - Methods: load_config(), save_config(), get(), set()
```

##### b. Personality Engine (`personality_simple.py`)
```python
@dataclass
class PersonalityProfile:
    # Big Five personality traits
    openness, conscientiousness, extraversion, agreeableness, neuroticism
    # Communication style
    tone, formality, verbosity, humor, empathy
    # Values and role
    values: List[str], role: str, relationship: str

class PersonalityEngine:
    - PRESETS: companion, mentor, friend, assistant
    - save(), apply_preset(), update(), to_prompt()
```

##### c. LLM Layer (`llm.py`)
```python
# Abstractions:
class Message: system(), user(), assistant()
class LLMResponse: content, model, usage, latency_ms
class LLMConfig: provider, model, api_key, api_base, temperature

# Providers:
class OpenAIProvider(BaseLLMProvider)
class ClaudeProvider(BaseLLMProvider)
class DummyProvider(BaseLLMProvider)

# Manager:
class LLMManager:
    - register_provider(), add_config(), set_active()
    - chat(), quick_chat()
    - Supports both streaming and non-streaming
```

##### d. Growth System (`growth_simple.py`)
```python
@dataclass
class Skill: name, category, current_level, target_level, total_hours
@dataclass
class Goal: id, title, description, status, progress

class GrowthTracker:
    # Skills
    - add_skill(), get_skill(), log_practice(), list_skills()
    # Goals
    - add_goal(), update_goal(), complete_goal(), delete_goal()
```

##### e. Diary System (`diary_simple.py`)
```python
@dataclass
class DiaryEntry: date, content, mood, tags

class DiaryStore:
    - save(), get(), list_entries(), search(), delete()
    - get_recent_context()  # For AI context
    - import_from_markdown() # Batch import
```

##### f. Hooks (`hooks_simple.py`)
```python
class TriggerType: SCHEDULE, EVENT, CONDITION
class EventType: CONVERSATION_STARTED, MEMORY_CREATED, DAILY, WEEKLY

class HookManager:
    - DEFAULT_HOOKS: morning-greeting, daily-summary
    - create_hook(), update_hook(), delete_hook(), enable(), disable()
```

##### g. Git Auto Commit (`git_auto_commit.py`)
```python
class GitAutoCommit:
    - init_repo(), set_remote(), commit(), push(), pull()
    # Quick commits:
    - commit_personality_change()
    - commit_hook_change()
    - commit_skill_change()
    - commit_memory_change()
```

---

#### 3. Memory System (`huaqi/memory/`)

##### a. Storage (`storage/markdown_store.py`)
```python
class MarkdownMemoryStore:
    # Stores conversations in Markdown with YAML frontmatter
    # Directory structure: YYYY/MM/YYYYMMDD_HHMMSS_session_id.md
    
    - save_conversation()
    - load_conversation()
    - list_conversations()
    - search_conversations()
    - export_conversation()  # JSON, HTML, TXT formats
```

##### b. Search (`search/`)
Three search algorithms:

```python
# 1. Text Search (text_search.py)
class TFIDFSearch:  # TF-IDF algorithm
class BM25Search:   # BM25 algorithm (better for short text)
class TextSearch:   # Unified interface

# 2. LLM Search (llm_search.py)
class LLMRelevanceSearch:
    - Uses LLM to judge relevance (no embedding needed)
    - Returns score + reason
    - Falls back to keyword matching

# 3. Hybrid Search (hybrid_search.py)
class HybridSearch:
    # Two-stage search:
    # 1. BM25 for fast recall (Top 20)
    # 2. LLM for precise ranking (Top 5)
    - search(), search_with_explanation()
```

---

### Data Storage Structure

```
/Users/lianzimeng/workspace/huaqi/
├── memory/
│   ├── config.yaml           # App configuration
│   ├── personality.yaml      # AI personality settings
│   ├── growth.yaml           # Skills & goals
│   ├── hooks.yaml            # Automation hooks
│   ├── diary/                # Journal entries (YYYY/MM/YYYY-MM-DD.md)
│   └── conversations/        # Chat history
├── users/
│   └── f22ad49319a36ac8.json # User profiles
└── users_data/
    └── f22ad49319a36ac8/
        ├── config/
        ├── memory/
        │   ├── context/
        │   ├── conversations/
        │   └── patterns/
        └── vectors/
```

---

### Current Implementation Status

| Component | Status | Notes |
|-----------|--------|-------|
| CLI Entry Point | ✅ Complete | ~1000 lines, feature-rich |
| Config Management | ✅ Complete | Pydantic-based |
| LLM Layer | ✅ Complete | OpenAI, Claude, Dummy |
| Personality Engine | ✅ Complete | Customizable traits |
| Growth Tracker | ✅ Complete | Skills + Goals |
| Diary System | ✅ Complete | Markdown + search |
| Hook System | ✅ Complete | Schedule + Event triggers |
| Git Auto Sync | ✅ Complete | Auto commit + push |
| Memory Storage | ✅ Complete | Markdown format |
| Text Search | ✅ Complete | TF-IDF + BM25 |
| Hybrid Search | ✅ Complete | BM25 + LLM |
| Memory Layers | ❌ Empty | Planned: session/working/long_term |
| Orchestration | ❌ Empty | Planned: agents, pipeline |
| Security | ❌ Empty | Planned: constitution, guardrails |
| Skills/MCP | ❌ Empty | Planned: external tools |

---

### Key Architecture Patterns

1. **Single User Mode**: All modules have `_simple.py` suffix, designed for single-user local use

2. **Lazy Initialization**: Components initialized only when needed via `ensure_initialized()`

3. **Markdown-First Storage**: All data in human-readable format for:
   - Git version control
   - Direct editing
   - Obsidian compatibility

4. **Provider Pattern**: LLM providers inherit from `BaseLLMProvider`, making it easy to add new providers

5. **No Embedding Dependency**: Search uses BM25 + LLM judgment instead of vector embeddings

---

### Refactoring Opportunities for Code Agent Architecture

Based on the analysis, here are key areas that would benefit from refactoring:

1. **Monolithic CLI**: The `cli.py` is ~1000 lines with mixed concerns:
   - Could separate into: `chat.py`, `commands/`, `ui/`

2. **Missing Orchestration Layer**: The `orchestration/` directory is empty but critical for:
   - Agent-based architecture
   - Task planning and execution
   - Multi-step workflows

3. **Planned but Unimplemented**:
   - `memory/layers/` - Session/Working/Long-term memory
   - `security/` - Input/output guardrails
   - `skills/` - MCP-based tool integration

4. **Data Flow**:
   - Currently linear: User → CLI → LLM → Response
   - Could become: User → Agent → Tools → Memory → LLM → Response

This architecture is well-designed for a personal AI companion but would need significant refactoring to become a code agent system with autonomous task execution, tool use, and planning capabilities.
