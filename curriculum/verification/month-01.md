---
nav_exclude: true
---

# Month 01 — Command Line & Git: Code Verification

**Summary:** PASS 24 · PASS-SYNTAX 9 · FAIL 0 · N/A 7

Method: shell snippets run `bash -n`; safe scripts executed in a `/tmp` sandbox with `HOME` redirected; the full Git workflow (init→add→commit→branch→merge→stash→restore) run in a throwaway repo. macOS-only (Homebrew, GUI editors) and `gh`-CLI/GitHub-auth steps are PASS-SYNTAX or N/A. Interactive vim/nano keystrokes are N/A.

## Per-lab results

| Lab | Block | Status |
|---|---|---|
| README | Mermaid diagrams (5) | N/A |
| README | `echo "a\nb\nc" \| grep b \| wc -l` (=1, zsh) | PASS |
| README | KC#3 `echo a>f; echo b>>f; cat` (a/b) | PASS |
| Lab 1 | Homebrew / `brew install` (macOS) | PASS-SYNTAX |
| Lab 1 | `pwd`/`ls`/`man` | PASS |
| Lab 1 | Stage 1 worked `cd~/mkdir/cd` | PASS |
| Lab 1 | Stage 2 faded nav (`cd ..`, `/usr`) | PASS |
| Lab 1 | Stage 3 independent nav | PASS |
| Lab 1 | step 6 touch/echo/cat/cp/mv | PASS |
| Lab 1 | step 7 `rm`, `rm -r` | PASS |
| Lab 1 | step 8 `ls \| less` (pager) | PASS-SYNTAX |
| Lab 1 | self-verify one-liner | PASS-SYNTAX (needs brew/gh) |
| Lab 2 | `bin/hello` + chmod | PASS |
| Lab 2 | PATH export in `~/.zshrc` | PASS-SYNTAX |
| Lab 2 | Stage 1 worked `mkproject` (+no-arg guard) | PASS |
| Lab 2 | Stage 2 faded `todo` (filled solution) | PASS |
| Lab 2 | Stage 3 independent `extract` (+ ref in step 6) | PASS |
| Lab 2 | `ccd` function (guard + real clone + cd) | PASS |
| Lab 2 | step 7 pipe + loop over `bin/` | PASS |
| Lab 2 | self-verify one-liner | PASS-SYNTAX |
| Lab 3 | `git config` / `gh auth login` | PASS-SYNTAX (gh auth) |
| Lab 3 | `git init` + `git status` | PASS |
| Lab 3 | `.gitignore` | PASS |
| Lab 3 | Stage 1 worked commit | PASS |
| Lab 3 | Stage 2 faded per-script commits (5) | PASS |
| Lab 3 | step 4 `log --graph`/`diff`/`show` | PASS |
| Lab 3 | Stage 3 branch `gss` + merge | PASS |
| Lab 3 | step 6 `stash`/`pop`/`restore` | PASS |
| Lab 3 | step 7 README commit | PASS |
| Lab 3 | step 8 `gh repo create` | PASS-SYNTAX |
| Lab 3 | step 9 issue/PR/`serve` (gh) | PASS-SYNTAX (serve syntax PASS) |
| Lab 3 | self-verify (`gh repo view`) | PASS-SYNTAX |
| Lab 4 | `vimtutor` / vim keystrokes / nano | N/A (interactive) |
| Lab 4 | `:%s/cat/dog/g`, mode diagram | N/A |
| Lab 4 | stretch `~/.vimrc` snippet | PASS (loads in vim) |

## FAIL details & fixes

None. All executable code behaves as the checkpoints claim.

## Notes (non-blocking)

- README Week-2 checkpoint and KC#3 rely on zsh `echo` interpreting `\n`; correct on the macOS/zsh target. In bash without `-e` the `\n` would be literal — fine, since the course standardizes interactive use on zsh.
- Lab 2 has cosmetic numbering slips: heading "4. Script 3 — `ccd`" follows Stage 3 (which built `extract`, also called "Script 3"), and step 5 is skipped (jumps to "6. Reference"). No effect on runnable code. Suggested fix: renumber `ccd` as Script 4 and the reference as step 5.
- Stage 1/2/3 progressions are internally consistent: the `todo` faded solution and the `extract` worked reference both run and match the worked `mkproject` pattern.
