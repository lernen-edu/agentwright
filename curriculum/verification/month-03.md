---
nav_exclude: true
---

# Month 3 — Python Fluency · Code Verification

**Summary: PASS 41 · PASS-SYNTAX 0 · FAIL 0 · N/A 9** (executed on Linux sandbox, Python 3.10 + uv-installed 3.12). Every runnable block actually ran; checkpoints matched exactly. This month is entirely offline and all-PASS.

## Lab 1 — Language Core and Data Structures

| Block | Status |
|---|---|
| Setup `uv init` (zsh, macOS brew) | N/A (brew) — uv project built & runs |
| Step 1 REPL `2+2`, str+str, `"11"+1` TypeError, `len` | PASS (all 4 outputs incl. TypeError) |
| Step 2 `main.py` f-strings (computes 12) | PASS |
| Step 3 types/conversion (`Total: 59.97`, str/float) | PASS |
| Step 4 `grade()` branching loop | PASS (`50 -> needs work`) |
| Step 5 `structures.py` list/dict/set/tuple | PASS (set collapses to 3) |
| Step 6 `report.py` list-of-dicts loop | PASS |
| Step 7 Stage 1 worked (names/roles comprehensions) | PASS |
| Step 7 Stage 2 faded (filled: engineers, ages dict) | PASS (`['Ada','Grace']`, ages dict) |
| Step 7 Stage 3 independent (average age) | PASS (`average age: 39.0`) |
| Faded skeleton with `____` | PASS-noted: compiles (`____` is valid identifier) |
| Step 8 git commit | N/A (illustrative git) |
| Mermaid pipeline diagram | N/A |

## Lab 2 — Files, JSON, CSV, Errors

| Block | Status |
|---|---|
| Setup / sample CSV | PASS (CSV created) |
| Step 1 `files.py` write/read/splitlines | PASS |
| Step 2 append x3 (count climbs 1,2,3) | PASS |
| Step 3 `convert.py` CSV→list-of-dicts | PASS (`age` is str `'36'`) |
| Step 4 JSON out + `jq '.[].name'` | PASS (3 names) |
| Step 5 JSON round-trip | PASS (`first name is Ada`) |
| Step 6 Stage 1 worked traceback | PASS (`FileNotFoundError ... 'nope.csv'`) |
| Step 6 Stage 2 faded (filled stderr/exit 1) | PASS (`exit code: 1`, no traceback) |
| Step 6 Stage 3 independent `to_int` | PASS (36, warn forty→0, 5) |
| Mermaid round-trip / decision diagrams | N/A |

## Lab 3 — Toolbelt CLI (Milestone)

| Block | Status |
|---|---|
| Step 1 `uv init --package` scaffold | PASS (src/toolbelt layout, 3.12) |
| Step 2 `csv2json.py` worked + run as module | PASS |
| `csv2json | jq` + error path | PASS (`exit: 1` on missing) |
| Step 3 `dirsize.py` faded (filled `?`/`.`/int) | PASS (Total + largest, exit 1 on bad dir) |
| Step 4 `note.py` add/list subcommands | PASS (timestamped, persists) |
| Step 5 `[project.scripts]` + bare commands | PASS (csv2json/dirsize/note all run via `uv run`) |
| `--help` screens | PASS |
| Step 6 README (four-backtick fences) | N/A (doc) |
| Step 7 commit + `gh repo create` | N/A (gh/network) |
| One-shot self-verify (`ALL TOOLS OK`) | PASS |

## FAIL details & fixes

None. All executable blocks passed and matched their stated checkpoints exactly.

### Notes
- `uv` (0.11.2) and `jq` available in sandbox; uv installed CPython 3.12.13 per the labs.
- All three Toolbelt commands installed via `[project.scripts]` and ran as bare commands, piped into `jq`, and exited non-zero on error paths — milestone DoD met.
- `brew`/`gh`/`code .` lines are macOS tooling (N/A here) but are not code logic; the surrounding Python all runs.
