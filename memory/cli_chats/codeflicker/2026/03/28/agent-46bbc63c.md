---
session_id: agent-46bbc63c
date: 2026-03-28
time_start: 2026-03-28T17:12:30.473Z
time_end: 2026-03-28T17:16:06.446Z
project: users-lianzimeng-workspace-huaqi-growing
git_branch: main
---

[2026-03-29 01:12:30] [user]: 在 /Users/lianzimeng/workspace/huaqi-growing 项目中，找到用户画像(profile/用户画像)相关的所有代码。重点关注：
1. profile 数据结构定义
2. profile 如何存储和读取
3. profile show 命令的实现
4. 是否有 LLM 动态生成 profile 的逻辑

请返回相关文件路径和关键代码片段。
[2026-03-29 01:12:41] [assistant]: 我会帮你在项目中找到用户画像(profile)相关的所有代码。让我开始搜索。
[2026-03-29 01:12:48] [assistant]: 好的，找到了关键文件。现在让我继续搜索profile相关的命令实现和LLM逻辑。
[2026-03-29 01:12:52] [assistant]: 现在让我读取这些关键文件的内容：
[2026-03-29 01:12:56] [assistant]: 现在让我搜索 profile show 命令和其他相关实现：
[2026-03-29 01:13:04] [assistant]: 现在让我获取完整的 profile_show 函数和其他相关命令：
[2026-03-29 01:13:09] [assistant]: 现在让我再搜索一下 LLM 动态生成 profile 的相关逻辑：
[2026-03-29 01:13:15] [assistant]: 现在让我查找在 chat_mode 中关于 profile 的使用：
[2026-03-29 01:13:18] [assistant]: 完美！现在让我看一下 chat_nodes.py 中关于 profile 的完整实现：
[2026-03-29 01:13:24] [assistant]: 现在让我查看完整的提取流程：
[2026-03-29 01:13:35] [assistant]: 完美！现在我有了完整的信息。让我整理成一份综合报告：
[2026-03-29 01:16:06] [assistant]: 现在，让我为用户整理所有发现的代码路径和关键片段：

## 📋 用户画像(Profile)代码全景分析

我已完整扫描项目，找到了用户画像相关的所有代码。以下是详细的找到结果：

---

### 1️⃣ **Profile 数据结构定义** 
📁 **文件**: `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/core/user_profile.py`

**核心类**：
- `UserIdentity` - 身份信息（名字、昵称、职业、公司、所在地、生日）
- `UserPreferences` - 用户偏好（沟通风格、响应长度、兴趣话题）
- `UserBackground` - 背景信息（教育、技能、爱好、家庭、目标、价值观）
- `UserProfile` - 完整画像（整合上述三类信息 + 元数据和提取历史）

```python
@dataclass
class UserProfile:
    """完整用户画像"""
    identity: UserIdentity = field(default_factory=UserIdentity)
    preferences: UserPreferences = field(default_factory=UserPreferences)
    background: UserBackground = field(default_factory=UserBackground)
    created_at: str = field(default_factory=lambda: datetime.now().isoformat())
    updated_at: str = field(default_factory=lambda: datetime.now().isoformat())
    version: int = 1
    extraction_history: List[Dict[str, Any]] = field(default_factory=list)
```

---

### 2️⃣ **Profile 存储和读取**
📁 **文件**: `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/core/user_profile.py`

**存储位置**: `~/.huaqi/memory/user_profile.yaml`

**管理器类**: `UserProfileManager`

```python
class UserProfileManager:
    """用户画像管理器"""
    PROFILE_FILENAME = "user_profile.yaml"
    
    def __init__(self, memory_dir: Optional[Path] = None):
        self.memory_dir = memory_dir or get_memory_dir()
        self.profile_path = self.memory_dir / self.PROFILE_FILENAME
        self._profile: Optional[UserProfile] = None
    
    def _load_or_create(self) -> UserProfile:
        """加载或创建用户画像"""
        if self.profile_path.exists():
            with open(self.profile_path, "r", encoding="utf-8") as f:
                data = yaml.safe_load(f) or {}
                return UserProfile.from_dict(data)
        return UserProfile()
    
    @property
    def profile(self) -> UserProfile:
        """获取用户画像（懒加载）"""
        if self._profile is None:
            self._profile = self._load_or_create()
        return self._profile
    
    def save(self):
        """保存用户画像"""
        self.profile_path.parent.mkdir(parents=True, exist_ok=True)
        with open(self.profile_path, "w", encoding="utf-8") as f:
            yaml.dump(self.profile.to_dict(), f, allow_unicode=True, sort_keys=False)
```

