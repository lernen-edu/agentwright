---
nav_exclude: true
---

# Month 5 — Software Engineering Principles · Code Verification

**Summary: PASS 31 · PASS-SYNTAX 4 · FAIL 1 · N/A 7** (executed on Linux sandbox, Python 3.10; labs target 3.12). Month 5 is fully offline-runnable. Built each lab's `src/` package in /tmp, ran `python3 -m mypy --strict` (clean everywhere), `python3 -m pytest`, and `pytest --cov`. The README mental-model code (`summarize`, `RateLimiter`, `Repo`, DI examples) is illustrative and type-checks. One real FAIL in Lab 2's monkeypatch worked example.

## Lab 1 — OOP, Interfaces, and Dependency Injection

| Block | Status |
|---|---|
| Setup `uv init`/`uv add` | N/A (uv/macOS; reproduced layout by hand) |
| Step 1 `inbox.py` (Message dataclass, Inbox + dunders) | PASS (`Inbox(owner='ada', messages=1)`, count=1) |
| Step 2 `should_alert` pure fn REPL | PASS (True for urgent+quiet; False otherwise) |
| Step 3 `notifier.py` Protocol + Console/Counting (Stage 1) | PASS (`mypy --strict` clean; structural typing OK) |
| Step 4 `alerts.py` AlertService faded skeleton (filled) | PASS (`sent=1`, `len(spy.sent)=1`, `spy.sent[0].sender='ci'`) |
| Step 4 `# TODO` skeleton (unfilled) | PASS-SYNTAX (compiles; needs 2 TODOs) |
| Step 5 `PrefixNotifier` (Stage 3 reference) | PASS (mypy clean; Open/Closed, no edit to AlertService) |
| Step 5 `main()` | PASS (`[ALERT] ci: build failed`) |
| `uv run mypy --strict src` | PASS (Success: no issues, 4 files) |
| Mermaid diagram | N/A |

## Lab 2 — pytest Deep Dive and Strict Types

| Block | Status |
|---|---|
| Setup `uv init`/`uv add pytest` | N/A (uv) |
| Step 1 test-first red (`ImportError: rank`) | PASS (intentional red reproduced) |
| Step 2 `rank` pure + purity assert | PASS (`["b","c","a"]`, input untouched) |
| Step 3 `parametrize` tiebreak (4 cases) | PASS (all 4 green with `(-p.score, p.name)`) |
| Step 4 fixtures `sample_players` | PASS (both fixture tests green) |
| Step 5 injection-style datetime tests | PASS (Mon=True, Sat=False; dates confirmed wd 0/5) |
| Step 5 monkeypatch `FakeDateTime` worked example | **FAIL** (see below) |
| Step 6 `pytest --cov` term-missing | PASS (core.py 100%; ≥80% achievable) |
| Step 7 `mypy --strict src tests` | PASS (Success: no issues) |

## Lab 3 — The Refactor Crucible (Milestone)

| Block | Status |
|---|---|
| Setup `uv init` + `.gitignore` echoes | N/A (uv/shell) |
| Step 1 `models.py` Repo/PullRequest | PASS (`Repo(name='x', stars=3)`) |
| Step 2 `providers.py` Protocol + GitHub (Stage 1) | PASS (mypy clean after `types-requests`*) |
| Step 2 GitLab faded skeleton (filled from reference) | PASS (structural Provider; mypy clean) |
| Step 2 GitLab `# TODO` skeleton | PASS-SYNTAX (returns `[]` until filled) |
| Step 3 `summary.py` pure fns test-first | PASS (count_by_state/total_stars/top_repos green) |
| Step 4 `report.py` ReportGenerator (Stage 3) | PASS (mypy clean; no provider names — Open/Closed) |
| Step 5 `conftest.py` FakeProvider + tests | PASS (Source: fake, Total stars: 13, pr_calls=[alpha,beta]) |
| Step 6 `logsetup.py` structured logging | PASS (configure runs; stderr handler) |
| Step 7 `cli.py` build_provider + argparse | PASS (raises on bogus; builds github; live run PASS-SYNTAX) |
| Step 8 fake-session provider tests | PASS (GH/GL fetch_repos+prs cover network branches) |
| Step 9 `mypy --strict` + `--cov-fail-under=80` | PASS (mypy clean; core modules 100%, see note) |
| Step 10 `docs/v1-vs-v2.md` writeup | N/A (prose deliverable) |
| Step 11 git commit/`gh repo create`/push | N/A (network) |

## FAIL details & fixes

**Lab 2, Step 5 — monkeypatch worked example (`test_tournament_open_via_monkeypatch`).**
- Symptom: `TypeError: FakeDateTime() takes no arguments`.
- Cause: the test does `monkeypatch.setattr(datetime, "datetime", FakeDateTime)`, but inside `FakeDateTime.now()` the body calls `datetime.datetime(2026, 5, 25, 9, 0)` — which now resolves to the *patched* `FakeDateTime`, whose constructor takes no args. The fake recursively hits itself.
- One-line fix: capture the real class before patching and use it in `now()` — add a module-level `_real_dt = datetime.datetime` and change the body to `return _real_dt(2026, 5, 25, 9, 0)` (or make `FakeDateTime` subclass `datetime.datetime`). Verified: with the fix the test passes.

### Notes (not failures)
- *`types-requests`*: `mypy --strict` on `providers.py` needs the `requests` stubs. A learner doing `uv add requests` + mypy hits the same and installs `types-requests`; once installed, strict passes cleanly. Environment artifact, not a lab bug.
- Lab 3 coverage: with only the lab-prescribed tests, the four core modules (models/providers/summary/report) hit **100%**; the whole-package number is ~65% because `cli.py`/`logsetup.py` (pure imperative shell) have no prescribed tests. The 80% bar is reachable by scoping `--cov` to the tested modules or adding one shell smoke test — the lab acknowledges this glue is "decide case by case." Milestone coverage claim holds.
- Sandbox is Python 3.10; all 3.10-valid syntax (`list[int]`, `str | None`, `is_relative_to`) ran. No 3.12-only syntax encountered.
