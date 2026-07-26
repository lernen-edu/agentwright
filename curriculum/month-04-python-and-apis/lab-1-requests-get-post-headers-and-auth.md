---
title: "Lab 1 — Requests Basics: GET, POST, Headers, and Auth"
parent: "Month 04 — Python Meets the Network: APIs as a First-Class Citizen"
grand_parent: Curriculum
nav_order: 1
---

# Lab 1 — Requests Basics: GET, POST, Headers, and Auth

**Time: ~3.5 hrs · Difficulty: Intro / Core · Builds on: Month 2 (HTTP/JSON), Month 3 (Python + uv)**

## Objective

Make your first HTTP calls from inside Python. You will replace the `curl`/HTTPie habits of Month 2 with the `requests` library: `GET` with query parameters and headers, read a `Response` object instead of a screen of text, send a `POST` with a JSON body to a real public API, and then authenticate to the GitHub API with a token you load from a `.env` file — never from code, never committed to Git. By the end you will have a small project that calls three real APIs and proves your secret is safe. This is the request/response cycle that every model API later in the course is built on.

## Setup

```zsh
brew install uv                         # if not already installed (Month 3)
uv python install 3.12                  # if not already installed
mkdir -p ~/agentic/month-04 && cd ~/agentic/month-04
uv init api-basics --package --python 3.12
cd api-basics
uv add requests python-dotenv
uv add httpx                            # we only peek at this at the end
git init
```

You will also use HTTPie from Month 2 (`brew install httpie`) to cross-check what your Python is doing, and you need a free GitHub account.

## Background

**Recall first (from memory):** In Month 2, what `curl` flag set a request header, and what flag sent a body? In Month 3, what does `data.get("login")` return if the key is missing? Answer before reading on — these map directly onto what you are about to do in Python.

In Month 2 you ran `curl https://api.github.com/users/octocat` and read the JSON on your screen. `requests` does the same call but hands you a `Response` *object* your program can branch on. The three things you attach to a request — query `params=`, `headers=`, and a JSON body via `json=` — map exactly onto the `curl` flags you already know (`?a=b`, `-H`, `-d`). The one genuinely new discipline is that *your code* must check the result: `requests` returns happily for a `404` or `500`, so a `200` is something you verify, not assume. The auth half of the lab installs the most important security habit of the whole course: secrets live in `.env`, `.env` is git-ignored, and you verify it before you ever push.

## Steps

Steps 1–3 teach the one genuinely new skill of this lab — *make a request and deliberately check the result* — as a gradual release: a fully worked GET, then a partly-faded version you complete, then an independent GET you write from scratch.

### Stage 1 — Worked example (I do)

### 1. Your first GET

Study this complete example, then run it. Every line is explained; you are not inventing anything yet. Create `src/api_basics/first_get.py`:

```python
import requests


def main():
    resp = requests.get("https://api.github.com/users/octocat", timeout=10)
    print("status:", resp.status_code)
    print("type:  ", resp.headers.get("Content-Type"))
    data = resp.json()                       # JSON body -> Python dict
    print(f"{data['login']} has {data['public_repos']} public repos")


if __name__ == "__main__":
    main()
```

```zsh
uv run python src/api_basics/first_get.py
```

**Checkpoint:** you see `status: 200`, a `Content-Type` of `application/json; charset=utf-8`, and a line like `octocat has 8 public repos`. You just made an HTTP call from Python and parsed JSON without `jq`.
**If not:** a `ModuleNotFoundError` means you ran bare `python` instead of `uv run python` — re-run with `uv run`. A hang means you dropped `timeout=`. A `KeyError` means the field name is wrong; print `data` to see the real shape.

### 2. Inspect the whole Response

The status code is one field on a rich object. Open a REPL and explore:

```zsh
uv run python
```

```python
import requests
r = requests.get("https://api.github.com/users/octocat", timeout=10)
r.status_code          # 200  (an int)
r.ok                   # True (any 2xx/3xx)
r.headers["Date"]      # response headers, case-insensitive dict
r.url                  # the final URL actually requested
r.text[:80]            # the raw body as a string
type(r.json())         # <class 'dict'> — parsed body
```

