# NanoCompact — Classification Taxonomy

Detailed rules for classifying conversation content during /nc.
Load this file for edge cases not covered by the main SKILL.md.

---

## By content type

### Code blocks
- Working code that was applied → KEEP as inline snippet (max 5 lines) or file reference
- Code that was attempted and failed → ELIMINATE
- Code explaining a concept → ELIMINATE if concept was understood and applied
- Config files modified → KEEP path + key changes only, not full content

### Error messages
- Error that led to the fix → KEEP verbatim (one line)
- Intermediate errors during exploration → ELIMINATE
- Current unresolved error → KEEP verbatim in `active:`

### Commands
- Command executed successfully → KEEP compressed in `resolved:` or `files:`
- Command that failed → ELIMINATE unless failure taught something critical
- Diagnostic command + output that was useful → KEEP as `diag result: X`
- Current next step command → KEEP in `active: next:`

### Explanations
- Explanation of concept user then applied → ELIMINATE (they got it)
- Explanation user questioned or ignored → ELIMINATE
- Explanation still relevant to active problem → KEEP as one-line note in `active:`

### Hypotheses
- Confirmed correct → move to `resolved:`
- Confirmed wrong → move to `discarded:` as one line
- Not yet tested → keep in `active:`
- Partially tested → keep in `active:` with current state

### File changes
- File modified and working → `files:` with path + what changed
- File modified but reverted → ELIMINATE
- File relevant to active problem but not yet modified → `active:` not `files:`

---

## Multi-turn patterns

### Debug loop (most common)
```
Pattern: try X → fail → try Y → fail → try Z → works
Compact to: fix: Z ✓ (tried X, Y — excluded)
```

### Clarification chain
```
Pattern: question → answer → follow-up → answer → ...
Compact to: final answer only, all context collapsed
```

### Architecture discussion
```
Pattern: option A vs B discussion → decision
Compact to: decision: B (A excluded: reason)
```

### Repeated context
```
Pattern: user restates same problem across messages
Compact to: first clean statement only
```

---

## Edge cases

### When nothing is resolved
Omit `resolved:` entirely. Do not write `resolved: nothing yet`.
Only include `session:` + `stack:` + `active:`.

### When session has multiple distinct problems
Split `active:` into labeled sub-problems:
```
active:
  [auth] secret mounted, still failing → check RBAC
  [deploy] image pull working → ✓ resolved, remove from active
```

### When user switches topic mid-session
Previous topic moves entirely to `resolved:` or `discarded:`.
New topic becomes the new `active:`.
Keep `files:` from previous topic only if still relevant.

### When fix is partial
```
resolved:
  · OOM → limits.memory: 512Mi ✓ (intermittent crashes remain)
active:
  · intermittent crashes after memory fix → readinessProbe suspect
```

### When /nc deep removes too much
User can say `/nc restore` to signal the compact was too aggressive.
In that case, regenerate from history with standard (not deep) rules.

---

## Token budget targets

| Mode | Target input tokens after compact |
|---|---|
| `/nc` standard | < 300 tokens |
| `/nc deep` | < 150 tokens |
| `/nc status` | < 300 tokens (read-only, no replacement) |

If the session is genuinely complex (many files, many resolved issues), exceeding these targets is acceptable — accuracy over token budget. But never exceed 500 tokens for a standard compact.

---

## What NOT to do

- Do not include the user's original question verbatim — compress it to topic
- Do not include Claude's full responses — extract only the actionable result
- Do not summarize summaries if /nc was already run earlier — rebuild from full history
- Do not invent context that wasn't in the conversation
- Do not omit unresolved problems to make the summary look cleaner
- Do not include timestamps or message counts — irrelevant to future context
