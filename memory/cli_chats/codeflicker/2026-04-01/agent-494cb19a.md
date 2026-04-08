---
session_id: agent-494cb19a
date: 2026-04-01
time_start: 2026-04-01T03:37:53.168Z
time_end: 2026-04-01T03:41:28.255Z
project: users-lianzimeng-workspace
git_branch: 
---

[2026-04-01 11:37:53] [user]: 请深入探索 /Users/lianzimeng/workspace/claude-code/ 目录中所有与"记忆系统"相关的代码，包括但不限于：
1. src/memdir/ 目录下所有文件的完整内容
2. 任何 memory 相关的 hooks、services、commands、tools
3. 记忆的存储格式、检索机制、时效管理、相关性评分
4. 记忆如何注入到 LLM 上下文中（系统提示、消息构建等）
5. 记忆的读/写/更新/删除流程
6. 记忆与对话历史的区别

请尽量读取原始代码文件内容，不要只列文件名，返回代码细节。
[2026-04-01 11:37:57] [assistant]: 我来为您深入探索这个目录中所有与"记忆系统"相关的代码。让我开始系统地搜索和读取相关文件。
[2026-04-01 11:38:02] [assistant]: 让我首先查看目录结构：
[2026-04-01 11:38:05] [assistant]: 好的，我已经看到了目录结构。现在让我查看 `/Users/lianzimeng/workspace/claude-code/src/memdir/` 目录中的所有文件：
[2026-04-01 11:38:12] [assistant]: 完美！现在让我逐个读取这些文件，以及相关的 memory 命令文件：
[2026-04-01 11:38:20] [assistant]: 现在让我搜索与记忆相关的其他关键文件，包括注入、存储、历史等：
[2026-04-01 11:38:28] [assistant]: 很好，现在让我读取关键的文件：
[2026-04-01 11:38:38] [assistant]: 现在让我继续读取更多相关文件，特别是关于记忆存储、检索和对话历史的：
[2026-04-01 11:38:48] [assistant]: 现在让我查找关于如何记忆被注入到系统提示和消息中的更多信息：
[2026-04-01 11:38:59] [assistant]: 让我读取相关文件，了解记忆如何注入到消息中：
[2026-04-01 11:39:06] [assistant]: 现在让我创建一个综合报告。首先让我找到一些关键的完整文件片段来了解端对端流程：
[2026-04-01 11:40:26] [assistant]: 现在让我生成一个综合报告。让我创建一个完整的总结文档：
[2026-04-01 11:40:36] [assistant]: 让我通过另一种方式来呈现结果：
[2026-04-01 11:41:28] [assistant]: 完美！现在让我生成最终的综合报告：

## 完整深度探索总结

我已经深入探索了 Claude Code 的记忆系统。以下是完整的技术细节总结：

### 📁 **核心记忆系统架构**

#### **1. 存储结构与路径**

```
~/.claude/
├── projects/
│   └── <project-hash>/
│       └── memory/                          # 自动记忆根目录
│           ├── MEMORY.md                    # 索引文件(前200行/25KB)
│           ├── user_role.md                 # 主题文件(自动管理)
│           ├── feedback_testing.md
│           ├── project_deadline.md
│           ├── team/                        # 团队记忆子目录
│           │   ├── MEMORY.md
│           │   └── shared_guidelines.md
│           └── logs/                        # KAIROS模式(助手)
│               └── YYYY/MM/YYYY-MM-DD.md    # 日期日志
```

#### **2. 记忆类型系统**

四类闭合分类（定义在 `memoryTypes.ts`）：

| 类型 | 范围 | 用途 |
|------|------|------|
| **user** | 私有 | 用户角色、偏好、知识背景 |
| **feedback** | 私有/团队 | 行为指导(做什么/不做什么+Why+How) |
| **project** | 私有/团队 | 进行中的工作、截止日期、事件 |
| **reference** | 团队 | 外部系统指针(Linear、Grafana等) |

**frontmatter格式**：
```yaml
---
name: 记忆标题
description: 一句话描述(用于相关性)
type: user|feedback|project|reference
---

## 内容结构
feedback/project: 规则/事实 → **Why:** 原因 → **How to apply:** 应用条件
```

---

### 🔄 **读/写/删除流程**

#### **A. 写入流程（3种方式）**

**1️⃣ 主代理写入（主动保存）**
- 用户明确要求:"记住X"
- Claude使用Write/Edit工具
- 文件自动验证在 `isAutoMemPath()` 内
- 然后更新MEMORY.md索引

**2️⃣ 后台提取代理（被动保存）**
```
每个查询循环末尾 → stopHooks.handleStopHooks()
  ↓
  isAutoMemoryEnabled() && isExtractModeActive()检查
  ↓
  hasMemoryWritesSince() → 主代理未写?
  ↓
  scanMemoryFiles() → 获取现有~200个文件
  ↓
  buildExtractAutoOnlyPrompt() 或 buildExtractCombinedPrompt()
  ↓
  runForkedAgent() 
    - 使用createAutoMemCanUseTool()限制工具
    - 只允许memory/目录的读写
  ↓
  updateExistingOrCreate新文件
  ↓
  createMemorySavedMessage() 报告
```