---

### 3️⃣ **Profile Show 命令实现**
📁 **文件**: `/Users/lianzimeng/workspace/huaqi-growing/cli.py` (行号 1510-1556)

```python
@profile_app.command("show")
def profile_show():
    """显示用户画像"""
    ensure_initialized()
    
    from huaqi_src.core.user_profile import get_profile_manager
    
    profile_manager = get_profile_manager()
    profile = profile_manager.profile
    
    console.print("\n[bold magenta]👤 用户画像[/bold magenta]\n")
    
    # 身份信息
    identity_table = Table(box=box.ROUNDED, title="身份信息")
    identity_table.add_column("项目", style="cyan")
    identity_table.add_column("值")
    
    identity = profile.identity
    identity_table.add_row("名字", identity.name or "未设置")
    identity_table.add_row("昵称", identity.nickname or "未设置")
    identity_table.add_row("职业", identity.occupation or "未设置")
    identity_table.add_row("公司", identity.company or "未设置")
    identity_table.add_row("所在地", identity.location or "未设置")
    identity_table.add_row("生日", identity.birth_date or "未设置")
    
    console.print(identity_table)
    console.print()
    
    # 背景信息
    background_table = Table(box=box.ROUNDED, title="背景信息")
    background_table.add_column("项目", style="cyan")
    background_table.add_column("内容")
    
    background = profile.background
    background_table.add_row("教育", background.education or "未设置")
    background_table.add_row("技能", ", ".join(background.skills) if background.skills else "未设置")
    background_table.add_row("爱好", ", ".join(background.hobbies) if background.hobbies else "未设置")
    background_table.add_row("目标", ", ".join(background.life_goals) if background.life_goals else "未设置")
    
    console.print(background_table)
```

**其他 Profile 相关命令**：
- `profile set <field> <value>` - 设置字段 (行号 1557-1592)
- `profile forget <field>` - 删除字段 (行号 1594-1627)

---

### 4️⃣ **LLM 动态生成 Profile - 启动时提取**
📁 **文件**: `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/core/user_profile.py` (核心类: `UserDataExtractor`)

#### **4.1 启动流程**
📁 **文件**: `/Users/lianzimeng/workspace/huaqi-growing/cli.py` (行号 1155-1173)

```python
# 启动时异步提取用户信息
try:
    from huaqi_src.core.user_profile import get_data_extractor
    from huaqi_src.core.llm import LLMConfig, LLMManager
    
    extractor = get_data_extractor()
    
    if not extractor.is_extracting() and extractor.get_result() is None:
        # 初始化 LLM
        _llm_for_extraction = LLMManager()
        # ... 配置 LLM ...
        extractor.start_extraction(_llm_for_extraction)
        console.print("[dim]💡 正在分析你的日记和对话...[/dim]\n")
except Exception:
    pass  # 提取失败不影响主流程
```

#### **4.2 UserDataExtractor 核心实现**

```python
class UserDataExtractor:
    """启动时从多数据源提取用户信息
    
    异步执行，不阻塞用户输入。
    失败时自动重试，直到成功。
    """
    
    MAX_RETRIES = 3           # 最大重试次数
    RETRY_DELAY_BASE = 2.0    # 基础延迟（秒），指数退避
    LLM_TIMEOUT = 30          # 每次 LLM 调用超时（秒）
    
    def start_extraction(
        self,
        llm_manager,
        on_complete: Optional[Callable[[Dict[str, Any]], None]] = None,
        on_status: Optional[Callable[[str], None]] = None
    ):
        """启动异步提取（后台线程，不阻塞）"""
        if self._is_extracting:
            return  # 已经在提取中
            
        self._is_extracting = True
        self._on_status_update = on_status
        
        def _extract_worker():
            try:
                result = self._extract_with_llm(llm_manager)
                self._extraction_result = result
                if on_complete:
                    on_complete(result)
            except Exception:
                self._extraction_result = {}
                if on_complete:
                    on_complete({})
            finally:
                self._is_extracting = False
        
        self._extraction_thread = threading.Thread(target=_extract_worker, daemon=True)
        self._extraction_thread.start()
```

