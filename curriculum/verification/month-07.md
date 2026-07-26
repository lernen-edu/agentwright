---
nav_exclude: true
---

# Month 7 — Extensible Software: Code Verification

**Environment:** Linux sandbox, Python 3.10 (labs target 3.12). `tomllib` substituted with
`tomli` (installed) where a config block imports it — logic executed identically, noted below.
All model/network calls stubbed with a localhost fake "Ollama" (OpenAI-compatible) HTTP server;
no paid APIs touched. Each lab built in a `/tmp` scratch package and executed.

## Summary counts

| Status | Count |
|---|---|
| PASS | 21 |
| PASS-SYNTAX | 4 |
| FAIL | 0 |
| N/A | — |

PASS-SYNTAX items are blocks requiring a live paid provider (OpenAI/Anthropic auth) or the
README's conceptual/anti-pattern excerpts. Everything load-bearing executed.

## Per-lab results

| Lab | Block / checkpoint | Status | Notes |
|---|---|---|---|
| 1 | `llm/base.py` Protocol + `ModelReply` | PASS | imports clean; `runtime_checkable` |
| 1 | `OllamaClient.complete` (live call) | PASS | ran vs localhost fake; `'pong'`, tokens 11/1 |
| 1 | `OpenAIClient` (faded fill-in) | PASS | imports; live call is paid → not invoked |
| 1 | `AnthropicClient` (independent) | PASS | normalizes content blocks to one `ModelReply` |
| 1 | `tests/test_conformance.py` | PASS | `2 passed` — structural conformance proven |
| 1 | `demo_inject.py` + no-branch grep | PASS | `[ollama]` line; grep confirms no provider branch |
| 2 | `llm/registry.py` `@register` + `make_client` | PASS | branch-free lookup; unknown raises clearly |
| 2 | decorate 3 providers + OpenRouter (4th) | PASS | `['anthropic','ollama','openai','openrouter']` |
| 2 | `config.toml` + `config.py` load/validate | PASS | tomli sub; typo `provder` fails loudly at startup |
| 2 | `build.py` config→registry selection | PASS | `primary provider from config: ollama` |
| 2 | `strategies.py` Strategy registry | PASS | plain/verbose selected by config |
| 2 | `tools.py` `@tool` registry (3 tools) | PASS | `list_files` self-registers, schemas=3 |
| 2 | `prompts/` v1+v2 + `load_prompt` | PASS | active version v2 loaded from config |
| 3 | `llm/fallback.py` `FallbackClient` | PASS | error classification correct |
| 3 | `tests/test_fallback.py` | PASS | `2 passed` — cascade + fatal-400-surfaces |
| 3 | `build_chain` from config | PASS | `['ollama','openrouter','ollama']` |
| 3 | `agent.py` provider-agnostic + grep | PASS | grep: "loop is provider-agnostic OK" |
| 3 | `run_shell` tool (allowlist/argv/timeout) | PASS | executed in failover demo |
| 3 | **Failover demo (the milestone)** | PASS | see below |
| 1–3 | OpenAI/Anthropic *live* calls | PASS-SYNTAX | require paid keys; offline |
| RM | README conceptual/anti-pattern snippets | PASS-SYNTAX | illustrative; same patterns proven in labs |

## The fallback chain (critical, executed)

Configured primary `openai` at `base_url=http://localhost:1` (refuses immediately →
`ConnectionError`), fallback = local fake "Ollama". Ran the full agent task (list files →
read each → write SUMMARY.md → `git add` → `git commit` → DONE). Observed:

- `agent.fallback WARNING provider openai unavailable (ConnectionError); falling over`
  on **every** step, then `recovered: ollama served after failover`.
- `[model:ollama]` drove all 6 tool steps to `DONE: summarized 2 files and committed.`
- `trace.jsonl` `served_by` values = `ollama` only; `sandbox/SUMMARY.md` written; new git
  commit `summary` present. Cost: $0 (local survivor).
- Fatal-error path verified separately: a 400 `HTTPError` is **re-raised, not cascaded**
  (`test_fatal_error_surfaces_not_cascades` passes). 429 and connection/timeout cascade.

This is exactly the checkpoint claim: blackholed primary → cascade to free local Ollama →
task finishes → failover provable from the trace.

## FAIL details & fixes

None. No broken blocks in Month 7.

## Notes for maintainers

- `config.py` block imports `tomllib` (3.11+). On the 3.10 sandbox I used the documented
  substitution `import tomli as tomllib`. The lab itself instructs running via `uv` with
  Python 3.12, where `tomllib` is stdlib — so this is an environment artifact, not a defect.
- All live OpenAI/Anthropic/OpenRouter calls are correctly gated as the optional paid path;
  the free Ollama path and all structural/registry/fallback logic run at $0.