**Checkpoint:** you can name what `.status_code`, `.ok`, `.headers`, `.text`, and `.json()` each give you. Note that `.json()` would *raise* if the body were not JSON — you will guard against that next.
**If not:** re-run each line in the REPL one at a time and read the type it returns; if the REPL exited, restart it with `uv run python` and re-import.

### Stage 2 — Faded practice (we do)

### 3. Send query parameters and a custom header

This is the same request-and-check skill, but you complete the mechanical parts. Never hand-build a URL with `?` and `&` again — pass a dict to `params=`. GitHub also requires a `User-Agent`. The skeleton below has two `# TODO` lines for you to fill; the structure and the status check are given. Create `src/api_basics/search.py`:

```python
import sys
import requests


def search_repos(query, per_page=5):
    resp = requests.get(
        "https://api.github.com/search/repositories",
        # TODO 1: pass params= with q, per_page, and sort="stars"
        headers={
            "Accept": "application/vnd.github+json",
            "User-Agent": "api-basics-lab",
        },
        timeout=10,
    )
    # TODO 2: if the status is not 200, print to stderr and raise SystemExit(1)
    return resp.json()["items"]


def main():
    for repo in search_repos("language:python topic:cli"):
        print(f"{repo['stargazers_count']:>7}  {repo['full_name']}")


if __name__ == "__main__":
    main()
```

Expected behavior: five most-starred Python CLI repos printed with star counts, and a clean non-zero exit (with a `stderr` message) if the status is not `200`.

<details><summary>Solution (peek only after attempting the TODOs)</summary>

```python
    resp = requests.get(
        "https://api.github.com/search/repositories",
        params={"q": query, "per_page": per_page, "sort": "stars"},
        headers={
            "Accept": "application/vnd.github+json",
            "User-Agent": "api-basics-lab",
        },
        timeout=10,
    )
    if resp.status_code != 200:
        print(f"search failed: HTTP {resp.status_code}", file=sys.stderr)
        raise SystemExit(1)
    return resp.json()["items"]
```
</details>

```zsh
uv run python src/api_basics/search.py
```

**Checkpoint:** you see five repositories printed with star counts, most-starred first. Confirm `requests` built the query string for you: add `print(resp.url)` temporarily and see the encoded `?q=...&per_page=5&sort=stars`.
**If not:** a `403` mentioning User-Agent means the header dict did not attach — check TODO 1 left `headers=` intact. A `KeyError: 'items'` means the call did not return `200` and your TODO 2 status check is missing, so you tried to read items off an error body.

### Stage 3 — Independent (you do)

Before cross-checking, write one small GET from scratch with no skeleton. Goal: in a new file `src/api_basics/repos.py`, fetch `https://api.github.com/users/octocat/repos` (set a `User-Agent`, a `timeout`, and check the status yourself), then print each repo's `name` and `language`. **Definition of done:** it prints a list and exits cleanly; if you point it at a nonexistent user it prints a one-line error to `stderr` and exits non-zero instead of crashing.

**Checkpoint:** running it lists octocat's repos with their languages.
**If not:** if it crashes on a missing field, you skipped the status check or read a key off an error body — guard with `data.get(...)` (Month 3) and branch on `resp.status_code` first.

### 4. Cross-check against HTTPie

Prove your Python is sending what you think. The HTTPie equivalent of the call above:

```zsh
http GET https://api.github.com/search/repositories \
  q=='language:python topic:cli' per_page==5 sort==stars \
  Accept:application/vnd.github+json User-Agent:api-basics-lab
```

**Checkpoint:** HTTPie returns the same JSON shape (`items`, `total_count`). When your Python and a known-good `curl`/HTTPie call agree, you have isolated "is it my code or the request?" — a debugging move you will use all year.
**If not:** `http: command not found` means HTTPie is not installed — `brew install httpie`. If HTTPie succeeds but Python fails, the bug is in your code, not the request — compare them field by field.

### 5. Send a POST with a JSON body

GitHub's write endpoints need auth, so practice the `POST` shape against a free echo API that requires none. Create `src/api_basics/post_demo.py`:

