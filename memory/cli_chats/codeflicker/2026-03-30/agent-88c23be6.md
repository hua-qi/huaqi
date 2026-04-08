---
session_id: agent-88c23be6
date: 2026-03-30
time_start: 2026-03-30T06:22:48.956Z
time_end: 2026-03-30T06:23:35.389Z
project: users-lianzimeng-workspace-xp
git_branch: main
---

[2026-03-30 14:22:48] [user]: Look into src/cli.py, src/knowledge.py, src/storage.py, and src/models.py for how "xp stats" and "xp analyze" compute their metrics. Focus on how error rate (报错率), adoption rate (采纳率), and auto infer matching logic work, since the user's screenshot shows an anomaly where error rate is higher with injection (50%) than without (15.8%), adoption rate is 0%, and auto infer keyword matching is very low. Return a concise summary of the code logic that leads to these calculations, so I can explain it to the user.
[2026-03-30 14:23:13] [assistant]: Let me also check the `_extract_keywords` function more closely and look for any other relevant logic:
[2026-03-30 14:23:35] [assistant]: ## Summary of `xp stats` and `xp analyze` Metrics Logic

Based on my analysis of the codebase, here's a concise explanation of how these metrics are computed and what might cause the anomalies in your screenshot:

---

### 1. Error Rate (报错率) Calculation

**Location:** `storage.py` lines 569-577 in `get_stats()`

```python
sessions_with = conn.execute(
    f"SELECT AVG(had_error_correction) as avg_err ... WHERE json_array_length(experience_ids_injected) > 0"
)
sessions_without = conn.execute(
    f"SELECT AVG(had_error_correction) as avg_err ... WHERE json_array_length(experience_ids_injected) = 0"
)
```

**How it works:**
- **Error rate = `AVG(had_error_correction)`** — the average of a boolean (0/1) value across sessions
- **The key grouping logic:** Uses `json_array_length(experience_ids_injected) > 0` to determine "has injection" vs "no injection"
- **The anomaly explanation:** If sessions with injection have higher error rates, it could be:
  1. **Selection bias**: Users only inject experience for harder tasks that naturally have more errors
  2. **Injection detection issue**: The `experience_ids_injected` array must be non-empty for injection to count

---

### 2. Adoption Rate (采纳率) Calculation

**There are TWO different adoption rates:**

#### A) Session-Level Adoption Rate (shown in `xp stats`)
- **Location:** `storage.py` lines 569-577
- **Formula:** `AVG(user_accepted)` — boolean average of whether user accepted the result
- **NOT the feedback adoption rate!** This is whether the user accepted the final output, not whether the injected experience was adopted

#### B) Feedback-Level Adoption Rate (shown in `xp analyze`)
- **Location:** `storage.py` lines 532-539 in `get_feedback_summary()`
- **Formula:** `adopted_count / total_feedback`
- **Requires explicit feedback calls** via `record_feedback` or `infer_adoption`

**Why adoption_rate shows 0% in your screenshot:**
- The `xp stats` command shows **session accept_rate**, not **experience adoption_rate**
- If no sessions with injection have `user_accepted=True`, the rate will be 0
- `xp analyze` shows a different adoption rate based on `feedback` table

---

### 3. Auto-Infer Keyword Matching Logic

**Location:** `knowledge.py` lines 438-482 in `infer_adoption()`

```python
def infer_adoption(...):
    # 1. Extract keywords from experience
    exp_keywords = exp.metadata.keywords if exp.metadata.keywords else _extract_keywords(exp_content)
    
    # 2. Match keywords in final response (case-insensitive, substring match)
    for kw in exp_keywords:
        if len(kw) > 2 and kw.lower() in final_lower:
            matched.append(kw)
    
    # 3. Calculate match rate
    match_rate = len(matched) / len(exp_keywords)
    
    # 4. Adoption logic
    if match_rate >= 0.3:   # 30%+ keywords matched
        adopted = True
    elif match_rate >= 0.1: # 10-30%
        adopted = True (low confidence)
    else:
        adopted = False
```

**Why keyword matching might be very low:**

1. **Keyword extraction is weak** (`storage.py` lines 133-142):
   ```python
   def _extract_keywords(text: str) -> list[str]:
       tokens = _tokenize(text)  # Simple regex: [a-z]+ or Chinese chars
       stopwords = {'the', 'a', 'is', ...}  # Small English/Chinese stopword list
       keywords = [t for t in tokens if len(t) > 1 and t not in stopwords]
       return list(set(keywords))  # No phrase extraction, no code awareness
   ```
   - **No n-gram/phrases**: Only single tokens, so "useEffect cleanup" becomes ["useeffect", "cleanup"]
   - **No code pattern extraction**: Can't identify code patterns like `[]` dependency array
   - **Simple substring matching**: "react" in final_lower will match "react", "reactive", "reaction" etc.

2. **Threshold is 30% for high-confidence adoption**: If experience has 10 keywords and only 2 match, `match_rate = 0.2` → low confidence adoption

3. **Short keyword filter**: `len(kw) > 2` filters out short but meaningful terms like "id", "ui", "db"

---

### 4. Key Data Flow

```
MCP Tools Called by Agent
         ↓
    record_session()
         ↓
    sessions table (storage.py:60-67)
         ↓
    get_stats() queries sessions table (storage.py:569-577)
         ↓
    xp stats displays results
```

**Important:** 
- `had_error_correction`, `iteration_count`, `user_accepted` are **self-reported by the agent** calling `record_session`
- The system trusts whatever the agent reports — no validation

---

### Root Causes of Your Anomaly

| Issue | Root Cause |
|-------|------------|
| **Error rate higher with injection** | Selection bias: harder tasks trigger experience injection. The grouping logic (`json_array_length > 0`) only counts sessions where the agent actually called `record_session` with non-empty `experience_ids_injected`. |
| **0% adoption rate** | `xp stats` shows `user_accepted` rate, NOT experience adoption. If agents aren't calling `infer_adoption` or `record_feedback`, the feedback table stays empty. |
| **Low keyword matching** | Keyword extraction is naive (single tokens only, no phrases, no code patterns). Matching is simple substring, so semantic equivalence isn't captured. |

---

### Recommendations for the User

1. **Check if `infer_adoption` is being called**: Look at the `feedback` table — if empty, auto-infer isn't running
2. **Check session data**: Look at `sessions` table to see actual `experience_ids_injected` values
3. **Keyword extraction is too simple**: Consider enhancing `_extract_keywords` to handle:
   - Code snippets (extract function names, patterns)
   - Multi-word phrases (n-grams)
   - Domain-specific vocabulary
