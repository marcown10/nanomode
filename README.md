<div align="center">
  <img src="logo.svg" width="180" alt="NanoMode logo" />
  <h1>NanoMode</h1>
  <p>Two Claude Code skills. One solves verbose output. One solves context bloat.</p>

  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
</div>

---

## The problem

Claude wastes tokens on both sides of every conversation:

```
output: "Sure! I'd be happy to help. The reason your container is restarting
         is likely because the main process is exiting..."
→ fix:  cause: main process exits → restart loop
        docker logs <id> | exit 137=OOM | 1=err | 0=cmd done

input:  after 30 exchanges, Claude reloads the entire verbose history
        every single message — even the parts that are long solved
→ fix:  /nc compresses 4.200 accumulated tokens → 180 token summary
```

**NanoMode** fixes the output. **NanoCompact** fixes the input.

---

## Install both

```bash
git clone https://github.com/marcown10/nanomode.git /tmp/nano
cp -r /tmp/nano/nanomode ~/.claude/skills/
cp -r /tmp/nano/nanocompact ~/.claude/skills/
rm -rf /tmp/nano
```

---

## NanoMode — output compression

**Activate:** type `nanomode` → `[NanoMode V2] ON.`

### The 6 rules (always active)

| Rule | Before | After |
|---|---|---|
| Kill auxiliary verbs | `The cause is that X` | `cause: X` |
| Symbol compression | `leads to a re-render` | `→ re-render` |
| Remove connector prose | `To fix this, run: docker logs` | `docker logs <id>` |
| Abbreviate paths | `/etc/ssh/sshd_config.d/*.conf` | `sshd_config.d/*.conf` |
| Collapse parallel items | 3-line bullet list | `0=done \| 1=err \| 137=OOM` |
| No preamble / sign-off | `Sure! Great question. I hope this helps!` | *(nothing)* |

### Always banned
```
Sure | Of course | Great question | I'd be happy to | Let me explain
I hope this helps | Let me know if you have questions
It might be worth | You could potentially | Depending on your setup
```

### Modes
| Command | Mode | Saving |
|---|---|---|
| `nanomode` / `/nano` | V2 dense (default) | **-83%** |
| `/structured` | V3 adaptive patterns | -71% |
| `/micro` | pure key:value | -85%+ |
| `/raw` | max symbol density | -88%+ |
| `normal mode` | off | — |

### V2 output example
```
cause: new obj ref each render → shallow compare fails → re-render
fix:
  inline obj → useMemo(() => ({...}), [deps])
  inline fn  → useCallback(() => fn(), [deps])
  child      → React.memo(Child)
debug: DevTools Profiler
```

### V3 patterns (`/structured`)
Claude classifies the question (zero token cost) → applies optimal structure:

| Pattern | When | Structure |
|---|---|---|
| `[DEBUG]` | "not working", "crashing" | fix-first → diag → escape |
| `[SETUP]` | "how do I install/configure" | config → usage → gotchas |
| `[FIX]` | specific error | before/after → why → verify |
| `[COMPARE]` | "vs", "which is better" | verdict → props → edge cases |
| `[REVIEW]` | "review my code/config" | verdict → ✗/~/✓ by severity |

**`[DEBUG]` example:**
```
[DEBUG] container exits → K8s backoff (10s→20s→40s→5min)

diag:
  kubectl logs <pod> --previous   → crash output, start here
  kubectl describe pod <pod>      → exit code + Events

exit→fix:
  1   → app bug → read --previous
  137 → OOM → increase limits.memory
  143 → liveness aggressive → increase initialDelaySeconds
  0   → CMD not long-running → fix entrypoint

stuck? kubectl run d --image=<img> -it --restart=Never -- /bin/sh
```

**`[REVIEW]` example:**
```
verdict: works but 2 blocking issues

✗ DB_PASSWORD in plaintext → .env + .gitignore | or Docker secrets
✗ no restart policy → restart: unless-stopped
~ version: '3' deprecated → remove line
~ no healthcheck → orchestrator can't detect unready app
✓ port mapping 80:3000 correct
```

### Never compressed
Code blocks · error messages (verbatim) · technical terms · security warnings · destructive ops (`rm -rf`, `DROP TABLE`, prod deploys)

### Benchmarks
8 prompts: React, JWT, PostgreSQL, Git, Docker, HAProxy, SSH, K8s.

| | Normal | V2 | V3 |
|---|---|---|---|
| avg tokens | 310 | **53** | 89 |
| vs Normal | — | **-83%** | -71% |

---

## NanoCompact — input compression

**Activate:** type `nanocompact`
**Use:** `/nc` during a long session

NanoMode reduces output per response. But after 30 exchanges, Claude reloads the entire conversation history as input every message — including all the already-solved problems, failed attempts, and repeated context.

`/nc` reads the full history, classifies every exchange by information value, and produces a dense summary that replaces the verbose history.

### Classification

**Keep:** confirmed fixes · decisions made · active hypothesis · relevant file paths · exact error that led to solution

**Eliminate:** failed attempts → one `discarded:` line · concepts explained and applied · repeated context · clarifying questions already resolved

**Collapse:** multi-turn debug → `cause: X → fix: Y ✓` · failed sequence → final working command only

### Summary format
```
session: <what this session is about>
stack:   <tech | versions | environment>

resolved:
  · <problem> → <fix> ✓

active:
  · <current problem>
  · <current hypothesis>
  · next: <exact next step>

files:
  · <path> (<what changed>)

discarded:
  · <hypothesis> → excluded (<why>)
```

### Example

28 exchanges on K8s CrashLoop → `/nc` →

```
session: debug K8s CrashLoop — pod auth-service, namespace production
stack: Node.js 18 | K8s 1.28

resolved:
  · exit 137 (OOM) → limits.memory: 512Mi ✓
  · liveness probe → initialDelaySeconds: 30 ✓
  · DB_PASSWORD secret missing → created in namespace ✓

active:
  · intermittent crashes remain after all fixes
  · suspect: readinessProbe failing on slow startup
  · next: kubectl describe pod auth-xxx → check readinessProbe

files:
  · k8s/deployments/auth.yaml (limits + probes)
  · k8s/secrets/production.yaml (DB_PASSWORD)

discarded:
  · network policy → excluded, other pods unaffected
  · image pull → excluded, no ImagePullBackOff in events
```

**~4.200 input tokens → ~180 tokens. -96%.**

### Commands
| Command | Action |
|---|---|
| `/nc` | Standard compact |
| `/nc deep` | Aggressive — session + active + resolved only |
| `/nc status` | Read-only status, no compacting |

### Works with NanoMode
If NanoMode is active, NanoCompact uses the same symbols (`→ \| = ✓ ✗`).
If NanoMode is off, NanoCompact still works independently.

---

## File structure

```
nanomode/
├── SKILL.md
└── references/
    ├── banned-patterns.md
    └── benchmarks.md

nanocompact/
├── SKILL.md
└── references/
    └── taxonomy.md
```

---

## Why explicit rules beat "be concise"

Vague style prompts degrade over long sessions — Claude reverts to verbose defaults. Both skills use SKILL.md files loaded fresh each session with enumerated, testable rules. No drift.

---

## Science

["Brevity Constraints Reverse Performance Hierarchies in Language Models"](https://arxiv.org/abs/2604.00025) (March 2026) — brief responses improved accuracy by 26pp on certain benchmarks. Less tokens ≠ less correct.

---

## License

MIT — compress freely.

## Credits

Built by Marco — Senior SysAdmin and DevOps engineer. Designed for real infra work: HAProxy, Kubernetes, SSH, Docker, and long sessions where every token counts.
