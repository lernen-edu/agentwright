---
nav_exclude: true
---

# Month 10 — Software Factories: Code Verification

**Environment:** Linux sandbox, Python 3.10 (labs target 3.12). pydantic 2.13, fastapi
0.136, httpx, ruff 0.15, mypy 2.1, pytest 9 + pytest-cov all present. The Month 7 `llm`
package was stubbed with a stage-aware fake `make_client(...)` (keyed off each stage's
system prompt; accepts `temperature`, exposes usage) producing a valid `Spec`, a SCOUT
brief, real `/health` code, a real test, and approve/reject verdicts. **Model stages
stubbed; the tool gates ran for REAL** — `python3 -m ruff`, `python3 -m mypy`,
`python3 -m pytest --cov` against a scaffolded FastAPI target app, plus real local `git`.
`uv run X` was substituted with `python3 -m X` in subprocess gate calls (noted). No paid
APIs. Built/executed in `/tmp/m10` (`factory/` + `targetapp/`).

## Summary counts

| Status | Count |
|---|---|
| PASS | 24 |
| PASS-SYNTAX | 4 |
| FAIL | 0 |
| N/A | 3 |

PASS-SYNTAX = blocks needing real GitHub (`gh auth`, `git push origin`, `gh pr create`,
`gh pr list/view`) or live Ollama/uv setup. N/A = `zsh` setup heredocs/`mkdir` and prose
markdown templates (SPEC.md, RETRO.md, features.txt).

## Per-lab results

| Lab | Block / checkpoint | Status | Notes |
|---|---|---|---|
| 1 | Setup (`uv init`, `ollama pull`) | PASS-SYNTAX | needs uv venv + Ollama |
| 1 | `factory/spec.py` Spec + validator | PASS | empty `acceptance_criteria` → ValidationError |
| 1 | `examples/specs.py` three hand specs | PASS | all three parse (incl. cross-cutting) |
| 1 | Stage 1 worked `prompts.py`+`plan.py` | PASS | `plan().to_markdown()` populated criteria |
| 1 | Stage 2 faded `plan_with_default_then_coder` | PASS | cheap→coder fallback returns Spec |
| 1 | Stage 3 indep `plan_strict` | PASS | rejects single-word criteria; all multi-word |
| 1 | step4 iterate→`SPEC.md` heredoc | N/A | prose template |
| 1 | DoD self-verify (`assert s.acceptance_criteria`) | PASS | "PLAN ok: ..." |
| 2 | targetapp scaffold + baseline `pytest` | PASS | `1 passed`; ruff+mypy clean baseline |
| 2 | `factory/stage.py` StageResult/StageMetrics | PASS | dataclasses import |
| 2 | `factory/trace.py` RunTrace (M9 reuse) | PASS | imports, emits |
| 2 | SCOUT (read-only brief) | PASS | cites `app/main.py`; ok=True on real files |
| 2 | BUILD (cold; jailed `_safe_write`) | PASS | writes `/health` route; ast-parses; diff --stat |
| 2 | BUILD jail rejects `../escape.txt` | PASS | raises ValueError |
| 2 | VALIDATE `ruff`+`mypy` (clean) | PASS | real tools exit 0 → ok=True |
| 2 | VALIDATE catches injected type error | PASS | mypy `[operator]` error in detail; ok=False |
| 2 | TEST real `pytest --cov` ≥ floor | PASS | writes test_health.py; 100% cov, `2 passed` |
| 2 | REVIEW approve on correct diff | PASS | verdict approve |
| 2 | REVIEW reject out-of-scope `/version` | PASS | rejects + cites out-of-scope (TEST/VALIDATE can't) |
| 2 | Stage 1 worked `pipeline.py` end-to-end | PASS | True; 7 trace events plan→…→pipeline |
| 2 | Stage 2 faded SCOUT single retry | PASS | logic consistent, emits up to 2 scout events |
| 2 | Stage 3 indep `build_attempts` counter | PASS | final event carries int matching build() calls |
| 2 | **Gate-failure feedback loop** | PASS | VALIDATE fail → BUILD retry (retries=1) → pass; build_attempts=2 |
| 3 | Setup (`gh auth`, repo create/push) | PASS-SYNTAX | needs GitHub |
| 3 | `factory/pr.py` local git (branch/commit/changelog) | PASS | branch+commit+CHANGELOG.md created for real |
| 3 | `factory/pr.py` `git push` / `gh pr create` | PASS-SYNTAX | no remote → push exits 128 (expected) |
| 3 | `factory/cli.py` end-to-end | PASS | runs full pipeline + run dir/trace; PR push → PASS-SYNTAX |
| 3 | Stage 1 worked `metrics.py` table | PASS | $/feature $0.0000, time-to-PR, success 3/3 |
| 3 | Stage 2 faded `tokens_by_stage` | PASS | aggregates per stage (0 tokens; see note) |
| 3 | Stage 3 indep `most_expensive_feature` | PASS | $0-tie falls back to slowest run |
| 3 | link PRs / RETRO.md / features.txt | N/A | prose/markdown deliverables |

## FAIL details & fixes

None. The full six-stage pipeline (PLAN→SCOUT→BUILD→VALIDATE→TEST→REVIEW) executes with
model stages stubbed and the real tool gates running: `ruff` + `mypy` gate VALIDATE on
exit code, `pytest --cov` gates TEST against a real coverage floor, BUILD is jailed to the
target repo, and REVIEW (fresh diff+spec context) catches an out-of-scope change the other
gates pass. The **retry/feedback loop was proven live**: an injected mypy type error makes
VALIDATE return ok=False, its detail feeds back into BUILD, the retry passes, and the run
ends with `build_attempts: 2`. Local git PR machinery (branch, commit, generated
CHANGELOG.md from the spec) runs for real; only the remote `git push`/`gh pr create` is
un-runnable here (no GitHub) → PASS-SYNTAX, per environment guidance.

## Minor observations (not FAILs)

- **Telemetry tokens are zero in the worked pipeline.** Lab 2's `pipeline.py` emits
  `stage=`/`eval_ok=` but not `tokens_in/out`/`served_by`, so `metrics.cost()` is $0 and
  `tokens_by_stage` totals 0. This is internally consistent — recording per-stage usage is
  an explicit Lab 2 stretch goal and a Lab 3 "If not" hint ("have each model stage emit
  usage"). Cost math is correct; it simply has no tokens to multiply until the stretch is done.
- **`metrics.main` assumes every trace line has a `stage` key** (`e["stage"]`). All emitted
  events include it, so no failure; a malformed external line would KeyError (the lab's own
  "If not" calls this out).
- **Worked example focus (pedagogy edit).** The Stage-1 "I do" blocks in all three labs
  (plan.py, the stage modules, pipeline.py, metrics.py) all run verbatim against the
  fakes/real tools; the faded (Stage 2) and independent (Stage 3) pieces compile and run
  when filled per the stated solution shape — consistent across the gradient.
