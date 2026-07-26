---
nav_exclude: true
---

# Month 02 — HTTP & JSON: Code Verification

**Summary:** PASS 31 · PASS-SYNTAX 8 · FAIL 0 · N/A 4

Method: `jq` filters executed against the lab's `sample.json` and reconstructed faithful samples; `curl` run live against public no-auth endpoints (api.github.com, httpbin.org, USGS GeoJSON, open-meteo) — all reachable. HTTPie (`http`) is absent in the sandbox → its command lines are PASS-SYNTAX (shell-tokenization valid). OpenWeather and authenticated GitHub steps need keys → PASS-SYNTAX with jq shape verified on reconstructed responses. Heredocs executed.

## Per-lab results

| Lab | Block | Status |
|---|---|---|
| README | Mermaid / anatomy diagrams | N/A |
| Lab 1 | setup `brew install httpie jq` | PASS-SYNTAX (macOS) |
| Lab 1 | S1 `curl /zen` (body) | PASS |
| Lab 1 | S1 `curl -i` status+headers | PASS |
| Lab 1 | S1 `curl -v` full exchange | PASS |
| Lab 1 | S2 `httpbin/get \| jq` | PASS |
| Lab 1 | S2 query string `.args` | PASS |
| Lab 1 | S2 custom header `-H` | PASS |
| Lab 1 | S3 POST JSON body `.json` | PASS |
| Lab 1 | S3 redirect `-L` (302→200) | PASS |
| Lab 1 | S3 status 404/500/429 | PASS |
| Lab 1 | S3 HTTPie GET/POST/header (3) | PASS-SYNTAX |
| Lab 1 | step 11 curl+jq vs HTTPie | PASS (curl) / PASS-SYNTAX (http) |
| Lab 1 | self-verify `.public_repos` | PASS |
| Lab 2 | setup heredoc `sample.json` | PASS |
| Lab 2 | S1 #1 identity `.` | PASS |
| Lab 2 | S1 #2 key access | PASS |
| Lab 2 | S1 #3 nested `.profile.*` | PASS |
| Lab 2 | S1 #4 array index `[0]`/`[-1]` | PASS |
| Lab 2 | S1 #5 iterate `[]` | PASS |
| Lab 2 | S1 #6 `length` (3) | PASS |
| Lab 2 | S2 #7 `select` (filled cond) | PASS |
| Lab 2 | S2 #8 reshape `{}` (shorthand) | PASS |
| Lab 2 | S2 #9 `map`/`add` (177) | PASS |
| Lab 2 | S2 #10 `sort_by`/`reverse`/`.[0]` | PASS |
| Lab 2 | S3 #11 live GitHub repos | PASS |
| Lab 2 | S3 #12 USGS slice + `select>=2.0` | PASS |
| Lab 2 | self-verify `\| jq length` | PASS |
| Lab 2 | stretch `group_by(.language)` | PASS |
| Lab 3 | setup `.gitignore`/`.env.example` heredoc | PASS |
| Lab 3 | S1 GitHub docs (6 answers) | N/A |
| Lab 3 | S1 #2 token-load `grep\|cut` len | PASS (fake token) |
| Lab 3 | S1 #3 ratelimit compare | PASS (unauth=60) / PASS-SYNTAX (auth) |
| Lab 3 | S1 #4 three endpoints | PASS (repo/search) / PASS-SYNTAX (`/user` auth) |
| Lab 3 | S1 #5 pagination Link header | PASS-SYNTAX (needs token; shape verified) |
| Lab 3 | S2 #6 USGS metadata | PASS |
| Lab 3 | S2 #7 three USGS views | PASS |
| Lab 3 | S3 #9-10 OpenWeather + 401 body | PASS-SYNTAX (jq shape PASS; key needed) |
| Lab 3 | S3 Open-Meteo fallback | PASS (live, no key) |
| Lab 3 | #12 `NOTEBOOK.md` heredoc | PASS |
| Lab 3 | #13 `gh repo create`; secret-scan grep | PASS-SYNTAX (gh) / PASS (grep CLEAN) |
| Lab 3 | self-verify clone + USGS | PASS-SYNTAX (gh clone) |

## FAIL details & fixes

None. Every jq filter produces exactly the shape its checkpoint claims, and every live curl returned the documented status/body.

## Notes (non-blocking)

- httpbin returns POST/JSON object keys alphabetically sorted; Lab 1 step 7 checkpoint shows them in input order. Same fields/values — cosmetic only.
- All "If not" recovery hints match real failure modes observed (e.g., unquoted URL → zsh glob; missing `User-Agent` → GitHub 403; HTML page → `jq` parse error).
- Stage 1/2/3 release is consistent: the `select`/reshape worked lines and their faded TODO twins resolve to the same filter family; the worked OpenWeather filter and faded coordinate variant share one shape.
