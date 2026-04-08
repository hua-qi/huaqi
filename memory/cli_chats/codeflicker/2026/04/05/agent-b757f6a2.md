---
session_id: agent-b757f6a2
date: 2026-04-05
time_start: 2026-04-05T12:34:57.685Z
time_end: 2026-04-05T12:37:29.467Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-04-05 20:34:57] [user]: 请详细阅读以下文件并返回完整内容：
1. huaqi_src/layers/capabilities/reports/providers/ 目录下所有文件
2. 任何与 WorldProvider、WorldNewsStorage、world news 相关的文件
3. huaqi_src/layers/capabilities/pipeline/platforms/xiaohongshu.py 完整内容
4. huaqi_src/layers/capabilities/pipeline/core.py 完整内容
5. huaqi_src/layers/capabilities/pipeline/models.py 完整内容
6. ContentItem 数据模型的字段定义

请返回每个文件的完整代码。
[2026-04-05 20:35:01] [assistant]: 我来为您详细阅读这些文件。首先让我逐个查看这些文件和目录。
[2026-04-05 20:35:05] [assistant]: 现在让我读取 providers 目录下的所有文件：
[2026-04-05 20:35:09] [assistant]: 完美！现在让我搜索关于 WorldNewsStorage 和世界新闻的相关文件：
[2026-04-05 20:35:13] [assistant]: 现在读取世界新闻相关的文件：
[2026-04-05 20:35:16] [assistant]: 现在让我查找 HuaqiDocument 数据模型的定义：
[2026-04-05 20:35:20] [assistant]: 完美！现在让我查看一下是否有关于世界新闻的其他关键文件：
[2026-04-05 20:35:23] [assistant]: 现在让我查看 __init__.py 文件来看看模块结构：
[2026-04-05 20:35:25] [assistant]: 让我再查看一下小红书发布器的base class：
[2026-04-05 20:35:29] [assistant]: 现在让我查看处理器的完整内容：
[2026-04-05 20:35:34] [assistant]: 现在让我读取这些关键文件：
[2026-04-05 20:35:38] [assistant]: 现在让我查看 processors 和 sources 的 __init__.py 文件：
[2026-04-05 20:35:40] [assistant]: 完美！现在让我创建一份详细的总结文档：
[2026-04-05 20:37:29] [assistant]: 完美！现在我有了所有信息。让我为您创建一个完整的总结文档：

---

# 完整代码阅读文档