#### **4.3 三层保障策略**

**第1层：LLM 智能提取**
```python
def _extract_with_llm(self, llm_manager) -> Dict[str, Any]:
    """使用 LLM 提取用户信息（带重试机制）"""
    # 采集数据：用户画像 + 日记 + 对话历史
    user_data = self._collect_user_data()
    
    # 构建提示词，分析日记和对话
    prompt = self._build_extraction_prompt(user_data)
    
    # 重试机制：最多3次，指数退避
    while self._retry_count < self.MAX_RETRIES:
        try:
            response = llm_manager.chat(messages)
            extracted = json.loads(response.content)
            # 应用提取结果
            self._apply_extraction(extracted, "startup_analysis")
            return extracted
        except Exception as e:
            self._retry_count += 1
            if self._retry_count < self.MAX_RETRIES:
                delay = self.RETRY_DELAY_BASE * (2 ** (self._retry_count - 1))
                # 延迟后重试...
```

**第2层：规则兜底提取**
```python
def _extract_with_fallback(self) -> Dict[str, Any]:
    """兜底提取方案：基于规则的关键词匹配"""
    extracted = {}
    
    patterns = [
        (r'我是([\\u4e00-\\u9fa5]{2,4})', 'name'),
        (r'我叫([\\u4e00-\\u9fa5]{2,4})', 'name'),
        (r'昵称[是为]?([\\u4e00-\\u9fa5\\w]{1,6})', 'nickname'),
        (r'([\\u4e00-\\u9fa5]{2,6})工程师', 'occupation'),
        (r'住在([^，。]{2,10})', 'location'),
        (r'我会([\\u4e00-\\u9fa5\\w\\s,]+)', 'skills'),
        (r'喜欢([\\u4e00-\\u9fa5\\w\\s,]+)', 'hobbies'),
        # ... 更多模式 ...
    ]
    
    # 逐项匹配并提取
    for pattern, field in patterns:
        matches = re.findall(pattern, content)
        if matches:
            value = matches[0].strip()
            extracted[field] = value
    
    return extracted
```

**第3层：空结果兜底**
```python
# 所有重试都失败时，返回空字典，不影响使用
return {}
```

---

### 5️⃣ **在对话中的增量提取**
📁 **文件**: `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/agent/nodes/chat_nodes.py`

#### **5.1 对话节点中的信息提取**
```python
def extract_user_info(state: AgentState) -> Dict[str, Any]:
    """从用户消息中提取用户信息节点
    
    检测用户自我介绍并更新画像
    """
    messages = state.get("messages", [])
    
    # 获取最后一条用户消息
    last_message = ""
    for msg in reversed(messages):
        if isinstance(msg, HumanMessage):
            last_message = msg.content
            break
    
    if not last_message:
        return {}
    
    try:
        profile_manager = get_profile_manager()
        extracted = profile_manager.extract_from_message(last_message)
        
        if extracted:
            return {"user_info_extracted": extracted}
    except Exception:
        pass
    
    return {}
```

#### **5.2 Profile 注入系统提示词**
```python
def build_context(state: AgentState) -> Dict[str, Any]:
    """构建上下文节点 - 组装系统提示词、人格画像、记忆等"""
    
    # 获取用户画像上下文
    user_profile_context = ""
    try:
        profile_manager = get_profile_manager()
        user_profile_context = profile_manager.get_system_prompt_addition()
    except Exception:
        pass
    
    # 构建系统提示词
    system_prompt = build_system_prompt(
        personality_context,
        user_profile_context,  # 注入用户信息
        user_insight_context,
    )
    
    return {"workflow_data": {"system_prompt": system_prompt}}

def build_system_prompt(
    personality_context: Optional[str] = None,
    user_profile_context: Optional[str] = None,
    user_insight_context: Optional[str] = None,
) -> str:
    """构建系统提示词"""
    base_prompt = """你是 Huaqi (花旗)，一个个人 AI 伴侣系统..."""
    
    if personality_context:
        base_prompt += f"\n\n{personality_context}\n"
    
    # 添加用户画像信息
    if user_profile_context:
        base_prompt += f"\n{user_profile_context}\n"
    
    # 添加用户理解洞察
    if user_insight_context:
        base_prompt += f"\n{user_insight_context}\n"
    
    return base_prompt
```