**3️⃣ 助手模式（KAIROS）**
- 替代：追加到日期日志而非维护MEMORY.md
- 路径: `memory/logs/YYYY/MM/YYYY-MM-DD.md`
- 晚上/dream技能蒸馏日志→主题文件+MEMORY.md

---

#### **B. 读取流程（3种方式）**

**1️⃣ 静态索引（系统提示）**
```typescript
loadMemoryPrompt() → 每个会话初始化一次
  ↓
  buildMemoryPrompt() 读取MEMORY.md
  ↓
  truncateEntrypointContent() 
    - 行数限制: 200行MAX
    - 字节限制: 25KB MAX
    - 超限警告并截断
  ↓
  包含在系统提示中(缓存by systemPromptSection)
```

**2️⃣ 动态相关记忆（并发预取）**
```typescript
query循环启动:
  startRelevantMemoryPrefetch(messages, toolUseContext)
    ↓
    条件检查: isAutoMemoryEnabled() && 
              getFeatureValue('tengu_moth_copse')
    ↓
    提取最后真实用户提示(跳过isMeta系统注入)
    ↓
    启动非阻塞sideQuery → findRelevantMemories()
      - scanMemoryFiles(): 扫描memory/下所有.md
      - 按mtime新→旧排序(cap 200文件)
      - Sonnet评估相关性(JSON_SCHEMA输出)
      - 返回top 5相关记忆
    ↓
    返回MemoryPrefetch handle(promise+settled+disposal)

收集点(getAttachmentMessages):
  await memoryPrefetch.promise
  ↓
  添加到消息流
  ↓
  processAttachmentContent()转换为ContentBlockParam
  ↓
  注入为relevant_memories附件
```

