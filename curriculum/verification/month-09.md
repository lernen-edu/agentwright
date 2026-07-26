---
nav_exclude: true
---

# Month 9 — Agent Harnesses: Code Verification

**Environment:** Linux sandbox, Python 3.10 (labs target 3.12). pydantic 2.13 present.
The Month 7 `llm` package and Month 8 `jail.py` (prerequisites) were stubbed: a fake
`make_client(...)` returning a `.complete()` whose canned `.text` varies by requested
model name (so per-role routing and fallback are observable), and a `safe_path` jail.
No Ollama, no paid APIs. `uv run python -m X` was substituted with `python3 -m X`
(libs live in the system Python; `uv run` would rebuild a venv) — orchestration logic
is identical. Subprocess worker spawning ran for real on Linux. Built/executed in
`/tmp/m9`.

## Summary counts

| Status | Count |
|---|---|
| PASS | 23 |
| PASS-SYNTAX | 3 |
| FAIL | 0 |
| N/A | 4 |

PASS-SYNTAX = blocks needing live Ollama/`ollama` CLI (`ollama list/serve/pull/ps`) or a
real second model for timing comparison. N/A = `zsh` setup/`mkdir`/`printf` and Mermaid/
ASCII diagrams and prose markdown templates (SPEC.md, POSTMORTEM.md, DEFENSE.md).

## Per-lab results

| Lab | Block / checkpoint | Status | Notes |
|---|---|---|---|
| 1 | Setup `uv init` / `ollama pull` / `serve` | PASS-SYNTAX | needs uv venv + Ollama daemon |
| 1 | import `pydantic, llm, jail` checkpoint | PASS | prints `ok` against stubs |
| 1 | `harness/config.py` HarnessConfig | PASS | dataclass imports clean |
| 1 | `harness/tools.py` (read-only; jailed) | PASS | `grep_logs` → file:line citations; no write path |
| 1 | Stage 1 worked `detection_context` | PASS | counts only, no bodies (3 summaries) |
| 1 | Stage 2 faded `window_context` (solution) | PASS | numbered window, lo-offset correct, budget slice |
| 1 | Stage 3 indep `rule_context` | PASS | loads only named files, under budget |
| 1 | `harness/run.py` single-agent triage | PASS | emits suspect JSON + evidence-citing root cause |
| 1 | DoD self-verify (`assert len(...) < 1000`) | PASS | "context is engineered" |
| 2 | `harness/worker.py` via stdin (subprocess) | PASS | prints structured JSON result |
| 2 | `harness/validate.py` (Pydantic) | PASS | evidence-free `critical` → `None` |
| 2 | Stage 1 worked `decompose` | PASS | JSON slice list, one per error-bearing file |
| 2 | Stage 1 worked `spawn_worker` (subprocess) | PASS | real subprocess, per-worker workdir created |
| 2 | Stage 2 faded `orchestrate` (solution) | PASS | cheap-first, escalate-on-reject, accepted flags |
| 2 | Stage 3 indep escalation cap | PASS | logic consistent; records rejected + reason |
| 2 | `harness/team.py` end-to-end | PASS | per-slice results + `runs/<ts>/worker-*` dirs |
| 2 | Per-role routing | PASS | model arg threaded per role (3b vs 7b) |
| 2 | Fallback on blackholed base_url | PASS | served_by=`ollama-fallback`; run survives |
| 2 | `tests/test_orchestration.py` | PASS | `3 passed` (validator accept/reject/bad-severity) |
| 3 | `harness/trace.py` RunTrace | PASS | emit+flush+save_input |
| 3 | traced `orchestrate` (faded) | PASS | trace.jsonl + slices.json + worker subdirs |
| 3 | `synthesize` (capable model) | PASS | summary over validated findings only |
| 3 | `harness/replay.py` (independent) | PASS | re-feeds slices.json → `<ts>-replay/trace.jsonl` |
| 3 | three real runs | PASS | 3 distinct `runs/<ts>/` (see timestamp note) |
| 3 | DoD self-verify (`wc -l`; replay clean) | PASS | replay reruns from disk clean |
| 3 | colima / containerized worker (stretch) | PASS-SYNTAX | needs container runtime (excluded) |

## FAIL details & fixes

None. Every load-bearing block executed against the stubs and behaved as its checkpoint
claims: the LEAD decomposes, WORKERS run as real subprocesses each with their own
working directory, the Pydantic VALIDATOR accepts/rejects + escalates, per-role routing
and the fallback chain are exercised, a JSONL trace is written, and trace→replay
reproduces the pipeline from saved inputs.

## Minor observations (not FAILs)

- **Run-dir timestamp collision (latent).** Both Lab 2 (`team.py`) and Lab 3 use
  `time.strftime("%Y%m%d-%H%M%S")` for `runs/<ts>`. Three runs fired within the same
  wall-clock second collapse into one directory (verified: a 1s gap yields three distinct
  dirs). Not a code defect, but learners scripting three back-to-back runs may overwrite.
  One-line hardening: append `-%f` via `datetime.now().strftime(...)` (note: `time.strftime`
  has no microsecond directive) or a short uuid.
- **Stage-2 faded/Stage-3 independent** are presented as TODO skeletons + a stated solution
  shape; filling them per the given solution compiles and runs — consistent.
