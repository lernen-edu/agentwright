---
nav_exclude: true
---

# Month 11 — Always-On Agents · Code Verification

**Environment:** Linux sandbox, Python 3.10 (labs assume 3.12 — no syntax used that 3.10 lacks). `sqlite3` CLI absent → all SQLite checks executed via Python's `sqlite3` module (equivalent). Ollama/launchd/cron/cloud unavailable → model calls stubbed, `.plist` validated for XML well-formedness, cron lines field-validated, cloud deploys marked PASS-SYNTAX.

## Summary counts

| Result | Count |
|---|---|
| PASS | 18 |
| PASS-SYNTAX | 7 |
| FAIL | 1 |
| N/A | 4 |

## Per-lab results

| Lab | Block / checkpoint | Result |
|---|---|---|
| 1 | Setup (brew/ollama/uv) | PASS-SYNTAX |
| 1 | Stage 1 `store.py` — dedup (same key twice, 1 row) | PASS |
| 1 | Stage 2 `mark_failed` (attempts=2, status pending) | PASS |
| 1 | Stage 3 `counts()` dict | PASS |
| 1 | `agent.py` drain — each job once, then idle (model stubbed) | PASS |
| 1 | **Durability proof** — crash mid-task leaves pending, resume completes once | PASS |
| 1 | launchd `.plist` XML well-formedness | PASS-SYNTAX |
| 1 | launchctl load/start/unload | N/A (macOS) |
| 1 | cron line (5 fields) | PASS-SYNTAX |
| 2 | `guard_spend` raises BEFORE call; spend durable across reconnect | PASS |
| 2 | Stage 2 `guarded_call` — fn never runs over cap | PASS |
| 2 | Stage 3 per-hour sub-cap | PASS (logic; impl per spec) |
| 2 | `CircuitBreaker` opens after 5, fails fast | PASS |
| 2 | Kill switch `should_stop` (STOP file + DB flag) | PASS |
| 2 | `alert()` reaches webhook (fake localhost server) | PASS |
| 2 | **SafetySupervisor cap_hit at $0.00** — log + alert | **FAIL** |
| 2 | SafetySupervisor breaker opens + breaker alert | PASS |
| 2 | Live `while True` loop exits within 1 iter on `touch STOP` | PASS |
| 2 | Log rotation (dated files) | PASS-SYNTAX |
| 3 | Stage 1 `backoff()` grows + jitter | PASS |
| 3 | Stage 1/2 `process` + `drain` DLQ flow — bad job → DLQ, alert, good jobs done | PASS |
| 3 | Stage 3 `replay_dlq` — restores pending/attempts=0 | PASS |
| 3 | cron line / Dockerfile syntax | PASS-SYNTAX |
| 3 | Oracle/Fly/systemd/Cloudflare deploy | N/A |
| 4 | Stage 1 `config.py` default-False gating | PASS |
| 4 | Stage 2 `maybe_send` — drafts off, sends on; allowlist hard-refuse | PASS |
| 4 | Stage 3 pytest gate (2 cases) | PASS |
| 4 | Deploy + 7-day unattended run | N/A |

## FAIL detail & fix

**Lab 2, Step 5 — `SafetySupervisor.call_model` never logs `cap_hit` / never alerts on a budget exception.**

The checkpoint and Definition-of-Done self-verify require: with `DAILY_CAP_USD=0.00`, one run logs `{"event":"cap_hit"}`, sends an alert, and does not call the model. Executed result: model is correctly NOT called, but **no `cap_hit` log and no alert fire.**

Root cause: `guard_spend(self.db, est_cost)` is called *before* the `try:` block, so `BudgetExceeded` propagates straight out of `call_model` and never reaches the `except (BudgetExceeded,):` clause that does `log(event="cap_hit")` and `alert(...)`. The `except` clause is therefore dead code on the cap path.

Confirmed by execution: capturing stdout on a `$0.00` cap run shows `cap_hit logged? False | alert fired? False`.

One-line fix — move the guard inside the `try`:

```python
self.breaker.before_call()
try:
    guard_spend(self.db, est_cost)   # <-- moved inside try
    out = fn()
    ...
```

Re-ran with the fix: `cap_hit logged? True | alert fired? True`. The other two supervisor safeties (breaker, kill switch) were unaffected and pass as written.

## Notes
- Lab 2 correctly "wraps the Lab 1 agent" per the recent edit; `store.py`/`agent.py` references are consistent across Labs 1–4.
- All SQLite durability/idempotency and DLQ logic is pure Python+sqlite3 and PASSES, including the crash→resume proof.
- Model calls (Ollama) were stubbed; only scheduling/durability/safety logic was under test, which is the labs' stated intent.
