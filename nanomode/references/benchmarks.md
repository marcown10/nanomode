# NanoMode Benchmarks

Token savings by task type and compression level.
Reference data — load only if user asks about expected savings or to tune level choice.

---

## By Task Type

| Task | Baseline | Nano | Micro | Raw | Best level |
|---|---|---|---|---|---|
| Explain a bug | ~1100 | ~160 (-85%) | ~90 (-92%) | ~45 (-96%) | micro |
| Fix auth/middleware | ~700 | ~120 (-83%) | ~70 (-90%) | ~35 (-95%) | nano |
| Setup DB connection pool | ~2300 | ~380 (-84%) | ~200 (-91%) | ~80 (-97%) | micro |
| Git rebase vs merge | ~700 | ~290 (-58%) | ~180 (-74%) | ~100 (-86%) | nano |
| Refactor callback → async | ~390 | ~300 (-23%) | ~220 (-44%) | ~140 (-64%) | nano |
| Microservices vs monolith | ~450 | ~310 (-31%) | ~200 (-56%) | ~120 (-73%) | nano |
| PR security review | ~680 | ~400 (-41%) | ~250 (-63%) | ~150 (-78%) | nano |
| Docker multi-stage build | ~1040 | ~290 (-72%) | ~170 (-84%) | ~80 (-92%) | micro |
| Debug race condition | ~1200 | ~230 (-81%) | ~130 (-89%) | ~60 (-95%) | micro |
| Implement React pattern | ~3450 | ~460 (-87%) | ~250 (-93%) | ~100 (-97%) | micro |
| Code review (long PR) | ~5000 | ~800 (-84%) | ~450 (-91%) | ~200 (-96%) | micro |
| Infra config (k8s/HAProxy) | ~2000 | ~350 (-82%) | ~200 (-90%) | ~90 (-95%) | micro |
| **Average** | **~1670** | **~340 (-80%)** | **~185 (-89%)** | **~85 (-95%)** | |

---

## Recommended Level by Use Case

| Use case | Recommended level | Reason |
|---|---|---|
| Quick debug session | nano | Readable + fast |
| Dense coding session (>30 min) | micro | High signal/noise |
| Infrastructure config generation | micro | Structure > prose |
| API/config reference lookup | raw | Just the values |
| Conceptual explanation | nano | Some prose needed |
| PR review | micro | Structured feedback |
| Log analysis | micro | Pattern recognition |
| Long context conversation | micro | Context budget matters |

---

## Science

A March 2026 paper "Brevity Constraints Reverse Performance Hierarchies in Language Models"
(arxiv.org/abs/2604.00025) found that constraining LLMs to brief responses improved accuracy
by 26 percentage points on certain benchmarks. Less token ≠ less correct. Often more correct.

Key insight: compression forces Claude to identify the actual answer rather than surrounding it
with scaffolding. The act of compression is itself a form of reasoning.

---

## Important Notes

- Savings shown are for **output tokens only**. Input/thinking tokens unaffected.
- Raw savings vary significantly by task verbosity. Conceptual tasks compress less than procedural.
- Token counts vary ±15% depending on exact prompt phrasing.
- At `raw` level, readability degrades for users unfamiliar with dense formats — recommend `micro` as default for most workflows.
