---
title: Verification Report
parent: Curriculum
nav_order: 110
---

# Code Verification Report

**Date:** 2026-05-25
**Scope:** Every fenced code block in all 40 labs across the 12 months.
**Method:** Six verification agents, each owning two months, extracted the code and *executed* everything that is deterministic and offline, then syntax/compile-checked the rest.

---

## Headline result

| Status | Count | Meaning |
|---|---:|---|
| **PASS** | **292** | Executed in the sandbox and behaved exactly as the lab's checkpoint claims. |
| **PASS-SYNTAX** | **65** | Not runnable in this environment (needs macOS-only tooling, Ollama, a live/paid API, or containers) but confirmed syntactically valid / compiles. |
| **FAIL** | **2** | Genuinely broken. Both found, fixed, and the fixes re-verified. |
| **N/A** | ~50 | Mermaid diagrams, illustrative output samples, or pseudocode — not meant to execute. |

Net: of everything that *should* run, **99.3% executed correctly on the first pass**, and the 2 defects are now fixed.

---

## What the environment could and couldn't run

The verification sandbox is **Linux with Python 3.10**, while the course targets **macOS with Python 3.12**. That mismatch defines the PASS-SYNTAX bucket — it is not a sign of broken code, just code that needs the real target environment:

- **macOS-only:** `launchd`/`launchctl` plists (validated as well-formed XML instead), Homebrew installs.
- **Local model runtime:** Ollama — every model call was instead driven by a **localhost fake OpenAI-compatible server** or a monkeypatched client, so the surrounding agent loops, orchestration, cost math, and safety logic all executed for real.
- **Live/paid services:** the Anthropic/OpenAI APIs and authenticated GitHub (`gh`, push) — the offline logic around them ran; only the authenticated round-trip is inspection-only.
- **Containers / Postgres:** Colima/Podman and the read-only Postgres role — the Python around them ran; role-level enforcement is inspection-only (modeled with SQLite).

Where a 3.12-only feature appeared (e.g. `tomllib`), the agents substituted an equivalent (`tomli`) to execute the logic and noted it.

---

## The two defects (fixed and re-verified)

**1. Month 5 · `lab-2-pytest-deep-dive-and-strict-types.md` · the monkeypatch test.**
`FakeDateTime.now()` called `datetime.datetime(...)` *after* `monkeypatch.setattr(datetime, "datetime", FakeDateTime)` had already replaced `datetime.datetime` with the no-arg `FakeDateTime`, causing `TypeError: FakeDateTime() takes no arguments` (an infinite self-reference).
**Fix applied:** capture the real class before patching — `real_datetime = datetime.datetime` — and return `real_datetime(2026, 5, 25, 9, 0)`. Re-verified: `1 passed`.

**2. Month 11 · `lab-2-safety-rails-...md` · `SafetySupervisor.call_model`.**
`guard_spend()` was called *outside* the `try` block, so the `BudgetExceeded` it raises bypassed the `except (BudgetExceeded,):` clause that logs `cap_hit` and fires the alert. The cap still blocked spending, but silently — failing the lab's own checkpoint (cap=$0.00 → log + alert).
**Fix applied:** moved `guard_spend(self.db, est_cost)` inside the `try` block. Re-verified: `model called? False | cap_hit logged? True | alert fired? True`.

---

## Highlights — what actually executed (not just compiled)

- **Months 1–2:** all `bin/` shell scripts ran with real/edge inputs; the full Git workflow (init→commits→branch→merge→stash) ran in a throwaway repo; every `jq` filter in the API Explorer's Notebook produced the claimed shapes; live unauthenticated GitHub/USGS/httpbin calls succeeded.
- **Months 3–4:** the `uv`-packaged Toolbelt CLIs (`csv2json`, `dirsize`, `note`) run as bare commands and pipe into `jq`; the hand-rolled retry/backoff/jitter was proven against a fake HTTP layer (429→200 retries, 503 exhausts with growing sleeps, 404 fast-fails, `Retry-After` honored); GitHub Pulse's pytest suite passed offline.
- **Months 5–6:** `mypy --strict` clean on the typed examples; the Refactor Crucible provider package hit 100% coverage on its core; the cost math matched to the cent ($0.0056 for a 2000/1000 Haiku-class call); the **from-scratch agent loop completed its real task** (ls→read→write SUMMARY.md→git commit) driven by the fake model, wrote a valid JSONL trace, and its jail rejected `../../etc/passwd` while the allowlist rejected `rm -rf /`.
- **Months 7–8:** the **fallback chain executed end-to-end** — primary blackholed at a dead port, cascaded to the local fake, finished the task for $0; fatal 4xx surfaced without retrying; the **attack-your-own-jail** suite rejected all 7 escapes (`..`, absolute, NUL, symlink); the FastAPI webhook accepted a valid HMAC signature and rejected tampered/replayed ones; the level-3 human gate blocked an irreversible action until approved.
- **Months 9–10:** the **Lead→Worker→Validator** harness ran with workers as real subprocesses in isolated working dirs, per-role model routing, Pydantic validation rejecting evidence-free findings, and trace→replay reproducing a run; the **six-stage factory** ran its real gates (`ruff`, `mypy`, `pytest --cov`) against a scaffolded app and proved the failed-gate→rebuild retry loop.
- **Months 11–12:** the SQLite job queue survived a mid-run crash and resumed exactly once; the circuit breaker, kill switch (sentinel-file and DB-flag), and retry→dead-letter-queue all fired correctly; the capstone's four seam tests (fallback, gate, regenerate, spend-cap) passed with stubs.

---

## Per-month detail

Full block-by-block tables are in this folder: `month-01.md` … `month-12.md`.

| Month | PASS | PASS-SYNTAX | FAIL |
|---|---:|---:|---:|
| 01 Command line & Git | ✓ | macOS/`gh` installs | 0 |
| 02 HTTP & JSON | ✓ | key-gated calls | 0 |
| 03 Python fluency | ✓ (all) | — | 0 |
| 04 Python & APIs | ✓ | token-gated GitHub | 0 |
| 05 SWE principles | ✓ | — | 1 → fixed |
| 06 First agent loop | ✓ (via fake model) | live Ollama/Anthropic | 0 |
| 07 Extensible software | ✓ | paid providers | 0 |
| 08 Agentic access | ✓ | Docker/Postgres role | 0 |
| 09 Agent harnesses | ✓ | live Ollama/uv setup | 0 |
| 10 Software factories | ✓ | `gh`/remote push | 0 |
| 11 Always-on agents | ✓ | launchd/cron/cloud | 1 → fixed |
| 12 Capstone | ✓ (seam tests) | deploy/14-day run | 0 |

**Bottom line:** the curriculum's code is sound. Every offline-deterministic block runs as written; the parts that need the real macOS + Ollama environment are syntactically valid and were exercised through stubs; the two genuine bugs are fixed and re-verified.
