<div align="center">
  <img src="logo.svg" width="200" alt="NanoMode logo" />
  <h1>NanoMode</h1>
  <p><strong>-83% output tokens. Full technical accuracy. No style drift.</strong></p>
  <p>A Claude Code skill that compresses responses using 6 explicit rules — eliminating wasted tokens without losing information.</p>

  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
</div>

---

## Install

```bash
mkdir -p ~/.claude/skills
curl -L https://github.com/marcown10/nanomode/archive/refs/heads/main.zip -o nanomode.zip
unzip nanomode.zip
cp -r nanomode-main/nanomode ~/.claude/skills/
rm -rf nanomode.zip nanomode-main
```

Or clone directly:

```bash
git clone https://github.com/marcown10/nanomode.git
cp -r nanomode/nanomode ~/.claude/skills/
```

Then in Claude Code:

```
nanomode
```

Claude responds: `[NanoMode V2] ON.`

---

## What it does

NanoMode compresses Claude's output using 6 systematic rules — eliminating wasted tokens without losing technical accuracy. Rules are explicit and encoded in the skill, so Claude follows them consistently across all content types and long sessions.

### The 6 Compression Rules

| Rule | Example |
|---|---|
| Kill auxiliary verbs | `cause: X` not `The cause is X` |
| Symbol compression | `137=OOM \| 0=done \| 143=SIGTERM` |
| Remove connector prose | delete "To fix this, run:" before a command |
| Abbreviate paths | `sshd_config.d/*.conf` not `/etc/ssh/sshd_config.d/*.conf` |
| Collapse parallel items | `P=ok \| L=locked` not a 3-line list |
| No preamble / no sign-off | start with the answer, end with the last useful token |

### Always banned

```
Sure | Of course | Great question | I'd be happy to | Let me explain
I hope this helps | Let me know if you have questions
It might be worth | You could potentially | Depending on your setup
```

---

## Modes

| Command | Mode | Token reduction |
|---|---|---|
| `nanomode` or `/nano` | **V2 dense** (default) | **-83%** |
| `/structured` | V3 adaptive patterns | -71% |
| `/micro` | pure key:value | -85%+ |
| `/raw` | max symbol density | -88%+ |
| `normal mode` | off | — |

### V2 dense (default)

Maximum compression. All 6 rules. Minimal structure overhead.

```
cause: new obj ref each render → shallow compare fails → re-render
fix:
  inline obj → useMemo(() => ({...}), [deps])
  inline fn  → useCallback(() => fn(), [deps])
  child      → React.memo(Child)
debug: DevTools Profiler
```

### V3 structured (`/structured`)

Same 6 rules + adaptive response pattern based on question type. Costs ~34 tokens more than V2 but every response is scannable in 2 seconds.

Claude classifies the question internally (zero token cost) then applies the optimal pattern:

| Pattern | Signal | Structure |
|---|---|---|
| `[DEBUG]` | "not working", "crashing" | fix-first → diag → rare → escape |
| `[SETUP]` | "how do I install", "configure" | config-first → usage → gotchas |
| `[FIX]` | specific error, "fix this" | before/after → why → verify |
| `[COMPARE]` | "vs", "which is better" | verdict → properties → when to deviate |
| `[REVIEW]` | "review my code" | verdict → ✗/~/✓ by severity |

**`[DEBUG]` example — K8s CrashLoopBackOff:**
```
[DEBUG] container exits → K8s backoff loop (10s→20s→40s→5min)

diag:
  kubectl logs <pod> --previous   → crash output, start here
  kubectl describe pod <pod>      → exit code in Last State + Events

exit→fix:
  1   → app bug → read --previous
  137 → OOM → increase limits.memory
  143 → liveness too aggressive → increase initialDelaySeconds
  0   → CMD is not long-running → fix entrypoint

stuck? kubectl run d --image=<img> -it --restart=Never -- /bin/sh
```