## 目录
1. [Reports Providers 目录下的所有文件](#1-reports-providers-目录下的所有文件)
2. [World 相关文件](#2-world-相关文件)
3. [Pipeline 核心文件](#3-pipeline-核心文件)
4. [数据模型](#4-数据模型)

---

## 1. Reports Providers 目录下的所有文件

### 1.1 `__init__.py` - 数据提供者基类和注册表
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import date


@dataclass
class DateRange:
    start: date
    end: date


class DataProvider(ABC):
    name: str
    priority: int = 50
    supported_reports: list

    @abstractmethod
    def get_context(self, report_type: str, date_range: "DateRange") -> "str | None":
        pass


_registry: list = []


def register(provider: DataProvider) -> None:
    _registry.append(provider)


def get_providers(report_type: str) -> list:
    return sorted(
        [p for p in _registry if report_type in p.supported_reports or "*" in p.supported_reports],
        key=lambda p: p.priority,
    )
```

**功能**: 
- 定义 `DataProvider` 抽象基类，所有数据提供者必须继承
- 提供 `DateRange` 数据类，表示时间范围
- 实现全局提供者注册表，支持按报告类型检索

---

### 1.2 `diary.py` - 日记内容提供者
```python
import datetime
from pathlib import Path
from typing import Optional

from huaqi_src.layers.capabilities.reports.providers import DataProvider, DateRange, register


class DiaryProvider(DataProvider):
    name = "diary"
    priority = 20
    supported_reports = ["*"]

    def __init__(self, data_dir: Optional[Path] = None):
        if data_dir is not None:
            self._data_dir = Path(data_dir)
        else:
            from huaqi_src.config.paths import require_data_dir
            self._data_dir = require_data_dir()

    def get_context(self, report_type: str, date_range: DateRange) -> "str | None":
        diary_dir = self._data_dir / "memory" / "diary"
        if not diary_dir.exists():
            return None

        if report_type == "daily":
            month_str = date_range.end.strftime("%Y-%m")
            diary_file = diary_dir / month_str / f"{date_range.end.isoformat()}.md"
            if not diary_file.exists():
                diary_file = diary_dir / f"{date_range.end.isoformat()}.md"
            if not diary_file.exists():
                return None
            content = diary_file.read_text(encoding="utf-8")[:800]
            return f"## 今日日记\n{content}"

        snippets = []
        current = date_range.end
        while current >= date_range.start:
            month_str = current.strftime("%Y-%m")
            f = diary_dir / month_str / f"{current.isoformat()}.md"
            if not f.exists():
                f = diary_dir / f"{current.isoformat()}.md"
            
            if f.exists():
                content = f.read_text(encoding="utf-8")[:300]
                snippets.append(f"### {current.isoformat()}\n{content}")
            
            current -= datetime.timedelta(days=1)
            if len(snippets) >= 7:
                break

        if not snippets:
            return None
        label = "近期日记片段" if report_type == "morning" else "本周日记片段"
        return f"## {label}\n" + "\n\n".join(snippets)


try:
    from huaqi_src.config.paths import get_data_dir
    _data_dir = get_data_dir()
    if _data_dir is not None:
        register(DiaryProvider(_data_dir))
except Exception:
    pass
```

**功能**:
- 优先级: 20 (中等)
- 支持所有报告类型 (priority 20 意味着较早被使用)
- 从文件系统中读取日记文件
- 日记目录结构支持: `memory/diary/YYYY-MM/YYYY-MM-DD.md` 或 `memory/diary/YYYY-MM-DD.md`
- 返回不同格式的日记上下文取决于报告类型

---

### 1.3 `events.py` - 事件流提供者
```python
import sqlite3
import time
from pathlib import Path
from typing import Optional

from huaqi_src.layers.capabilities.reports.providers import DataProvider, DateRange, register


class EventsProvider(DataProvider):
    name = "events"
    priority = 60
    supported_reports = ["daily", "weekly"]

    def __init__(self, data_dir: Optional[Path] = None):
        if data_dir is not None:
            self._data_dir = Path(data_dir)
        else:
            from huaqi_src.config.paths import require_data_dir
            self._data_dir = require_data_dir()

    def get_context(self, report_type: str, date_range: DateRange) -> "str | None":
        db_path = self._data_dir / "events.db"
        if not db_path.exists():
            return None

        import datetime as dt
        start_ts = int(dt.datetime.combine(date_range.start, dt.time.min).timestamp())
        end_ts = int(dt.datetime.combine(date_range.end, dt.time.max).timestamp())

        try:
            conn = sqlite3.connect(str(db_path))
            rows = conn.execute(
                "SELECT source, actor, content, timestamp FROM events "
                "WHERE timestamp >= ? AND timestamp <= ? "
                "ORDER BY timestamp DESC LIMIT 20",
                (start_ts, end_ts),
            ).fetchall()
            conn.close()
        except Exception:
            return None

        if not rows:
            return None

        lines = ["## 近期事件流"]
        for row in rows:
            source, actor, content, ts = row
            import datetime as dt2
            date_str = dt2.datetime.fromtimestamp(ts).strftime("%Y-%m-%d %H:%M") if ts else ""
            lines.append(f"- [{date_str}] {source}/{actor}：{(content or '')[:100]}")

        return "\n".join(lines)


try:
    from huaqi_src.config.paths import get_data_dir
    _data_dir = get_data_dir()
    if _data_dir is not None:
        register(EventsProvider(_data_dir))
except Exception:
    pass
```

**功能**:
- 优先级: 60 (较晚使用，因为数据库查询可能较慢)
- 仅支持 daily 和 weekly 报告
- 从 SQLite 数据库查询事件
- 数据库表结构: `events (source, actor, content, timestamp)`
- 查询最近 20 条事件

---

### 1.4 `growth.py` - 成长目标和技能提供者
```python
from pathlib import Path
from typing import Optional

from huaqi_src.layers.capabilities.reports.providers import DataProvider, DateRange, register


class GrowthProvider(DataProvider):
    name = "growth"
    priority = 50
    supported_reports = ["weekly", "quarterly"]

    def __init__(self, data_dir: Optional[Path] = None):
        if data_dir is not None:
            self._data_dir = Path(data_dir)
        else:
            from huaqi_src.config.paths import require_data_dir
            self._data_dir = require_data_dir()

    def get_context(self, report_type: str, date_range: DateRange) -> "str | None":
        growth_file = self._data_dir / "memory" / "growth.yaml"
        if not growth_file.exists():
            return None

        import yaml
        with open(growth_file, encoding="utf-8") as f:
            data = yaml.safe_load(f)
        if not data:
            return None

        lines = ["## 成长目标与技能"]
        goals = data.get("goals", [])
        if goals:
            lines.append("### 目标")
            for g in goals:
                title = g.get("title", "")
                status = g.get("status", "")
                progress = g.get("progress", "")
                line = f"- {title}（{status}）"
                if progress:
                    line += f" {progress}"
                lines.append(line)

        skills = data.get("skills", [])
        if skills:
            lines.append("### 技能")
            for s in skills:
                lines.append(f"- {s.get('name', '')}：{s.get('level', '')}")

        return "\n".join(lines)


try:
    from huaqi_src.config.paths import get_data_dir
    _data_dir = get_data_dir()
    if _data_dir is not None:
        register(GrowthProvider(_data_dir))
except Exception:
    pass
```

**功能**:
- 优先级: 50 (中等)
- 仅支持 weekly 和 quarterly 报告
- 从 YAML 文件读取成长数据
- 文件路径: `memory/growth.yaml`
- 解析目标列表和技能列表

---

### 1.5 `learning.py` - 学习进度提供者
```python
import datetime
from pathlib import Path
from typing import Optional

from huaqi_src.layers.capabilities.reports.providers import DataProvider, DateRange, register


class LearningProvider(DataProvider):
    name = "learning"
    priority = 30
    supported_reports = ["daily", "weekly", "quarterly"]

    def __init__(self, data_dir: Optional[Path] = None):
        if data_dir is not None:
            self._data_dir = Path(data_dir)
        else:
            from huaqi_src.config.paths import require_data_dir
            self._data_dir = require_data_dir()

    def get_context(self, report_type: str, date_range: DateRange) -> "str | None":
        learning_dir = self._data_dir / "learning"
        if not learning_dir.exists():
            return None

        from huaqi_src.layers.capabilities.learning.progress_store import LearningProgressStore
        store = LearningProgressStore(base_dir=learning_dir)
        courses = store.list_courses()
        if not courses:
            return None

        lines = ["## 学习进度"]
        for course in courses:
            completed = sum(1 for l in course.lessons if l.status == "completed")
            total = course.total_lessons
            current_lesson = next(
                (l for l in course.lessons if l.index == course.current_lesson), None
            )
            current_title = current_lesson.title if current_lesson else "—"
            lines.append(
                f"- **{course.skill_name}**：{completed}/{total} 章完成，当前：{current_title}"
            )

        sessions_dir = learning_dir / "sessions"
        if sessions_dir.exists() and report_type in ("daily", "weekly"):
            recent_sessions = []
            current = date_range.end
            while current >= date_range.start:
                date_str = current.strftime("%Y%m%d")
                for f in sessions_dir.glob(f"{date_str}_*.md"):
                    recent_sessions.append(f.read_text(encoding="utf-8")[:200])
                current -= datetime.timedelta(days=1)
            if recent_sessions:
                lines.append("\n## 近期学习记录")
                lines.extend(recent_sessions[:3])

        return "\n".join(lines)


try:
    from huaqi_src.config.paths import get_data_dir
    _data_dir = get_data_dir()
    if _data_dir is not None:
        register(LearningProvider(_data_dir))
except Exception:
    pass
```

**功能**:
- 优先级: 30 (较早使用)
- 支持所有三种报告: daily, weekly, quarterly
- 使用 `LearningProgressStore` 从课程目录读取学习进度
- 文件结构: `learning/courses/` 和 `learning/sessions/YYYYMMDD_*.md`
- 收集最近学习的课程进度和学习记录

---

### 1.6 `people.py` - 关系人提供者
```python
from pathlib import Path
from typing import Optional

from huaqi_src.layers.capabilities.reports.providers import DataProvider, DateRange, register


class PeopleProvider(DataProvider):
    name = "people"
    priority = 40
    supported_reports = ["*"]

    def __init__(self, data_dir: Optional[Path] = None):
        if data_dir is not None:
            self._data_dir = Path(data_dir)
        else:
            from huaqi_src.config.paths import require_data_dir
            self._data_dir = require_data_dir()

    def get_context(self, report_type: str, date_range: DateRange) -> "str | None":
        people_dir = self._data_dir / "people"
        if not people_dir.exists():
            return None

        from huaqi_src.layers.growth.telos.dimensions.people.graph import PeopleGraph
        graph = PeopleGraph(data_dir=self._data_dir)
        people = graph.list_people()
        if not people:
            return None

        active = [p for p in people if p.interaction_frequency > 0]
        if not active:
            active = people

        active.sort(key=lambda p: p.interaction_frequency, reverse=True)

        if report_type == "morning":
            lines = ["## 近期活跃关系人"]
            for p in active[:3]:
                line = f"- {p.name}（{p.relation_type}）"
                if p.notes:
                    line += f"：{p.notes}"
                lines.append(line)
        elif report_type == "daily":
            lines = ["## 关系网络动态"]
            for p in active[:5]:
                lines.append(f"- {p.name}（{p.relation_type}）：近30天互动 {p.interaction_frequency} 次")
        elif report_type == "weekly":
            lines = ["## 关系人概览"]
            for p in active[:8]:
                line = f"- {p.name}（{p.relation_type}）"
                if p.profile:
                    line += f"：{p.profile[:50]}"
                lines.append(line)
        else:
            lines = ["## 关系人全貌"]
            for p in sorted(people, key=lambda x: x.interaction_frequency, reverse=True):
                line = f"- {p.name}（{p.relation_type}，{p.emotional_impact}影响）"
                if p.profile:
                    line += f"：{p.profile[:80]}"
                lines.append(line)

        return "\n".join(lines)


try:
    from huaqi_src.config.paths import get_data_dir
    _data_dir = get_data_dir()
    if _data_dir is not None:
        register(PeopleProvider(_data_dir))
except Exception:
    pass
```

**功能**:
- 优先级: 40 (中等)
- 支持所有报告类型 ("*")
- 使用 `PeopleGraph` 从关系图数据库读取人物信息
- 人物属性: name, relation_type, interaction_frequency, profile, notes, emotional_impact
- 根据不同报告类型返回不同数量和格式的关系人信息

---

### 1.7 `weekly_reports.py` - 周报摘要提供者
```python
from pathlib import Path
from typing import Optional

from huaqi_src.layers.capabilities.reports.providers import DataProvider, DateRange, register


class WeeklyReportsProvider(DataProvider):
    name = "weekly_reports"
    priority = 70
    supported_reports = ["quarterly"]

    def __init__(self, data_dir: Optional[Path] = None):
        if data_dir is not None:
            self._data_dir = Path(data_dir)
        else:
            from huaqi_src.config.paths import require_data_dir
            self._data_dir = require_data_dir()

    def get_context(self, report_type: str, date_range: DateRange) -> "str | None":
        weekly_dir = self._data_dir / "reports" / "weekly"
        if not weekly_dir.exists():
            return None

        weekly_files = sorted(weekly_dir.glob("*.md"), reverse=True)[:13]
        if not weekly_files:
            return None

        snippets = []
        for f in weekly_files:
            snippets.append(f"### {f.stem}\n{f.read_text(encoding='utf-8')[:200]}")

        return "## 本季度周报摘要\n" + "\n\n".join(snippets)


try:
    from huaqi_src.config.paths import get_data_dir
    _data_dir = get_data_dir()
    if _data_dir is not None:
        register(WeeklyReportsProvider(_data_dir))
except Exception:
    pass
```

**功能**:
- 优先级: 70 (最后被使用)
- 仅支持 quarterly 报告
- 从 `reports/weekly/` 目录读取周报文件
- 最多收集最近 13 周的周报摘要

---

### 1.8 `world.py` - 世界热点提供者 (WorldProvider)
```python
from pathlib import Path
from typing import Optional

from huaqi_src.layers.capabilities.reports.providers import DataProvider, DateRange, register


class WorldProvider(DataProvider):
    name = "world"
    priority = 10
    supported_reports = ["morning", "daily"]

    def __init__(self, data_dir: Optional[Path] = None):
        if data_dir is not None:
            self._data_dir = Path(data_dir)
        else:
            from huaqi_src.config.paths import require_data_dir
            self._data_dir = require_data_dir()

    def get_context(self, report_type: str, date_range: DateRange) -> "str | None":
        world_dir = self._data_dir / "world"
        if not world_dir.exists():
            return None
        today = date_range.end.isoformat()
        world_file = world_dir / f"{today}.md"
        if not world_file.exists():
            return None
        content = world_file.read_text(encoding="utf-8")[:1000]
        return f"## 今日世界热点\n{content}"


try:
    from huaqi_src.config.paths import get_data_dir
    _data_dir = get_data_dir()
    if _data_dir is not None:
        register(WorldProvider(_data_dir))
except Exception:
    pass
```

**功能**:
- **优先级: 10 (最高优先级，最先被使用)**
- 支持 morning 和 daily 报告
- 从 `world/YYYY-MM-DD.md` 文件读取当天的世界热点
- 返回前 1000 字的内容

---

## 2. World 相关文件

### 2.1 `huaqi_src/layers/data/world/base_source.py` - 世界新闻源基类
```python
from abc import ABC, abstractmethod
from huaqi_src.layers.data.collectors.document import HuaqiDocument


class BaseWorldSource(ABC):
    source_id: str = ""

    @abstractmethod
    def fetch(self) -> list[HuaqiDocument]:
        """抓取最新数据，返回 HuaqiDocument 列表"""
        ...
```

**功能**:
- 定义世界新闻数据源的抽象基类
- 所有世界新闻源必须实现 `fetch()` 方法
- 返回 `HuaqiDocument` 对象列表

---

### 2.2 `huaqi_src/layers/data/world/fetcher.py` - 世界新闻采集器
```python
from typing import Optional
from huaqi_src.layers.data.collectors.document import HuaqiDocument
from huaqi_src.layers.data.world.base_source import BaseWorldSource


class WorldNewsFetcher:
    def __init__(self, sources: Optional[list] = None):
        self.sources = sources or []

    def fetch_all(self) -> list[HuaqiDocument]:
        docs = []
        for source in self.sources:
            try:
                results = source.fetch()
                docs.extend(results)
            except Exception as e:
                print(f"[WorldNewsFetcher] 数据源 {source.source_id} 失败: {e}")
        return docs
```

**功能**:
- 聚合多个世界新闻源的采集器
- `fetch_all()` 方法从所有源并行采集新闻
- 捕获和处理单个源的异常，不影响整体流程

---

### 2.3 `huaqi_src/layers/data/world/storage.py` - 世界新闻存储 (WorldNewsStorage)
```python
import datetime
from pathlib import Path
from typing import Optional

from huaqi_src.layers.data.collectors.document import HuaqiDocument


class WorldNewsStorage:
    def __init__(self, data_dir: Optional[Path] = None):
        if data_dir is not None:
            self.world_dir = Path(data_dir) / "world"
        else:
            from huaqi_src.config.paths import get_world_dir
            self.world_dir = get_world_dir()
        self.world_dir.mkdir(parents=True, exist_ok=True)

    def save(self, docs: list[HuaqiDocument], date: Optional[datetime.date] = None):
        if not docs:
            return
        if date is None:
            date = datetime.date.today()
        file_path = self.world_dir / f"{date.isoformat()}.md"
        lines = [f"# 世界感知摘要 {date.isoformat()}\n"]
        for doc in docs:
            lines.append(f"## {doc.metadata.get('feed_name', doc.source)}\n")
            lines.append(f"{doc.content}\n")
            lines.append("---\n")
        file_path.write_text("\n".join(lines), encoding="utf-8")

    def search(self, query: str, days: int = 7) -> list[str]:
        query_lower = query.lower()
        results = []
        for md_file in sorted(self.world_dir.glob("*.md"), reverse=True)[:days]:
            content = md_file.read_text(encoding="utf-8")
            for section in content.split("---"):
                if query_lower in section.lower():
                    results.append(section.strip())
        return results
```

**功能**:
- 将采集的世界新闻存储到本地文件系统
- 文件格式: `world/YYYY-MM-DD.md`
- 文件内容格式: 按数据源分段，用 `---` 分隔
- `search()` 方法支持在最近N天的新闻中查询特定关键词

---

### 2.4 `huaqi_src/layers/data/world/sources/rss_source.py` - RSS 新闻源
```python
import datetime
import hashlib

import feedparser

from huaqi_src.layers.data.collectors.document import HuaqiDocument
from huaqi_src.layers.data.world.base_source import BaseWorldSource


class RSSSource(BaseWorldSource):
    def __init__(self, url: str, name: str = ""):
        self.url = url
        self.name = name or url
        self.source_id = f"rss:{url}"

    def fetch(self) -> list[HuaqiDocument]:
        feed = feedparser.parse(self.url)
        docs = []
        for entry in feed.entries[:20]:
            content = entry.get("summary", entry.get("title", ""))
            title = entry.get("title", "")
            full_content = f"# {title}\n\n{content}"
            doc_id = hashlib.md5(entry.get("link", title).encode()).hexdigest()[:12]
            published = entry.get("published_parsed")
            if published:
                ts = datetime.datetime(*published[:6])
            else:
                ts = datetime.datetime.now()
            docs.append(
                HuaqiDocument(
                    doc_id=doc_id,
                    doc_type="world_news",
                    source=f"rss:{self.url}",
                    content=full_content,
                    timestamp=ts,
                    metadata={"url": entry.get("link", ""), "feed_name": self.name},
                )
            )
        return docs
```

**功能**:
- 从 RSS Feed 抓取世界新闻
- 最多抓取单个 Feed 的前 20 条条目
- 将 RSS entry 转换为 `HuaqiDocument` 格式
- 保留原始链接和 Feed 名称在元数据中

---

### 2.5 `huaqi_src/layers/data/collectors/document.py` - HuaqiDocument 数据模型
```python
import datetime
from dataclasses import dataclass, field
from typing import Any


@dataclass
class HuaqiDocument:
    doc_id: str
    doc_type: str
    source: str
    content: str
    timestamp: datetime.datetime
    summary: str = ""
    people: list[str] = field(default_factory=list)
    metadata: dict[str, Any] = field(default_factory=dict)

    def to_dict(self) -> dict[str, Any]:
        return {
            "doc_id": self.doc_id,
            "doc_type": self.doc_type,
            "source": self.source,
            "content": self.content,
            "timestamp": self.timestamp.isoformat(),
            "summary": self.summary,
            "people": self.people,
            "metadata": self.metadata,
        }
```

**字段说明**:
- `doc_id` (str): 文档唯一ID (通常是 MD5 哈希)
- `doc_type` (str): 文档类型，如 "world_news"
- `source` (str): 数据源标识，如 "rss:https://example.com"
- `content` (str): 完整内容
- `timestamp` (datetime): 发布时间戳
- `summary` (str): 内容摘要 (可选)
- `people` (list[str]): 涉及的人物列表 (可选)
- `metadata` (dict): 额外元数据，如 "feed_name", "url" 等

---

## 3. Pipeline 核心文件

### 3.1 `huaqi_src/layers/capabilities/pipeline/models.py` - ContentItem 数据模型
```python
"""内容流水线数据模型"""

from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum
from typing import Dict, List, Optional, Any


class ContentSource(Enum):
    """内容来源"""
    X = "x"
    RSS = "rss"
    MANUAL = "manual"


class ContentStatus(Enum):
    """内容处理状态"""
    RAW = "raw"           # 原始采集
    SUMMARIZED = "summarized"  # 已摘要
    TRANSLATED = "translated"  # 已翻译
    READY = "ready"       # 可发布
    PUBLISHED = "published"   # 已发布
    FAILED = "failed"     # 处理失败


class PlatformType(Enum):
    """目标平台"""
    XIAOHONGSHU = "xiaohongshu"


@dataclass
class ContentItem:
    """内容条目"""
    id: str
    source: ContentSource
    title: str
    content: str
    url: Optional[str] = None
    author: Optional[str] = None
    published_at: Optional[datetime] = None
    status: ContentStatus = ContentStatus.RAW
    
    # 处理结果
    summary: Optional[str] = None
    translated_content: Optional[str] = None
    
    # 小红书格式
    xiaohongshu_content: Optional[str] = None
    xiaohongshu_tags: List[str] = field(default_factory=list)
    
    # 元数据
    metadata: Dict[str, Any] = field(default_factory=dict)
    created_at: datetime = field(default_factory=datetime.now)
    updated_at: datetime = field(default_factory=datetime.now)


@dataclass
class PipelineResult:
    """流水线处理结果"""
    item: ContentItem
    success: bool
    stage: str
    error: Optional[str] = None
    processed_at: datetime = field(default_factory=datetime.now)


@dataclass
class XiaoHongShuPost:
    """小红书发布内容"""
    title: str
    content: str
    tags: List[str] = field(default_factory=list)
    images: List[str] = field(default_factory=list)  # 图片路径
    original_url: Optional[str] = None
```

**ContentItem 字段完整说明**:
| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| id | str | 必需 | 内容唯一ID |
| source | ContentSource | 必需 | 内容来源 (X, RSS, MANUAL) |
| title | str | 必需 | 标题 |
| content | str | 必需 | 原始内容 |
| url | Optional[str] | None | 原始链接 |
| author | Optional[str] | None | 作者 |
| published_at | Optional[datetime] | None | 发布时间 |
| status | ContentStatus | RAW | 处理状态 |
| summary | Optional[str] | None | 摘要内容 (摘要器生成) |
| translated_content | Optional[str] | None | 翻译后的内容 (翻译器生成) |
| xiaohongshu_content | Optional[str] | None | 小红书格式内容 (格式器生成) |
| xiaohongshu_tags | List[str] | [] | 小红书标签列表 |
| metadata | Dict[str, Any] | {} | 额外元数据 |
| created_at | datetime | now() | 创建时间 |
| updated_at | datetime | now() | 更新时间 |

---

### 3.2 `huaqi_src/layers/capabilities/pipeline/core.py` - 完整内容
```python
"""内容流水线核心

负责协调数据源、处理器和发布平台的完整流程
"""

import asyncio
from datetime import datetime
from pathlib import Path
from typing import List, Optional, Dict, Any

from .models import ContentItem, ContentStatus, PipelineResult
from .sources import BaseSource, XMockSource, RSSMockSource
from .processors import BaseProcessor, Summarizer, Translator, XiaoHongShuFormatter
from .platforms import BasePublisher, XiaoHongShuPublisher


class ContentPipeline:
    """内容处理流水线
    
    流程: 采集 -> 摘要 -> 翻译 -> 格式化 -> 发布
    """
    
    def __init__(
        self,
        sources: Optional[List[BaseSource]] = None,
        processors: Optional[List[BaseProcessor]] = None,
        publisher: Optional[BasePublisher] = None,
    ):
        self.sources = sources or []
        self.processors = processors or []
        self.publisher = publisher
        
        # 统计信息
        self.stats = {
            "fetched": 0,
            "processed": 0,
            "published": 0,
            "failed": 0,
        }
    
    async def run(
        self,
        since: Optional[datetime] = None,
        limit: int = 10,
        dry_run: bool = False,
    ) -> Dict[str, Any]:
        """执行完整流水线
        
        Args:
            since: 只处理此时间之后的内容
            limit: 每个源最大采集数量
            dry_run: 是否只预览不发布
            
        Returns:
            执行统计
        """
        print("=" * 50)
        print("🚀 启动内容流水线")
        print("=" * 50)
        
        # 1. 采集内容
        print("\n📥 步骤 1: 采集内容")
        items = await self._fetch_all(since, limit)
        print(f"   共采集 {len(items)} 条内容")
        
        if not items:
            print("\n⚠️ 没有新内容，流水线结束")
            return self.stats
        
        # 2. 处理内容
        print("\n🔧 步骤 2: 处理内容")
        processed = await self._process_all(items)
        print(f"   处理完成: {len([r for r in processed if r.success])}/{len(processed)}")
        
        # 3. 发布内容
        if self.publisher:
            ready_items = [
                r.item for r in processed
                if r.success and r.item.status == ContentStatus.READY
            ]
            
            if dry_run:
                # 预览模式：保存草稿但不发布
                print("\n📝 步骤 3: [预览模式] 保存草稿")
                await self._publish_all(ready_items, dry_run=True)
            else:
                print("\n📤 步骤 3: 发布内容")
                published = await self._publish_all(ready_items, dry_run=False)
                print(f"   发布完成: {published}/{len(ready_items)}")
        
        # 输出统计
        print("\n" + "=" * 50)
        print("📊 流水线统计")
        print("=" * 50)
        for key, value in self.stats.items():
            print(f"   {key}: {value}")
        
        return self.stats
    
    async def _fetch_all(
        self,
        since: Optional[datetime],
        limit: int,
    ) -> List[ContentItem]:
        """从所有源采集内容"""
        all_items = []
        
        for source in self.sources:
            try:
                items = await source.fetch(since=since, limit=limit)
                all_items.extend(items)
                print(f"   ✓ {source.source_type.value}: {len(items)} 条")
            except Exception as e:
                print(f"   ✗ {source.source_type.value}: 失败 - {e}")
        
        self.stats["fetched"] = len(all_items)
        return all_items
    
    async def _process_all(self, items: List[ContentItem]) -> List[PipelineResult]:
        """处理所有内容"""
        results = []
        
        for item in items:
            result = await self._process_item(item)
            results.append(result)
            
            if result.success:
                self.stats["processed"] += 1
            else:
                self.stats["failed"] += 1
        
        return results
    
    async def _process_item(self, item: ContentItem) -> PipelineResult:
        """处理单个内容"""
        for processor in self.processors:
            try:
                result = await processor.process(item)
                if not result.success:
                    return result
            except Exception as e:
                return PipelineResult(
                    item=item,
                    success=False,
                    stage=processor.name,
                    error=str(e),
                )
        
        return PipelineResult(
            item=item,
            success=True,
            stage="complete",
        )
    
    async def _publish_all(self, items: List[ContentItem], dry_run: bool = False) -> int:
        """发布所有内容
        
        Args:
            items: 内容列表
            dry_run: 如果为 True，只保存草稿不发布
        """
        published = 0
        
        for item in items:
            try:
                # 检查 publisher 是否支持 dry_run 参数
                if hasattr(self.publisher, 'publish') and 'dry_run' in self.publisher.publish.__code__.co_varnames:
                    if await self.publisher.publish(item, dry_run=dry_run):
                        published += 1
                        if not dry_run:
                            self.stats["published"] += 1
                else:
                    # 不支持 dry_run 的 publisher
                    if not dry_run:
                        if await self.publisher.publish(item):
                            published += 1
                            self.stats["published"] += 1
            except Exception as e:
                print(f"   ✗ 发布失败 [{item.id}]: {e}")
        
        return published
    
    def preview(self, item: ContentItem) -> str:
        """预览内容"""
        return item.xiaohongshu_content or item.content


class PipelineBuilder:
    """流水线构建器"""
    
    def __init__(self):
        self.sources: List[BaseSource] = []
        self.processors: List[BaseProcessor] = []
        self.publisher: Optional[BasePublisher] = None
    
    def add_x_source(self, mock: bool = True, **config) -> "PipelineBuilder":
        """添加 X 数据源"""
        if mock:
            self.sources.append(XMockSource(config))
        else:
            from .sources import XSource
            self.sources.append(XSource(config))
        return self
    
    def add_rss_source(self, mock: bool = True, **config) -> "PipelineBuilder":
        """添加 RSS 数据源"""
        if mock:
            self.sources.append(RSSMockSource(config))
        else:
            from .sources import RSSSource
            self.sources.append(RSSSource(config))
        return self
    
    def add_summarizer(self, max_length: int = 500) -> "PipelineBuilder":
        """添加摘要处理器"""
        self.processors.append(Summarizer(max_length=max_length))
        return self
    
    def add_translator(self, use_llm: bool = True) -> "PipelineBuilder":
        """添加翻译处理器"""
        self.processors.append(Translator(use_llm=use_llm))
        return self
    
    def add_xiaohongshu_formatter(self) -> "PipelineBuilder":
        """添加小红书格式器"""
        self.processors.append(XiaoHongShuFormatter())
        return self
    
    def set_xiaohongshu_publisher(self, **config) -> "PipelineBuilder":
        """设置小红书发布器"""
        self.publisher = XiaoHongShuPublisher(config)
        return self
    
    def build(self) -> ContentPipeline:
        """构建流水线"""
        return ContentPipeline(
            sources=self.sources,
            processors=self.processors,
            publisher=self.publisher,
        )


# 默认流水线配置
def create_default_pipeline(
    draft_dir: Optional[str] = None,
    mock_sources: bool = True,
) -> ContentPipeline:
    """创建默认流水线
    
    默认流程: X/RSS -> 摘要 -> 翻译 -> 小红书格式 -> 草稿
    """
    builder = PipelineBuilder()
    
    # 数据源
    builder.add_x_source(mock=mock_sources)
    builder.add_rss_source(mock=mock_sources)
    
    # 处理器
    builder.add_summarizer(max_length=500)
    builder.add_translator(use_llm=False)  # 简化版本
    builder.add_xiaohongshu_formatter()
    
    # 发布器
    config = {}
    if draft_dir:
        config["draft_dir"] = draft_dir
    builder.set_xiaohongshu_publisher(**config)
    
    return builder.build()
```

---

### 3.3 `huaqi_src/layers/capabilities/pipeline/platforms/xiaohongshu.py` - 完整内容
```python
"""小红书发布器

支持两种模式:
1. 真实发布: 通过小红书 API 或模拟浏览器发布 (需登录)
2. 草稿模式: 生成内容到本地，手动复制发布
"""

import os
import json
from datetime import datetime
from pathlib import Path
from typing import Optional

from .base import BasePublisher
from ..models import ContentItem, ContentStatus


class XiaoHongShuPublisher(BasePublisher):
    """小红书内容发布器"""
    
    platform_name = "xiaohongshu"
    
    def __init__(self, config: dict = None):
        super().__init__(config)
        from huaqi_src.config.paths import get_drafts_dir
        self.draft_dir = Path(self.config.get("draft_dir", str(get_drafts_dir())))
        self.draft_dir = self.draft_dir.expanduser() if str(self.draft_dir).startswith("~") else self.draft_dir
        self.draft_dir.mkdir(parents=True, exist_ok=True)
        
        # 真实发布配置 (可选)
        self.api_token = self.config.get("api_token") or os.getenv("XIAOHONGSHU_TOKEN")
        self.enable_auto_publish = self.config.get("auto_publish", False) and bool(self.api_token)
    
    async def publish(self, item: ContentItem, dry_run: bool = False) -> bool:
        """发布内容到小红书
        
        当前实现: 保存为草稿文件
        未来: 支持 API 自动发布
        
        Args:
            item: 内容条目
            dry_run: 如果为 True，只保存草稿不更新状态
        """
        try:
            if self.enable_auto_publish and not dry_run:
                return await self._auto_publish(item)
            else:
                return await self._save_draft(item, dry_run=dry_run)
                
        except Exception as e:
            print(f"[XiaoHongShu] 发布失败: {e}")
            item.status = ContentStatus.FAILED
            return False
    
    async def _save_draft(self, item: ContentItem, dry_run: bool = False) -> bool:
        """保存为草稿文件
        
        Args:
            item: 内容条目
            dry_run: 如果为 True，不更新 item 状态
        """
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        filename = f"{timestamp}_{item.id[:8]}.md"
        filepath = self.draft_dir / filename
        
        # 构建草稿内容
        content = self._build_draft(item)
        
        # 保存文件
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(content)
        
        # 同时保存元数据
        meta_filepath = self.draft_dir / f"{filename}.json"
        metadata = {
            "item_id": item.id,
            "source": item.source.value,
            "original_url": item.url,
            "title": item.title,
            "tags": item.xiaohongshu_tags,
            "created_at": datetime.now().isoformat(),
            "status": "draft",
        }
        with open(meta_filepath, 'w', encoding='utf-8') as f:
            json.dump(metadata, f, ensure_ascii=False, indent=2)
        
        if not dry_run:
            item.status = ContentStatus.PUBLISHED  # 标记为已处理
        print(f"[XiaoHongShu] 草稿已保存: {filepath}")
        return True
    
    async def _auto_publish(self, item: ContentItem) -> bool:
        """自动发布到小红书 (待实现)"""
        # TODO: 实现小红书 API 发布
        print("[XiaoHongShu] 自动发布功能待实现，保存为草稿")
        return await self._save_draft(item)
    
    def _build_draft(self, item: ContentItem) -> str:
        """构建草稿内容"""
        lines = [
            f"# {item.title}",
            "",
            "---",
            "",
            "## 小红书内容",
            "",
            item.xiaohongshu_content or item.content,
            "",
            "---",
            "",
            "## 元数据",
            "",
            f"- **来源**: {item.source.value}",
            f"- **原文链接**: {item.url or 'N/A'}",
            f"- **作者**: {item.author or 'N/A'}",
            "",
            "## 标签",
            "",
        ]
        
        for tag in item.xiaohongshu_tags:
            lines.append(f"- {tag}")
        
        lines.extend([
            "",
            "---",
            "",
            "## 操作说明",
            "",
            "1. 复制上方的【小红书内容】",
            "2. 打开小红书 APP",
            "3. 创建新笔记，粘贴内容",
            "4. 选择合适的图片",
            "5. 点击发布",
            "",
        ])
        
        return "\n".join(lines)
    
    async def health_check(self) -> bool:
        """检查发布器状态"""
        # 检查草稿目录是否可写
        try:
            test_file = self.draft_dir / ".health_check"
            test_file.write_text("ok")
            test_file.unlink()
            return True
        except:
            return False
    
    def list_drafts(self) -> list:
        """列出所有草稿"""
        drafts = []
        for f in self.draft_dir.glob("*.md"):
            if not f.name.endswith(".json.md"):
                drafts.append({
                    "filename": f.name,
                    "path": str(f),
                    "created": datetime.fromtimestamp(f.stat().st_mtime).isoformat(),
                })
        return sorted(drafts, key=lambda x: x["created"], reverse=True)
```

---

### 3.4 处理器文件

#### 3.4.1 `processors/base.py`
```python
"""处理器基类"""

from abc import ABC, abstractmethod
from typing import List

from ..models import ContentItem, PipelineResult


class BaseProcessor(ABC):
    """内容处理器基类"""
    
    name: str = "base"
    
    @abstractmethod
    async def process(self, item: ContentItem) -> PipelineResult:
        """处理内容
        
        Args:
            item: 内容条目
            
        Returns:
            处理结果
        """
        pass
    
    async def process_batch(self, items: List[ContentItem]) -> List[PipelineResult]:
        """批量处理"""
        results = []
        for item in items:
            try:
                result = await self.process(item)
                results.append(result)
            except Exception as e:
                results.append(PipelineResult(
                    item=item,
                    success=False,
                    stage=self.name,
                    error=str(e),
                ))
        return results
```

#### 3.4.2 `processors/summarizer.py`
```python
"""内容摘要处理器"""

from ..models import ContentItem, ContentStatus, PipelineResult
from .base import BaseProcessor


class Summarizer(BaseProcessor):
    """生成内容摘要"""
    
    name = "summarizer"
    
    def __init__(self, max_length: int = 500):
        self.max_length = max_length
    
    async def process(self, item: ContentItem) -> PipelineResult:
        """生成摘要"""
        try:
            # 简单提取前几句作为摘要
            # 实际应使用 LLM 生成高质量摘要
            content = item.content
            
            # 如果内容短，直接使用
            if len(content) <= self.max_length:
                item.summary = content
            else:
                # 截取前 max_length 字符，在句号处截断
                truncated = content[:self.max_length]
                last_period = truncated.rfind('.')
                last_cn_period = truncated.rfind('。')
                cut_pos = max(last_period, last_cn_period)
                
                if cut_pos > self.max_length * 0.5:
                    item.summary = truncated[:cut_pos + 1]
                else:
                    item.summary = truncated + "..."
            
            item.status = ContentStatus.SUMMARIZED
            
            return PipelineResult(
                item=item,
                success=True,
                stage=self.name,
            )
            
        except Exception as e:
            return PipelineResult(
                item=item,
                success=False,
                stage=self.name,
                error=str(e),
            )
```

#### 3.4.3 `processors/translator.py`
```python
"""内容翻译处理器"""

import os
from typing import Optional

from ..models import ContentItem, ContentStatus, PipelineResult
from .base import BaseProcessor


class Translator(BaseProcessor):
    """将英文内容翻译为中文"""
    
    name = "translator"
    
    def __init__(self, use_llm: bool = True):
        self.use_llm = use_llm
    
    async def process(self, item: ContentItem) -> PipelineResult:
        """翻译内容"""
        try:
            content = item.summary or item.content
            
            # 检测是否为英文内容
            if not self._is_english(content):
                # 已经是中文，跳过
                item.translated_content = content
                item.status = ContentStatus.TRANSLATED
                return PipelineResult(item=item, success=True, stage=self.name)
            
            # 翻译
            if self.use_llm:
                translated = await self._translate_with_llm(content)
            else:
                translated = self._simple_translate(content)
            
            item.translated_content = translated
            item.status = ContentStatus.TRANSLATED
            
            return PipelineResult(
                item=item,
                success=True,
                stage=self.name,
            )
            
        except Exception as e:
            return PipelineResult(
                item=item,
                success=False,
                stage=self.name,
                error=str(e),
            )
    
    def _is_english(self, text: str) -> bool:
        """检测文本主要为英文"""
        ascii_chars = sum(1 for c in text if ord(c) < 128)
        return ascii_chars / len(text) > 0.7 if text else False
    
    async def _translate_with_llm(self, text: str) -> str:
        """使用 LLM 翻译"""
        # TODO: 集成真实 LLM 翻译
        # 当前返回模拟翻译结果
        return self._simple_translate(text)
    
    def _simple_translate(self, text: str) -> str:
        """简单翻译 (模拟)"""
        # 实际项目中应调用翻译 API 或 LLM
        # 这里返回带标记的原文表示已"翻译"
        return f"[中文翻译] {text[:200]}..."
```

#### 3.4.4 `processors/xiaohongshu_formatter.py`
```python
"""小红书格式转换器"""

import re
from typing import List

from ..models import ContentItem, ContentStatus, PipelineResult, XiaoHongShuPost
from .base import BaseProcessor


class XiaoHongShuFormatter(BaseProcessor):
    """将内容转换为小红书格式"""
    
    name = "xiaohongshu_formatter"
    
    # 小红书相关热门标签
    DEFAULT_TAGS = [
        "#干货分享",
        "#自我提升",
        "#职场成长",
        "#学习方法",
        "#个人成长",
    ]
    
    def __init__(self, max_length: int = 1000):
        self.max_length = max_length
    
    async def process(self, item: ContentItem) -> PipelineResult:
        """格式化为小红书风格"""
        try:
            content = item.translated_content or item.summary or item.content
            
            # 生成标题
            title = self._generate_title(item)
            
            # 格式化正文
            formatted = self._format_body(content)
            
            # 提取/生成标签
            tags = self._generate_tags(item, content)
            
            # 组装小红书内容
            xiaohongshu_text = self._assemble_post(title, formatted, tags, item.url)
            
            item.xiaohongshu_content = xiaohongshu_text
            item.xiaohongshu_tags = tags
            item.status = ContentStatus.READY
            
            return PipelineResult(
                item=item,
                success=True,
                stage=self.name,
            )
            
        except Exception as e:
            return PipelineResult(
                item=item,
                success=False,
                stage=self.name,
                error=str(e),
            )
    
    def _generate_title(self, item: ContentItem) -> str:
        """生成吸引人的标题"""
        original_title = item.title
        
        # 如果是 tweet，提取核心观点作为标题
        if item.source.value == "x":
            # 使用原文前几字 + 评论风格
            content = item.content
            if len(content) > 20:
                return f"💡 {content[:20]}..."
            return f"💡 {content}"
        
        # RSS 文章标题优化
        if original_title:
            # 添加 emoji 增强吸引力
            if not any(emoji in original_title for emoji in ["💡", "🔥", "📚", "✨"]):
                return f"📚 {original_title}"
        
        return original_title or "💡 每日精选"
    
    def _format_body(self, content: str) -> str:
        """格式化正文"""
        # 限制长度
        if len(content) > self.max_length:
            content = content[:self.max_length] + "..."
        
        # 添加段落间距
        paragraphs = content.split('\n')
        formatted = '\n\n'.join(p.strip() for p in paragraphs if p.strip())
        
        # 添加小红书风格的开头和结尾
        opening = "姐妹们，今天分享一个超有用的观点👇\n\n"
        closing = "\n\n💭 你怎么看？评论区聊聊~"
        
        return opening + formatted + closing
    
    def _generate_tags(self, item: ContentItem, content: str) -> List[str]:
        """生成标签"""
        tags = []
        
        # 根据内容选择相关标签
        content_lower = content.lower()
        
        # 科技/AI 相关内容
        if any(kw in content_lower for kw in ["ai", "人工智能", "技术", "python", "code"]):
            tags.extend(["#AI", "#科技", "#编程"])
        
        # 职场/成长内容
        if any(kw in content_lower for kw in ["work", "startup", "创业", "职场", "效率"]):
            tags.extend(["#职场干货", "#创业", "#效率工具"])
        
        # 学习/知识内容
        if any(kw in content_lower for kw in ["learn", "read", "book", "学习", "读书"]):
            tags.extend(["#学习方法", "#读书", "#知识分享"])
        
        # 添加默认标签
        if len(tags) < 3:
            tags.extend(self.DEFAULT_TAGS[:3-len(tags)])
        
        # 限制标签数量
        return tags[:5]
    
    def _assemble_post(
        self,
        title: str,
        body: str,
        tags: List[str],
        source_url: str = None,
    ) -> str:
        """组装完整帖子"""
        lines = [
            title,
            "",
            body,
            "",
            " ".join(tags),
        ]
        
        if source_url:
            lines.extend(["", f"🔗 原文: {source_url}"])
        
        return "\n".join(lines)
    
    def to_post_object(self, item: ContentItem) -> XiaoHongShuPost:
        """转换为小红书发布对象"""
        return XiaoHongShuPost(
            title=item.title,
            content=item.xiaohongshu_content or "",
            tags=item.xiaohongshu_tags,
            original_url=item.url,
        )
```

---

### 3.5 数据源文件

#### 3.5.1 `sources/base.py`
```python
"""数据源基类"""

from abc import ABC, abstractmethod
from typing import List, Optional
from datetime import datetime

from ..models import ContentItem, ContentSource


class BaseSource(ABC):
    """数据源基类"""
    
    source_type: ContentSource
    
    def __init__(self, config: dict = None):
        self.config = config or {}
    
    @abstractmethod
    async def fetch(
        self,
        since: Optional[datetime] = None,
        limit: int = 10,
    ) -> List[ContentItem]:
        """获取内容
        
        Args:
            since: 从此时间之后的内容
            limit: 最大数量
            
        Returns:
            内容列表
        """
        pass
    
    @abstractmethod
    async def health_check(self) -> bool:
        """检查数据源健康状态"""
        pass
    
    def _generate_id(self, original_id: str) -> str:
        """生成唯一ID"""
        import hashlib
        data = f"{self.source_type.value}_{original_id}"
        return hashlib.md5(data.encode()).hexdigest()[:16]
```

#### 3.5.2 `sources/x_source.py`
```python
"""X (Twitter) 数据源"""

import os
import asyncio
from datetime import datetime
from typing import List, Optional

try:
    import httpx
    HAS_HTTPX = True
except ImportError:
    HAS_HTTPX = False

from .base import BaseSource
from ..models import ContentItem, ContentSource, ContentStatus


class XSource(BaseSource):
    """X (Twitter) 数据源
    
    使用 Twitter API v2 获取推文
    """
    
    source_type = ContentSource.X
    
    def __init__(self, config: dict = None):
        super().__init__(config)
        self.bearer_token = self.config.get("bearer_token") or os.getenv("X_BEARER_TOKEN")
        self.api_base = "https://api.twitter.com/2"
        
    async def fetch(
        self,
        since: Optional[datetime] = None,
        limit: int = 10,
    ) -> List[ContentItem]:
        """获取推文
        
        注意: 这里使用简化实现，实际应调用 Twitter API
        """
        if not HAS_HTTPX:
            print("[XSource] httpx not installed, skipping")
            return []
            
        if not self.bearer_token:
            print("[XSource] No bearer token configured")
            return []
        
        # TODO: 实现真实的 Twitter API 调用
        # 这里返回示例数据用于测试
        return self._get_mock_data(limit)
    
    def _get_mock_data(self, limit: int) -> List[ContentItem]:
        """获取示例数据"""
        samples = [
            {
                "id": "123456",
                "author": "naval",
                "content": "The purpose of life is not to be happy. It is to be useful, to be honorable, to be compassionate, to have it make some difference that you have lived and lived well.",
                "url": "https://twitter.com/naval/status/123456",
            },
            {
                "id": "234567",
                "author": "paulg",
                "content": "Startups die because they lose focus, not because they run out of money. Money is just a symptom of losing focus.",
                "url": "https://twitter.com/paulg/status/234567",
            },
            {
                "id": "345678",
                "author": "sama",
                "content": "AI is the most important technology humanity has ever developed. It will change everything about how we live and work.",
                "url": "https://twitter.com/sama/status/345678",
            },
        ]
        
        items = []
        for i, data in enumerate(samples[:limit]):
            item = ContentItem(
                id=self._generate_id(data["id"]),
                source=ContentSource.X,
                title=f"Tweet by @{data['author']}",
                content=data["content"],
                url=data["url"],
                author=data["author"],
                status=ContentStatus.RAW,
            )
            items.append(item)
        
        return items
    
    async def health_check(self) -> bool:
        """检查 API 状态"""
        if not self.bearer_token:
            return False
        return True


class XMockSource(XSource):
    """X 模拟数据源 (用于测试)"""
    
    async def fetch(
        self,
        since: Optional[datetime] = None,
        limit: int = 10,
    ) -> List[ContentItem]:
        """返回模拟数据"""
        await asyncio.sleep(0.1)  # 模拟网络延迟
        return self._get_mock_data(limit)
```

#### 3.5.3 `sources/rss_source.py`
```python
"""RSS 数据源"""

import asyncio
from datetime import datetime
from typing import List, Optional

try:
    import feedparser
    HAS_FEEDPARSER = True
except ImportError:
    HAS_FEEDPARSER = False

from .base import BaseSource
from ..models import ContentItem, ContentSource, ContentStatus


class RSSSource(BaseSource):
    """RSS 数据源"""
    
    source_type = ContentSource.RSS
    
    def __init__(self, config: dict = None):
        super().__init__(config)
        self.feeds = self.config.get("feeds", [])
        
    async def fetch(
        self,
        since: Optional[datetime] = None,
        limit: int = 10,
    ) -> List[ContentItem]:
        """获取 RSS 内容"""
        if not HAS_FEEDPARSER:
            print("[RSSSource] feedparser not installed, skipping")
            return []
        
        if not self.feeds:
            print("[RSSSource] No RSS feeds configured")
            return []
        
        items = []
        for feed_url in self.feeds[:3]:  # 限制数量
            try:
                feed_items = await self._parse_feed(feed_url, since, limit // len(self.feeds) + 1)
                items.extend(feed_items)
            except Exception as e:
                print(f"[RSSSource] Failed to parse {feed_url}: {e}")
        
        return items[:limit]
    
    async def _parse_feed(
        self,
        feed_url: str,
        since: Optional[datetime],
        limit: int,
    ) -> List[ContentItem]:
        """解析单个 RSS Feed"""
        # 使用线程池执行同步的 feedparser
        import concurrent.futures
        
        with concurrent.futures.ThreadPoolExecutor() as executor:
            future = executor.submit(feedparser.parse, feed_url)
            feed = await asyncio.wrap_future(future)
        
        items = []
        for entry in feed.entries[:limit]:
            # 解析发布时间
            published = None
            if hasattr(entry, 'published_parsed') and entry.published_parsed:
                published = datetime(*entry.published_parsed[:6])
            
            # 过滤时间
            if since and published and published < since:
                continue
            
            # 提取内容
            content = entry.get('summary', '') or entry.get('description', '')
            if hasattr(entry, 'content'):
                content = entry.content[0].value
            
            item = ContentItem(
                id=self._generate_id(entry.get('id', entry.link)),
                source=ContentSource.RSS,
                title=entry.get('title', 'Untitled'),
                content=content[:2000],  # 限制长度
                url=entry.get('link'),
                author=feed.feed.get('title'),
                published_at=published,
                status=ContentStatus.RAW,
                metadata={
                    'feed_url': feed_url,
                    'feed_title': feed.feed.get('title'),
                }
            )
            items.append(item)
        
        return items
    
    async def health_check(self) -> bool:
        """检查 RSS 源状态"""
        if not HAS_FEEDPARSER:
            return False
        if not self.feeds:
            return False
        return True


class RSSMockSource(RSSSource):
    """RSS 模拟数据源"""
    
    def __init__(self, config: dict = None):
        super().__init__(config)
        # 预配置一些示例 feeds
        self.feeds = ["https://example.com/feed.xml"]
    
    async def fetch(
        self,
        since: Optional[datetime] = None,
        limit: int = 10,
    ) -> List[ContentItem]:
        """返回模拟数据"""
        await asyncio.sleep(0.1)
        
        samples = [
            {
                "id": "rss_001",
                "title": "2024年AI发展报告：大模型技术突破",
                "content": "今年大语言模型在理解和生成能力上取得了显著进展...",
                "url": "https://example.com/ai-report-2024",
                "author": "科技日报",
            },
            {
                "id": "rss_002",
                "title": "Python 3.13 新特性解析",
                "content": "Python 3.13 带来了性能提升和新语法特性...",
                "url": "https://example.com/python-313",
                "author": "Python Weekly",
            },
            {
                "id": "rss_003",
                "title": "如何建立个人知识管理系统",
                "content": "在信息爆炸的时代，建立有效的知识管理系统至关重要...",
                "url": "https://example.com/knowledge-management",
                "author": "效率工具",
            },
        ]
        
        items = []
        for data in samples[:limit]:
            item = ContentItem(
                id=self._generate_id(data["id"]),
                source=ContentSource.RSS,
                title=data["title"],
                content=data["content"],
                url=data["url"],
                author=data["author"],
                status=ContentStatus.RAW,
            )
            items.append(item)
        
        return items
```

---

## 4. 数据模型总结

### 4.1 ContentItem 完整字段说明

| 字段 | 类型 | 必需 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | str | ✓ | - | 内容唯一标识符 (MD5 哈希) |
| `source` | ContentSource | ✓ | - | 内容来源 (X, RSS, MANUAL) |
| `title` | str | ✓ | - | 内容标题 |
| `content` | str | ✓ | - | 原始内容正文 |
| `url` | Optional[str] | ✗ | None | 原始内容链接 |
| `author` | Optional[str] | ✗ | None | 作者名称 |
| `published_at` | Optional[datetime] | ✗ | None | 发布时间 |
| `status` | ContentStatus | ✗ | RAW | 处理状态 |
| `summary` | Optional[str] | ✗ | None | 摘要文本 (由 Summarizer 生成) |
| `translated_content` | Optional[str] | ✗ | None | 翻译后内容 (由 Translator 生成) |
| `xiaohongshu_content` | Optional[str] | ✗ | None | 小红书格式内容 (由 XiaoHongShuFormatter 生成) |
| `xiaohongshu_tags` | List[str] | ✗ | [] | 小红书标签 |
| `metadata` | Dict[str, Any] | ✗ | {} | 额外元数据 |
| `created_at` | datetime | ✗ | datetime.now() | 创建时间 |
| `updated_at` | datetime | ✗ | datetime.now() | 更新时间 |

### 4.2 数据流转过程

```
Raw ContentItem (从源采集)
    ↓
Summarizer 处理 → ContentItem.summary, status=SUMMARIZED
    ↓
Translator 处理 → ContentItem.translated_content, status=TRANSLATED
    ↓
XiaoHongShuFormatter 处理 → ContentItem.xiaohongshu_content, xiaohongshu_tags, status=READY
    ↓
XiaoHongShuPublisher 发布 → ContentItem.status=PUBLISHED
```

---

这个文档完整地包含了您请求的所有文件的完整代码及详细说明。
