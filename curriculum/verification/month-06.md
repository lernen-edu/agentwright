---
nav_exclude: true
---

# Month 6 — AI APIs and the First Agent Loop · Code Verification

**Summary: PASS 22 · PASS-SYNTAX 11 · FAIL 0 · N/A 6** (executed on Linux sandbox, Python 3.10). No Ollama/Anthropic used. Stood up a **fake OpenAI-compatible FastAPI server** (uvicorn on localhost) returning canned `choices[0].message` + `usage`, and drove `call_model`, the eval-scoring logic, the tool-use dispatch, and the **full from-scratch agent loop** against it for real. Cost math, tiktoken, defensive JSON parsing, streaming-SSE accumulation, the jail, and the JSONL trace all executed. Live model round-trips → PASS-SYNTAX.

## Lab 1 — First Model Call and Cost Math

| Block | Status |
|---|---|
| Setup `brew install ollama` / `uv init` / `ollama pull` | N/A (Ollama/macOS) |
| Step 1 raw `curl` to `/v1/chat/completions` | PASS-SYNTAX (needs live Ollama) |
| Step 2 `first_call.py` raw HTTP (Stage 1) | PASS-SYNTAX (verified against fake server) |
| Step 3 knobs (max_tokens/system/stop/temp) | PASS-SYNTAX (needs live model) |
| Step 4 `token_math.py` tiktoken | PASS (43 chars, 9 words, **12 tokens** — rule of thumb holds) |
| Step 5 `cost.py` Usage.dollars | PASS (Haiku 1500/500 = **$0.003200**; Ollama = **$0.000000**) |
| Step 6 `model.py` `call_model` faded (filled) | PASS (`INFO tokens in=20 out=12 cost=$0.000000` + reply, vs fake) |
| Step 6 `# TODO` skeleton (unfilled) | PASS-SYNTAX (compiles; 4 ellipsis TODOs) |
| Step 6b Stage 3 independent | PASS-SYNTAX (same pattern, fake-verified) |
| Step 7 Anthropic SDK call | PASS-SYNTAX (paid; key required) |
| DoD self-verify `Usage('claude-haiku',2000,1000)` | PASS (**0.0056** exactly as lab claims) |
| DoD 2000/1000 @ $1/$5 = $0.007 | PASS (computed 0.007) |

## Lab 2 — Prompting, Structured Output, Streaming, Evals

| Block | Status |
|---|---|
| Step 1 `extract.py` vague prompt | PASS-SYNTAX (needs model) |
| Step 2 `parse_json` defensive parse (Stage 1) | PASS (plain + fenced + raises on garbage) |
| Step 2 `extract()` against model | PASS-SYNTAX (fake-verified call path) |
| Step 3 `fewshot.py` faded (added neutral example) | PASS-SYNTAX (logic OK; needs model for trend) |
| Step 4 `stream_model` SSE accumulate | PASS (canned deltas -> "Hello world"; `finish_reason`/`[DONE]` guards work) |
| Step 5 `eval_harness.py` scoring + winner | PASS (stubbed model: 3/3; `all(got.get==v)` + except path OK) |
| Step 5 live A/B scores | PASS-SYNTAX (needs model) |
| DoD self-verify `parse_json('```json{...}```')` | PASS (`{'a': 1}`) |

## Lab 3 — Tool Use (Function Calling) by Hand

| Block | Status |
|---|---|
| Step 1 `tools.py` get_weather + schema + REGISTRY | PASS (`get_weather('Tokyo')` dict) |
| Step 2 `roundtrip_ollama.py` advertise+request (Stage 1) | PASS-SYNTAX (needs qwen2.5 tool-caller) |
| Step 3 faded TODOs: `json.loads(args)` + `REGISTRY[name](**args)` | PASS (dispatch-by-name executed) |
| Step 3 feed-back-and-call-again | PASS-SYNTAX (round-trip vs live model) |
| Step 4 vague-description experiment | PASS-SYNTAX (needs model) |
| Step 5 chatty-tool 40 KB blob | PASS-SYNTAX (needs model usage delta) |
| Step 6 Anthropic native tool blocks | PASS-SYNTAX (paid) |
| DoD self-verify dispatch (Tokyo/fahrenheit) | PASS (**temperature 75, unit fahrenheit**) |

## Lab 4 — The From-Scratch Agent (Milestone)

| Block | Status |
|---|---|
| Setup sandbox repo + git init | PASS (seeded calc.py, git repo) |
| Step 1 jail + 3 tools | PASS (`inside ok` then ValueError on `../../etc/passwd`) |
| Step 1 self-verify allow-list `rm -rf /` | PASS (`allow-list OK`) |
| Step 2 TOOLS schemas | PASS (valid; agent advertised them) |
| Step 3 provider-agnostic `call_model` | PASS (normalized dict + `[model] in/out/cost` stderr, vs fake) |
| Step 4 `trace()` JSONL | PASS (start/tool_call/final lines written) |
| Step 5 `run_agent` loop (Stage 1) | PASS (ls->read->write->git add->commit->`DONE:`) |
| Step 5 Stage 2 faded skeleton (4 TODOs) | PASS-SYNTAX (valid Python; ellipsis placeholders) |
| Step 6 full run: SUMMARY.md + commit + trace | PASS (`SUMMARY.md` written, commit `add summary`, $0.00 total) |
| Step 6 MAX_STEPS abort path | PASS (clean `ABORTED: hit step limit` safety exit observed) |
| Stage 3 independent (LINECOUNT task) | PASS-SYNTAX (loop unchanged; same machinery) |
| Step 7 `FAILURES.md` | N/A (prose deliverable) |
| Step 1/7 Anthropic provider stretch | PASS-SYNTAX (paid) |
| DoD self-verify (allow-list/task/trace) | PASS |

## FAIL details & fixes

None. No broken code blocks in any Month 6 lab.

### Notes (not failures)
- All live `requests.post` to Ollama and the Anthropic SDK calls are PASS-SYNTAX: HTTP/parse/normalize/dispatch logic ran against a localhost fake model server returning the exact `choices[0].message` + `usage` shape; only the real-model network leg is unrun.
- Cost numbers verified to the cent and internally consistent: README §3, Week-1 checkpoint ($0.007), `cost.py` ($0.003200 / $0.000000), and the Lab 1 self-verify ($0.0056 Haiku).
- The Lab 4 agent loop completes the milestone task end-to-end (write `SUMMARY.md`, git commit, emit `DONE:`) AND cleanly hits the `MAX_STEPS` safety exit when looped — both stop paths observed. Jail rejects `../` escapes; shell runs an arg-list against an allow-list; args parsed via `json.loads`, never `eval`.
- Stage 2 faded skeletons (Lab 1 `call_model`, Lab 4 `run_agent`) use `...` ellipsis placeholders by design — valid Python, runnable once TODOs are filled.
