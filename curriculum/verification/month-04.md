---
nav_exclude: true
---

# Month 4 — Python and APIs · Code Verification

**Summary: PASS 26 · PASS-SYNTAX 6 · FAIL 0 · N/A 10** (executed on Linux sandbox). Live network reached GitHub public API + httpbin successfully. Retry/backoff/jitter and pagination logic were executed against a fake HTTP layer (429→200, 503-exhaust, 404 fast-fail, Retry-After, multi-page Link). Token-only paths → PASS-SYNTAX.

## Lab 1 — Requests: GET, POST, Headers, Auth

| Block | Status |
|---|---|
| Setup `uv init`/`uv add` | PASS (requests, dotenv, httpx installed) |
| Step 1 `first_get.py` (Stage 1 worked) | PASS (`octocat has 8 public repos`, 200) |
| Step 2 Response fields REPL | PASS (status/ok/headers/url/json) |
| Step 3 `search.py` faded (params+UA, status check) | PASS (5 starred Python CLI repos) |
| Step 3 `# TODO` skeleton (unfilled) | PASS-SYNTAX (compiles; needs params filled) |
| Step 3 `<details>` solution | PASS (used to fill TODOs) |
| Stage 3 `repos.py` independent (reference) | PASS (octocat repos+language live) |
| Step 5 `post_demo.py` httpbin POST | PASS (Content-Type json, body echoed) |
| Step 6 create token (browser) | N/A |
| Step 7 HTTPie token verify | N/A (HTTPie/macOS) |
| Step 8 `.env` + gitignore | N/A (shell) |
| Step 9 `whoami.py` auth call | PASS-SYNTAX (compiles; no-token guard exits 1 correctly) |
| Step 10 leak-check grep | PASS (`CLEAN: no token in history`) |
| Step 11 httpx peek | PASS (`200 octocat`) |
| Mermaid / DoD self-verify | N/A / covered |

## Lab 2 — Resilience: Timeouts, Retry, Rate Limits

| Block | Status |
|---|---|
| Step 1 `timeout_demo.py` (live delay/5, t=2) | PASS (ReadTimeout raised) |
| Step 2 catch both families (live) | PASS (transport msg, http 503, ok:200) |
| Step 3 `http.py` `get_with_retry` import | PASS (`ok`; package layout) |
| Stage 2 faded `_backoff` (filled) | PASS (cap8: …,8.7,9.0 flattened) |
| Stage 2 `_backoff` `# TODO` skeleton | PASS-SYNTAX (compiles with `...`) |
| Step 4 `retry_demo.py` 503 backoff | PASS (3 incr. jittered sleeps 1.4/2.4/4.7, final 503, 4 calls) |
| Step 4 backoff math (fake) | PASS (`1,2,4,8,16`+jitter) |
| 429→200 retry-then-succeed (fake) | PASS (retries once, returns 200) |
| Retry-After honored (fake) | PASS (slept exactly 7.0s) |
| transport-error retry-then-raise (fake) | PASS (2 sleeps, raised on last) |
| Step 5 `404` fast-fail (fake) | PASS (1 call, no sleep) |
| Step 6 `ratelimit.py` | PASS-SYNTAX (compiles; needs token) |
| Step 7 `get_all_pages` paginate (fake 3-page) | PASS (8 items across 3 pages via `next`) |
| Step 8 `max_pages=1` cap (fake) | PASS (stops at 1 page despite next link) |
| Mermaid diagrams | N/A (3) |

## Lab 3 — GitHub Pulse (Milestone)

| Block | Status |
|---|---|
| Step 1/2 layout + `http.py` import | PASS |
| Step 3 `client.py` `gather` | PASS-SYNTAX (compiles; live+token) |
| Step 4 `report.py` pure fns (Stage 1) | PASS (`[('Python',2),('Go',1)]`) |
| Step 5 `render_markdown` faded (filled Languages) | PASS (empty + realistic samples render) |
| Step 6 `__init__.py` `main` + `--help` | PASS (help offline; gather=PASS-SYNTAX) |
| Step 7 pytest suite (3 + my 4th starred) | PASS (`4 passed`) |
| Step 8 monkeypatch fake-network test | PASS (`5 passed`, 404 in 1 call) |
| Step 9/10 commit/push/`uv tool install` | N/A (network) / PASS-SYNTAX |

## FAIL details & fixes

None. No broken blocks.

### Minor notes (not failures)
- Module is named `http.py`; safe as the package submodule `github_pulse.http` / `resilience.http`. Only a loose top-level file named `http.py` would shadow stdlib — the labs never place it there, so no issue.
- Token-dependent live calls (`whoami`, `ratelimit`, `gather`, installed `github-pulse octocat`) classified PASS-SYNTAX: code compiles and offline guards/logic execute correctly; only the authenticated GitHub round-trip is unrunnable without a secret.
