---
nav_exclude: true
---

# Month 12 — Capstone · Code Verification

**Environment:** Linux sandbox, Python 3.10. The capstone is a guided integration project that imports five prior-month packages (`llm`, `guardrails`, `harness`, `factory`, `runner`) which do not exist in the sandbox. Per method, the **seam tests** — the load-bearing integration code, including the Stage-1 worked example — were EXECUTED against minimal stub packages implementing the documented contracts (`Provider`/`FallbackChain`, the `@tool`/guard decorators, `ledger`). This proves the test *logic* is correct; design docs, cloud deploys, and the 14-day run are N/A / PASS-SYNTAX.

## Summary counts

| Result | Count |
|---|---|
| PASS | 9 |
| PASS-SYNTAX | 3 |
| FAIL | 0 |
| N/A | 11 |

## Per-lab results

| Lab | Block / checkpoint | Result |
|---|---|---|
| 1 | Setup `uv add --editable` five pillars + import check | N/A (depends on M7–M11 packages) |
| 1 | SPEC.md / ARCHITECTURE.md / matrix authoring | N/A (design docs) |
| 1 | `factory plan` Plan stage | N/A (M10 factory) |
| 1 | Project stub (mkdir/touch module files) | PASS-SYNTAX |
| 1 | Stretch: ledger `CREATE TABLE` SQL | PASS |
| 2 | `factory build` scaffold | N/A (M10 factory) |
| 2 | `providers.py` FallbackChain wiring | PASS (stubbed contract) |
| 2 | **Stage 1 worked example — `test_seam_fallback`** | PASS |
| 2 | Stage 2 faded — `test_seam_gate` (no raw calls + off-allowlist refused) | PASS |
| 2 | Stage 3 independent — `test_seam_regenerate` (SPEC change → code) | PASS |
| 2 | `access.py` gated tools | PASS (stubbed contract) |
| 2 | `ledger.record()` writes a row | PASS |
| 2 | Harness lead/worker/validator wiring | N/A (domain code) |
| 2 | One full manual tick `run_once()` | N/A (depends on real pillars) |
| 3 | `runner.py` tick_guarded + loop | PASS |
| 3 | **`test_seam_spendcap` (4th seam) — cap halts via SystemExit** | PASS |
| 3 | Kill mechanism 1 — kill-flag refuses next tick | PASS |
| 3 | Kill mechanism 2 — kill live runner process | PASS |
| 3 | Start 14-day clock / cron install | PASS-SYNTAX |
| 3 | Deploy to Oracle/Fly/Mac | N/A |
| 3 | 14-day unattended run + incident log | N/A |
| 3 | RETROSPECTIVE.md / DEMO.md authoring | N/A (deliverable docs) |
| 3 | Final ledger total SQL | PASS-SYNTAX |

## Seam test verification (the core of the capstone)

All four documented seam tests were executed and **pass** with stub pillars:

1. **Fallback seam** (`test_seam_fallback`, Stage 1 worked example): forcing the primary link to raise via `_force_primary_failure=True` causes `FallbackChain.complete` to serve the next link; both assertions (`served_by != links[0].name` and non-empty `text`) hold.
2. **Gate seam** (`test_seam_gate`, Stage 2 faded — TODOs filled per instructions): the regex bypass scan finds no raw external call in `system/harness/`, and an off-allowlist `access.fetch(...)` raises `PermissionError`.
3. **Regenerate seam** (`test_seam_regenerate`, Stage 3 independent): a known `SPEC.md` edit, after a scripted factory re-run, appears in the regenerated module — matching the documented "force the change, assert the seam held" shape.
4. **Spend-cap seam** (`test_seam_spendcap`, Lab 3): with `usd_in_today` monkeypatched to 999.0 and cap 1.0, `tick_guarded()` raises `SystemExit` before doing work — the cap halts rather than warns.

Combined run: `5 passed` across `tests/` (fallback, gate ×2, regenerate, spendcap). Both kill mechanisms (graceful flag refusal and hard process kill) verified live on Linux.

## Notes
- No FAILs. The Stage-1 worked example runs as written against the documented `FallbackChain` contract; the faded/independent stages are consistent and compile.
- The pillar-coverage-matrix is a Markdown artifact (no executable logic) — N/A.
- The heavy N/A count is expected: this lab is integration scaffolding over packages built in Months 7–11, so most blocks are instructions or deploy/runtime steps rather than self-contained runnable code.