```python
import requests


def main():
    payload = {"agent": "github-pulse", "month": 4, "alive": True}
    resp = requests.post("https://httpbin.org/post", json=payload, timeout=10)
    resp.raise_for_status()                  # raise on 4xx/5xx
    echoed = resp.json()
    print("server saw Content-Type:", echoed["headers"]["Content-Type"])
    print("server saw body:", echoed["json"])


if __name__ == "__main__":
    main()
```

```zsh
uv run python src/api_basics/post_demo.py
```

**Checkpoint:** the output shows `server saw Content-Type: application/json` and your exact payload echoed back under `json`. Note that `json=` set the body *and* the `Content-Type` header for you — you never serialized anything by hand. `raise_for_status()` is the "fail loud on a bad status" alternative to the manual `if` in Step 3.
**If not:** an `SSLError` or timeout on `httpbin.org` means it is flaky — retry, or use `https://postman-echo.com/post` (see Troubleshooting). A `KeyError` on `headers`/`json` means the echo shape differs on the substitute host; print `echoed` to find the right keys.

### 6. Create a GitHub personal access token

In a browser: GitHub → **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**. Give it a name like `github-pulse`, a short expiry, **read-only public access** (no extra scopes needed this month), and generate it. Copy the value now — GitHub shows it once.

**Checkpoint:** you have a token string starting with `github_pat_` (fine-grained) or `ghp_` (classic) copied to your clipboard.
**If not:** if you navigated away before copying, GitHub will not show the value again — delete the token and generate a fresh one.

### 7. Verify the token with HTTPie *before* touching code

```zsh
http GET https://api.github.com/user \
  "Authorization: Bearer YOUR_TOKEN_HERE" \
  Accept:application/vnd.github+json User-Agent:api-basics-lab
```

**Checkpoint:** you get a `200` and your own GitHub profile JSON back. Verifying the credential outside your code first means that when the Python version fails, you know it is the code, not the token.
**If not:** a `401` means the token is wrong, expired, or you pasted extra whitespace — regenerate and retry. A `403` about User-Agent means you dropped the `User-Agent:` header from the HTTPie line.

### 8. Put the token in `.env` and git-ignore it FIRST

Order matters. Ignore before you create:

```zsh
echo ".env" >> .gitignore
echo ".venv/" >> .gitignore
printf 'GITHUB_TOKEN=%s\n' 'YOUR_TOKEN_HERE' > .env
```

**Checkpoint:** `git status` does **not** list `.env`. If it does, stop and fix `.gitignore` before continuing — this is the whole point.
**If not:** if `.env` shows as tracked, you created it before ignoring it — run `git rm --cached .env`, confirm the `.gitignore` line is exactly `.env`, and re-check `git status`.

### 9. Load the secret and make an authenticated call

Create `src/api_basics/whoami.py`:

```python
import os
import sys
import requests
from dotenv import load_dotenv


def main():
    load_dotenv()                            # read .env into the environment
    token = os.environ.get("GITHUB_TOKEN")
    if not token:
        print("Set GITHUB_TOKEN in .env", file=sys.stderr)
        raise SystemExit(1)

    resp = requests.get(
        "https://api.github.com/user",
        headers={
            "Authorization": f"Bearer {token}",
            "Accept": "application/vnd.github+json",
            "User-Agent": "api-basics-lab",
        },
        timeout=10,
    )
    if resp.status_code == 401:
        print("Token rejected (401). Check it.", file=sys.stderr)
        raise SystemExit(1)
    resp.raise_for_status()
    me = resp.json()
    print(f"Authenticated as {me['login']} ({me.get('name', 'no name')})")
    print("rate limit remaining:", resp.headers.get("X-RateLimit-Remaining"))


if __name__ == "__main__":
    main()
```

```zsh
uv run python src/api_basics/whoami.py
```

**Checkpoint:** you see `Authenticated as <your-login>` and a rate-limit-remaining number near 5000 (authenticated GitHub requests get a much higher budget than the ~60/hour for anonymous ones — note that for Lab 2). The token came from `.env`, never from code.
**If not:** a `401` here but a working HTTPie call in Step 7 means the token did not load — confirm `.env` is in the project root, the line is exactly `GITHUB_TOKEN=...` with no quotes, and `load_dotenv()` runs before `os.environ`. See Troubleshooting.

