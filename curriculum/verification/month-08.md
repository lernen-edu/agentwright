---
nav_exclude: true
---

# Month 8 — Agentic Access: Code Verification

**Environment:** Linux sandbox, Python 3.10 (labs target 3.12). The MCP SDK (`mcp 1.27.1`)
installed via pip and the full handshake ran. No Docker/Postgres/Colima available → container
shell and the Postgres read-only role are PASS-SYNTAX, with the surrounding Python executed and
the role-enforcement claim modeled with sqlite3 (which has no roles — claim noted PASS-SYNTAX).
FastAPI webhook tested with `fastapi.testclient.TestClient` as instructed.

## Summary counts

| Status | Count |
|---|---|
| PASS | 22 |
| PASS-SYNTAX | 6 |
| FAIL | 0 |
| N/A | — |

## Per-lab results

| Lab | Block / checkpoint | Status | Notes |
|---|---|---|---|
| 1 | `danger_demo.py` (`shell=True` hole) | PASS | `owned.txt` = `PWNED` — injection demonstrated |
| 1 | `safe_cli.run_cli` (argv/allowlist/cwd) | PASS | `rm` rejected before any process starts |
| 1 | `run_cli_json` (faded) | PASS | returns `{stdout,stderr,exit_code}` |
| 1 | `run_git` subcommand guard (independent) | PASS | `git push` raises; `git status` works |
| 1 | timeout + return-code | PASS | grep→`[exit 1]`; sleep→`TimeoutExpired` at 2s |
| 1 | `jail.safe_path` | PASS | escape raises, valid resolves |
| 1 | **`test_jail.py` (attack your own jail)** | PASS | `7 passed`: `..`, abs path, NUL, symlink→/etc all rejected |
| 1 | `CliTool`/`ReadFileTool` (M7 interface) | PASS-SYNTAX | needs full M7 agent registry; shape valid |
| 2 | `mcp_server.py` (FastMCP, stdio) | PASS | FastMCP imports & runs on 3.10 |
| 2 | `mcp_client.py` handshake/list/call | PASS | real round-trip; `safe-hands-in-the-wild` |
| 2 | `mcp_tool.py` `McpSlugifyTool` (faded) | PASS | `hello-agentic-world` through interface |
| 2 | `McpWordCountTool` (faded task) | PASS | returns count dict via MCP |
| 2 | `mcp dev` inspector | PASS-SYNTAX | needs Node web UI (optional per lab) |
| 3 | `receiver.py` HMAC verify + replay | PASS | via TestClient (below) |
| 3 | `sender.py` signing | PASS | sign logic matches receiver (used in tests) |
| 3 | unsigned/tampered/stale → 401 | PASS | all three rejected |
| 3 | valid signature → 200 accepted | PASS | TestClient confirms |
| 3 | replay → `duplicate-ignored` | PASS | second identical id ignored |
| 3 | `egress.fetch_url` allowlist | PASS | non-allowlisted host raises (deny path) |
| 3 | `fetch_url` allow path (github) | PASS-SYNTAX | needs network egress |
| 3 | `redact.py` | PASS | `Authorization: Bearer sk-...` → `***redacted***` |
| 4 | `toolkit.py` danger-gate `dispatch` | PASS | level-1 runs unprompted |
| 4 | **human gate (level-3)** | PASS | stub `no`→`PermissionError`, file survives; `yes`→deletes |
| 4 | `ContainerShellTool` | PASS-SYNTAX | allowlist guard executes & rejects; `docker run` not runnable |
| 4 | `QueryDbTool` SELECT path | PASS | modeled w/ sqlite3 (`[(1,'Ada'),(2,'Linus')]`) |
| 4 | read-only role refuses `DROP` | PASS-SYNTAX | no Postgres; role enforcement is DB-only |
| 4 | psycopg `QueryDbTool` source | PASS-SYNTAX | psycopg not installed offline; syntax OK |
| 4 | `app.py` webhook→agent | PASS | signed→200 `{status:done}`, unsigned→401 |
| RM | README conceptual/anti-pattern snippets | PASS-SYNTAX | illustrative; patterns proven in labs |

## Load-bearing security checkpoints (executed)

- **Jail hardening:** `test_jail.py` all 7 green — the symlink-to-`/etc` case (the important
  one) is rejected because `resolve()` follows the link and `is_relative_to(ROOT)` then fails.
  `..`, absolute paths, and NUL bytes all bounce.
- **Subprocess safety:** argv lists (no shell), allowlist default-deny, timeout fires,
  non-zero return codes reported honestly. The `shell=True` hole was demonstrated first.
- **MCP:** real SDK; `initialize → tools/list → tools/call` all completed over stdio; adapter
  presents the MCP tool behind the Month 7 interface with no special-casing.
- **Webhook (must-pass):** HMAC-SHA256 over `f"{ts}." + raw_body`, constant-time compare,
  300s skew check, in-memory seen-IDs. Valid accepted, **tampered/unsigned/stale rejected
  401, replay → duplicate-ignored.** Verified with TestClient.
- **Danger gate:** logic lives in `dispatch`; level-3 `delete_path` blocks until `yes`.
  Declined raises `PermissionError` and the file survives; approved deletes. Stubbed stdin.
- **Egress allowlist:** non-allowlisted host denied before the request is sent.

## FAIL details & fixes

None. No broken blocks in Month 8.

## Adaptations made (per harness rules)

- `mcp_tool.py`/`mcp_client.py` use `command="python3"` instead of `"uv"` for the stdio
  subprocess (no `uv`-managed venv in the sandbox). Protocol behavior is identical.
- `query_db` modeled with sqlite3; the *read-only role enforcement by Postgres* claim is
  PASS-SYNTAX (sqlite enforces no roles), but the SELECT/connection Python executed.
- Container shell (`docker run --rm --network none --user 1000:1000 …`) is PASS-SYNTAX:
  the allowlist guard before the subprocess was executed and correctly rejects non-allowlisted
  argv; the actual container launch is not runnable here.
