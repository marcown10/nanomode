---
name: NanoCompact
description: >
  Compresses conversation history into a dense summary to reduce input token accumulation
  in long sessions. Use when the user says "/nc", "nanocompact", "compact context",
  "session is getting long", or "losing context". Also pairs with NanoMode — NanoMode
  compresses output per response, NanoCompact compresses the accumulated input context.
  Trigger when conversation exceeds ~15 exchanges or user seems to be repeating context.
---

# NanoCompact

NanoMode compresses output tokens per response.
NanoCompact compresses the input context that accumulates over a session.

Together they cover both sides of token consumption.

---

## Activation

| Command | Action |
|---|---|
| `/nc` | Standard compact — preserve decisions, eliminate noise |
| `/nc deep` | Aggressive — eliminate all exploration, keep only confirmed facts |
| `/nc status` | Show session state without compacting |

---

## What NanoCompact does

When triggered, Claude reads the entire conversation history and produces a **dense session summary** structured by information value. This summary replaces the verbose history as the working context.

The summary has four sections — always in this order:

```
session: <one-line description of what this session is about>
stack:   <tech stack, versions, environment — only what's been mentioned>

resolved:
  · <problem> → <fix applied> ✓
  · <problem> → <fix applied> ✓

active:
  · <current problem being worked on>
  · <current hypothesis or suspicion>
  · next: <exact next step>

files:
  · <path> (<what was changed or why it's relevant>)

discarded:
  · <hypothesis> → excluded (<why>)
```

Only include sections that have content. If nothing is resolved yet, omit `resolved:`. If no files were touched, omit `files:`.

---

## Classification rules

Before writing the summary, classify every exchange in the conversation:

### KEEP (compress and include)

- Fix applied and confirmed working → `resolved:`
- Architectural decision made → `active:` or inline context
- Command executed successfully → `files:` or `resolved:`
- Exact error message that led to the solution → keep verbatim
- File paths and config keys relevant to active problem → `files:`
- Current hypothesis not yet confirmed → `active:`

### ELIMINATE (drop entirely)

- Hypotheses explored and ruled out → move to `discarded:` as one line
- Sequences of failed attempts → keep only final result
- Explanations of concepts the user already applied → drop
- Filler, hedging, connector prose → drop (NanoMode already handles this per-response)
- Clarifying questions that were answered → drop, keep only the answer
- Repeated context across multiple messages → one occurrence only

### COLLAPSE (merge into one entry)

- Multi-turn debugging of same issue → `cause: X → fix: Y ✓`
- "Try this... no wait... try that" sequences → final working command only
- Context restated across messages → deduplicate

---

## Compression rules (same as NanoMode)

Apply inside the summary:

- `→` for causation / sequence
- `|` for alternatives
- `=` for assignments / must-be
- `✓` for confirmed working
- `✗` for confirmed not working
- No auxiliary verbs: `fix: X` not `The fix was X`
- No preamble: summary starts immediately with `session:`
- Abbreviate paths when unambiguous

---

## Example

**Before /nc** (28 exchanges, ~4.200 input tokens accumulated):

```
[user] my K8s pod keeps restarting
[claude] cause: container exits → backoff loop. check kubectl logs --previous...
[user] exit code is 137
[claude] 137 = OOM. increase limits.memory...
[user] changed to 256Mi still crashing
[claude] try 512Mi, also check liveness probe...
[user] 512Mi works but still crashing intermittently  
[claude] liveness probe too aggressive, increase initialDelaySeconds...
[user] set to 30s, still seeing crashes
[claude] check if secret is mounted...
[user] found it — DB_PASSWORD secret not in namespace
... (20 more exchanges)
```

**After /nc** (~180 input tokens):

```
session: debug K8s CrashLoop — pod auth-service, namespace production
stack: Node.js 18 | K8s 1.28 | Deployment auth-service

resolved:
  · exit 137 (OOM) → limits.memory: 512Mi ✓
  · liveness probe too aggressive → initialDelaySeconds: 30 ✓
  · DB_PASSWORD secret missing from namespace → created ✓

active:
  · pod still crashes intermittently after all fixes
  · suspicion: readinessProbe failing on slow startup
  · next: kubectl describe pod auth-xxx → check readinessProbe config

files:
  · k8s/deployments/auth.yaml (limits, liveness, readiness)
  · k8s/secrets/production.yaml (DB_PASSWORD added)

discarded:
  · network policy blocking → excluded, other pods unaffected
  · image pull issue → excluded, ImagePullBackOff not in events
```

**Token reduction: ~4.200 → ~180 input tokens (-96%)**

---

## /nc deep

More aggressive. In addition to standard rules:
- Remove `discarded:` section entirely (already resolved, no longer relevant)
- Remove `files:` if files are already stable (no active edits)
- Collapse `stack:` to one line
- Keep only `session:` + `active:` + confirmed resolutions

Use when approaching context window limit and need maximum space.

---

## /nc status

Read-only — does not compact. Produces the same summary format but outputs it as a status report without replacing the context. Use to check session state mid-session without losing history.

```
[NanoCompact: status]
session: ...
resolved: N items
active: ...
```

---

## Works with NanoMode

If NanoMode is active, NanoCompact inherits its compression symbols and rules.
The summary output follows NanoMode formatting automatically.

If NanoMode is not active, NanoCompact still produces dense output — it does not require NanoMode to function.

---

## Reference files

- `references/taxonomy.md` — detailed classification rules for edge cases