### 10. Run the leak check before any push

```zsh
git add -A
git commit -m "Lab 1: requests basics + auth"
git log -p | grep -i -E 'github_pat_|ghp_|GITHUB_TOKEN=' || echo "CLEAN: no token in history"
```

**Checkpoint:** you see `CLEAN: no token in history`. Your committed code references `GITHUB_TOKEN` as a *name* only; the *value* lives solely in the ignored `.env`. (The `grep` matching `GITHUB_TOKEN=` would catch an accidentally committed `.env`; the var name in your `.py` files has no `=value` so it does not trip it.)
**If not:** if the grep prints a token line, `.env` was committed. Stop, rotate (revoke) the token on GitHub immediately — it is compromised — then `git rm --cached .env`, fix `.gitignore`, and scrub history before any push.

### 11. (90-second peek) The same call in `httpx`

```python
# uv run python -c "..."  or a scratch file
import httpx
r = httpx.get("https://api.github.com/users/octocat", timeout=10)
print(r.status_code, r.json()["login"])
```

**Checkpoint:** identical output to Step 1 with a near-identical API. You now recognize `httpx` as "`requests` plus async (which we don't need yet)."
**If not:** a `ModuleNotFoundError: httpx` means the peek dependency was skipped — run `uv add httpx`.

## Definition of Done

- `uv run python src/api_basics/first_get.py` prints a `200` and a parsed field.
- `search.py` returns repositories using `params=` and a `User-Agent`, and fails cleanly (non-zero exit, `stderr` message) on a non-200.
- `post_demo.py` shows the server received your JSON body with `Content-Type: application/json`.
- `whoami.py` authenticates using a token loaded from `.env` and prints your login plus the rate-limit-remaining header.
- `.env` is git-ignored, absent from `git status`, and the leak check prints `CLEAN`.
- Self-verify in one line:

```zsh
git status --porcelain | grep -q '\.env' && echo "FAIL: .env tracked" || echo "OK: .env ignored"
```

**Self-explain:** in one sentence, why does loading the token from `.env` keep your secret safe even though the code that uses it is committed to Git?

## Stretch Goals

1. **Handle a deliberate 404.** Point `first_get.py` at `https://api.github.com/users/this-user-does-not-exist-zzz` and make it print a clean "user not found" instead of a `KeyError` on `data['login']`.
2. **Read more headers.** Print `X-RateLimit-Limit` and `X-RateLimit-Reset` from `whoami.py` and convert the reset timestamp to a human time with `datetime.fromtimestamp`.
3. **A `Session`.** Replace repeated `headers=` with a `requests.Session()` that carries the `User-Agent` and `Authorization` for every call — a pattern Lab 3 uses.
4. **Compare bodies.** Add `data=` (form-encoded) vs `json=` to `post_demo.py` and observe how `httpbin` reports each differently.

## Troubleshooting

- **`403` with a message about User-Agent.** GitHub rejects requests with no `User-Agent`. Add one to `headers=`.
- **`.json()` raises `JSONDecodeError`.** The body was not JSON (often an HTML error page). Check `resp.status_code` and `resp.headers["Content-Type"]` before calling `.json()`.
- **`401 Unauthorized` in `whoami.py` but HTTPie worked.** The token did not load. Confirm `.env` is in the project root, the line is exactly `GITHUB_TOKEN=...` with no quotes or spaces, and you called `load_dotenv()` before reading `os.environ`.
- **`KeyError: 'GITHUB_TOKEN'`.** `.env` missing or misnamed (it must be literally `.env`), or `load_dotenv()` not called. Use `os.environ.get(...)` and fail with a clear message, as the code does.
- **`.env` shows up in `git status`.** You created it before adding it to `.gitignore`, or it is already tracked. `git rm --cached .env`, confirm the `.gitignore` line, and re-check.
- **`ModuleNotFoundError: requests` / `dotenv`.** Run via `uv run python ...` (not bare `python`), and confirm `uv add requests python-dotenv` succeeded — check they appear in `pyproject.toml`.
- **`SSLError` on `httpbin.org`.** Occasionally flaky; retry, or substitute `https://postman-echo.com/post` (same echo shape under a different top-level key).