#### **5.3 生成用于 Prompt 的摘要**
```python
def get_system_prompt_addition(self) -> str:
    """获取用于系统提示词的画像信息"""
    summary = self.profile.get_summary()
    if not summary:
        return ""
    return f"\n\n### 用户信息\n{summary}"

def get_summary(self) -> str:
    """生成画像摘要（用于 Prompt）"""
    parts = []
    
    # 身份信息
    if self.identity.name:
        parts.append(f"名字是{self.identity.name}")
    if self.identity.occupation:
        parts.append(f"职业是{self.identity.occupation}")
    if self.identity.location:
        parts.append(f"住在{self.identity.location}")
    
    # 背景信息
    if self.background.skills:
        parts.append(f"技能：{', '.join(self.background.skills)}")
    if self.background.hobbies:
        parts.append(f"爱好：{', '.join(self.background.hobbies)}")
    
    return "用户" + "，".join(parts) + "。" if parts else ""
```

---

### 6️⃣ **人格画像更新机制**
📁 **文件**: `/Users/lianzimeng/workspace/huaqi-growing/huaqi_src/core/personality_updater.py`

通过分析日记和对话，自动识别用户画像变化：

```python
class PersonalityUpdater:
    """人格画像更新管理器"""
    
    def analyze_recent(
        self,
        days: int = 7,
        min_confidence: float = 0.6,
    ) -> Optional[ProfileUpdateProposal]:
        """分析最近的日记，生成更新提案"""
        # 获取最近的日记
        recent_entries = self.diary_store.list_entries(...)
        
        # 分析变化
        changes = self.analyzer.analyze_entries(entries_data, current_profile)
        
        # 创建提案（用户确认后才应用）
        proposal = ProfileUpdateProposal(...)
        self._save_proposal(proposal)
        return proposal
```

---

### 📊 **文件清单和对应关系**

| 功能 | 文件路径 | 关键类/函数 |
|------|---------|-----------|
| **数据结构定义** | `user_profile.py` | `UserProfile`, `UserIdentity`, `UserPreferences`, `UserBackground` |
| **存储和读取** | `user_profile.py` | `UserProfileManager` |
| **CLI 命令** | `cli.py` (L1500+) | `profile_show`, `profile_set`, `profile_forget` |
| **启动时 LLM 提取** | `user_profile.py` | `UserDataExtractor` |
| **对话中信息提取** | `chat_nodes.py` | `extract_user_info`, `build_context` |
| **人格更新建议** | `personality_updater.py` | `PersonalityUpdater`, `ProfileAnalyzer` |
| **文档** | `docs/USER_PROFILE_EXTRACTION.md` | 完整流程设计文档 |

---

### 🔄 **完整数据流**

```
启动 CLI
  │
  ├─→ UserDataExtractor.start_extraction()  (后台异步)
  │    ├─→ 收集数据（画像+日记+对话）
  │    ├─→ LLM 智能提取（3次重试）
  │    │    └─→ 失败 → 规则兜底提取
  │    └─→ 应用到 user_profile.yaml
  │
  ├─→ chat_mode() 启动对话循环
  │    ├─→ 用户输入
  │    ├─→ extract_user_info() 增量提取
  │    ├─→ build_context() 注入 profile 到系统提示词
  │    ├─→ LLM 生成回复
  │    └─→ 返回结果
  │
  └─→ 定期检查 (PersonalityUpdater)
       └─→ 生成画像更新提案（需用户确认）
```

所有代码已完整找到！🎉