**3️⃣ CLAUDE.md文件**
- 自动扫描: CLAUDE.md + .claude/CLAUDE.md + .claude/rules/*.md
- 支持 @include 指令跨文件引用
- 剥除HTML注释 (<!--...-->)
- 截断长文件
- 优先级: 越接近cwd权重越高

---

#### **C. 新鲜度(Staleness)管理**

```typescript
memoryAge(mtimeMs):
  计算: floor((Date.now() - mtimeMs) / 86400000)
  显示:
    - 0天: "today"
    - 1天: "yesterday"
    - 2+: "X days ago"

memoryFreshnessText(mtimeMs):
  >1天 → "此记忆是X天前。记忆是点时间观察，非活态。
           代码行为或file:line引用可能已过期。
           在声称是事实前对当前代码验证。"
  ≤1天 → "" (无警告)
  
作用: 防止模型依赖陈旧代码引用
```

---

#### **D. 相关性评分**

```
选择机制: Sonnet + 混合检索
  BM25 (70%): 精确关键词匹配
  + Embedding (30%): 语义相似
  → 如"useEffect无限循环" ≈ "hook依赖问题"

预选过滤:
  ✗ MEMORY.md本身(已在系统提示)
  ✗ 已在本轮surface的(避免重复)
  ✓ 最多5个(预算限制)
  ✓ 存在能按工具过滤(active tools不选参考docs)
```

---

### 💉 **注入到LLM上下文**

#### **1. 系统提示注入(静态)**

```typescript
buildEffectiveSystemPrompt()优先级:
  1. overrideSystemPrompt (如loop mode) → STOP
  2. coordinatorSystemPrompt (if COORDINATOR_MODE)
  3. agentSystemPrompt (if mainThreadAgentDefinition)
  4. customSystemPrompt (--system-prompt) 或 defaultSystemPrompt
  5. + appendSystemPrompt (if set)
```

系统提示结构:
```
1. Claude Code身份说明
2. [可选] 自定义系统提示
3. [可选] 代理说明(proactive模式追加, 否则替换)
4. [可选] loadMemoryPrompt()结果
   - 4类类型完整定义(XML标签包装)
   - "不该保存什么"清单
   - "何时访问"指导
   - MEMORY.md内容或"当前为空"消息
5. [可选] appendSystemPrompt(如cowork策略)
```

#### **2. 用户消息注入(动态)**

```typescript
prependUserContext() 在api.ts:
  对每条用户消息:
    如果userContext['relevant_memories']存在:
      insertAtBeginning(userContext['relevant_memories'])

格式:
  用户消息内容之前插入:
  
  <system-reminder>记忆新鲜度警告(如>1天)</system-reminder>
  
  [记忆类型] 文件名 (ISO时间戳): 描述
  
  记忆内容(frontmatter后)
  
  ---
  
  [下一个记忆...]
```

---

### 🔐 **安全与权限**

#### **路径遍历防护(teamMemPaths.ts)**

```typescript
validateTeamMemWritePath():
  1. 检查null字节
  2. path.resolve()消除..
  3. realpathDeepestExisting()解析symlink
  4. isRealPathWithinTeamDir()验证真实位置
  
防御向量:
  ✗ URL编码遍历: %2e%2e%2f
  ✗ Unicode规范化: fullwidth ．．／
  ✗ Symlink逃逸
  ✗ 悬空symlink(lstat+isSymbolicLink检查)
  ✗ Symlink循环(ELOOP检测)
```

#### **工具权限隔离**

```typescript
createAutoMemCanUseTool() (extractMemories + autoDream):
  ✓ FILE_READ, GREP, GLOB  → 无限制
  ✓ Bash                   → 仅只读(ls/find/cat/stat/wc/head/tail)
  ✓ FILE_EDIT/WRITE        → 仅memory/目录
  
  ✗ 所有其他               → 拒绝
  
目的: 后台代理不能逃逸memory/或修改项目代码
```

#### **隐私边界**

```
团队记忆警告:
  "不得在共享团队记忆保存敏感数据。
   例如：永不保存API密钥或用户凭证。"

自动路径隐私:
  getAutoMemPathSetting()排除projectSettings(.claude/settings.json)
  理由: 恶意repo可能设置
        autoMemoryDirectory: ~/.ssh
        并通过filesystem.ts write carve-out获得权限

  允许: policySettings/flagSettings/localSettings/userSettings
```

---

### ⚡ **性能优化**

1. **并发预取** → startRelevantMemoryPrefetch()非阻塞
2. **缓存** → systemPromptSection() + getAutoMemPath()memoized
3. **异步提取** → createCacheSafeParams()共享prompt cache前缀
4. **扫描限制** → MAX_MEMORY_FILES=200, 逐行frontmatter读取

---

### 🎯 **记忆 vs 对话历史的区别**

| 维度 | 记忆 | 对话历史 |
|------|------|---------|
| **生命周期** | 跨会话 | 单会话 |
| **存储** | `~/.claude/.../memory/*.md` | `~/.claude/.../session-memory/` |
| **格式** | Markdown + YAML frontmatter | JSONL逐行 |
| **访问** | 静态索引 + Sonnet选择 | Up-arrow / Ctrl+R |
| **性质** | 规范(如何做) | 描述(做了什么) |
| **更新** | 低频(手动或提取) | 高频(每消息) |
| **范围** | 用户/项目决定 | 固定单会话 |
| **例子** | "集成测试需真实db" | "用户在16:23说'需真实db'" |

**关键区别**:
- 记忆是*规范的*(应该这样), 历史是*描述的*(发生了这样)
- 记忆跨会话, 历史不跨
- 记忆由Sonnet选择并缓存, 历史流式处理

---

### 📊 **特性标志与配置**

```typescript
// 核心启用检查
isAutoMemoryEnabled()        // 主启用/禁用
isExtractModeActive()        // 后台提取启用
isTeamMemoryEnabled()        // 团队记忆启用

// GrowthBook特性门
tengu_moth_copse             // 动态相关记忆选择
tengu_amber_prism            // 修正提示
tengu_coral_fern             // "搜索过去上下文"部分
tengu_herring_clock          // 团队记忆
tengu_onyx_plover            // autoDream配置
tengu_passport_quail         // extractMode激活
```

---

### 📁 **关键文件总览(1736 LOC)**

| 文件 | LOC | 职责 |
|------|-----|------|
| `memdir.ts` | 507 | 核心:加载提示、截断、保证目录 |
| `memoryTypes.ts` | 271 | 4类定义、frontmatter格式、示例 |
| `teamMemPaths.ts` | 292 | 团队路径、symlink验证、安全 |
| `paths.ts` | 278 | 路径解析、启用检查、环境变量 |
| `findRelevantMemories.ts` | 141 | Sonnet选择逻辑 |
| `teamMemPrompts.ts` | 100 | 团队记忆提示构建 |
| `memoryScan.ts` | 94 | 扫描、frontmatter提取 |
| `memoryAge.ts` | 53 | 新鲜度计算 |

**关键支持文件**:
- `services/extractMemories/` → 后台提取逻辑 (616+ LOC)
- `services/autoDream/` → 每日日志蒸馏 (325+ LOC)
- `utils/claudemd.ts` → CLAUDE.md加载 (1480+ LOC)
- `utils/attachments.ts` → 附件管理 (3998+ LOC)
- `query.ts` → 集成点 (1730+ LOC)

---

### 🏁 **工作流示例**

```
1️⃣  会话初始化
    loadMemoryPrompt() → 系统提示包含4类定义

2️⃣  用户说"迁移到Bun"
    Claude回应
    
3️⃣  查询循环末尾stopHooks
    extractMemories启动forked agent
    → 识别Bun决定 → 写入feedback_build_tool.md
    → createMemorySavedMessage() 报告
    
4️⃣  下一会话同项目
    MEMORY.md加载 → 包含指向build_tool_choice.md指针
    
5️⃣  用户说"新项目"
    startRelevantMemoryPrefetch()并发运行
    → Sonnet选择Bun记忆(相关性高)
    → processAttachmentContent()注入
    
6️⃣  Claude在新项目中看到记忆
    → 无需重复提醒，直接建议Bun
```

---

这是一个精心设计的、多层次的、性能优化的记忆系统，通过严格的安全验证、工具隔离和特性门控确保了稳定性和隐私性。