**`[FIX]` example — JWT expiry:**
```
[FIX] don't check manually — jwt.verify() handles expiry

before: if (payload.exp < Date.now()) ...
after:
  try {
    req.user = jwt.verify(token, SECRET);
    next();
  } catch (err) {
    res.status(401).json({ error: err.name === 'TokenExpiredError' ? 'expired' : 'invalid' });
  }

why:    exp in seconds | Date.now() in ms | < instead of <= lets expired tokens through
verify: expired token → 401 expired ✓
```

**`[COMPARE]` example — Git rebase vs merge:**
```
[COMPARE] merge on shared branches, rebase only locally

merge:  merge commit | exact history | safe | hashes unchanged
rebase: linear history | rewrites hashes → breaks others who already pulled

use merge:  git merge --no-ff feature  (integrate into main)
use rebase: git rebase origin/main     (update feature | clean before PR)
⚠           never rebase already-pushed/shared branch
```

---

## Benchmarks

Real token counts measured with word-level tokenization (×1.3 multiplier, standard approximation for Claude's BPE tokenizer). 8 technical prompts across debugging, setup, explanation, and infra topics.

| Prompt | Normal | NanoMode V2 | NanoMode V3 |
|---|---|---|---|
| React re-render | 298 | **46** | 84 |
| JWT expiry | 334 | **48** | 91 |
| PostgreSQL pool | 316 | **52** | 68 |
| Git rebase vs merge | 337 | **58** | 83 |
| Docker crash | 314 | **58** | 88 |
| HAProxy 503 | 312 | **60** | 96 |
| SSH root login | 273 | **52** | 94 |
| K8s CrashLoop | 296 | **53** | 107 |
| **Average** | **310** | **53** | **89** |
| **vs Normal** | — | **-83%** | -71% |

V3 costs ~34 more tokens per response vs V2 but delivers predictable, scannable structure.

### Why the 6 rules work

The techniques with the highest per-pattern impact:
- **Symbol compression**: `137=OOM | 0=done` instead of sentences → -69% on those patterns
- **Connector prose removal**: deleting "To fix this, run:" before every command → -71%
- **Path abbreviation**: `sshd_config.d/*` vs full paths → -40%
- **Inline collapse**: `P=ok | L=locked` instead of bullet lists → -47%

---

## What NanoMode never compresses

| Element | Rule |
|---|---|
| Code blocks | Always full |
| Error messages | Verbatim |
| Technical terms | Exact — polymorphism stays polymorphism |
| Security warnings | Full |
| Destructive ops | `rm -rf`, `DROP TABLE`, prod deploys — warn fully |

---

## File structure

```
nanomode/
├── SKILL.md                    # Main skill file — loaded by Claude Code
└── references/
    ├── banned-patterns.md      # Extended banned phrase list with regex
    └── benchmarks.md           # Token savings by task type and level
```

`references/` files are lazy-loaded — only when needed for edge cases. The main `SKILL.md` is the only file Claude reads on every activation.

---

## Switch between modes mid-session

```
/structured   → switch to V3 (complex debug sessions, team use)
/nano         → switch back to V2 (maximum compression)
normal mode   → turn off NanoMode
```

---

## Why explicit rules

Most token-saving approaches rely on style hints — vague instructions like "be concise". NanoMode uses explicit, enumerated rules instead:

- **No style drift** — rules don't degrade over long conversations
- **Consistent across content types** — applies to prose, paths, exit codes, option lists, commands
- **V3 structured mode** — readable `[DEBUG]`/`[FIX]`/`[COMPARE]` patterns for complex sessions
- **Infra-aware** — built with DevOps/SRE workflows in mind (HAProxy, K8s, SSH, Docker)

---

## Science

A March 2026 paper ["Brevity Constraints Reverse Performance Hierarchies in Language Models"](https://arxiv.org/abs/2604.00025) found that constraining LLMs to brief responses improved accuracy by 26 percentage points on certain benchmarks. Less tokens ≠ less correct. Often more correct.

---

## License

MIT — compress freely.

## Credits

Marown
