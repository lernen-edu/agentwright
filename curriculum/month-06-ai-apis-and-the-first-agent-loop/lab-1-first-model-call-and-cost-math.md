---
title: Lab 1 — First Model Call and Cost Math
parent: Month 06 — AI APIs and the First Agent Loop
grand_parent: Curriculum
nav_order: 1
---

# Lab 1 — First Model Call and Cost Math

**Time: ~3 hrs · Difficulty: Intro · Builds on: Month 4 (requests, .env), Month 5 (functions, type hints, structured logging)**

## Objective

Make your first language-model call — and make it the *right* way, by hand, over raw HTTP, so nothing is hidden. You will install Ollama and run an open model locally for **$0**, POST a chat-completion request to it with plain `requests`, read the reply, and pull the token counts out of the response. You will then write a reusable `call_model()` function that returns the text *and* a token/cost record, and (optionally) point it at the paid Anthropic API to see the same call cost real money. By the end you can call a model, read its usage, and state the dollar cost of any call.

## Setup

```zsh
# Install Ollama (the local-model runtime) and pull two small models
brew install ollama
ollama serve &            # starts the local server on http://localhost:11434
ollama pull llama3.1:8b   # ~4.7 GB; general chat
ollama pull qwen2.5:7b    # ~4.7 GB; strong tool-use support, used in Lab 3-4

# New uv project for the month
mkdir -p ~/agentic/month-06 && cd ~/agentic/month-06
uv init . --python 3.12
uv add requests python-dotenv
```

**Checkpoint:** Run `ollama list` and you should see `llama3.1:8b` and `qwen2.5:7b`. Run `curl http://localhost:11434/api/tags` and you should get JSON listing your models. If `ollama serve &` says the address is already in use, the server is already running (the Homebrew install may auto-start it) — that is fine.

> **Note on `ollama serve`.** The `&` backgrounds the server in your current terminal. If you close the terminal it stops. To keep it running, either leave that terminal open or run `brew services start ollama` once to have macOS keep it alive.

## Background

**Recall first (from memory):** From Month 4 — what one `requests` method sends a JSON body, and how do you turn the reply into a dict? What does `resp.raise_for_status()` do? If those are crisp, this lab is "Month 4, new URL."

Ollama exposes two HTTP shapes: its own `/api/chat`, and an **OpenAI-compatible** one at `/v1/chat/completions`. We use the OpenAI-compatible shape throughout this month because it is the same shape thousands of tools speak, and it maps cleanly onto the concepts in the README (roles, `max_tokens`, `stop`). The Anthropic Messages API is a *different* shape (system prompt as a separate field, slightly different fields), which is exactly why one `call_model()` wrapper is worth writing.

## Steps

### 1. The raw call, with curl

Before any Python, prove the endpoint works with the `curl`/`jq` skills from Month 2:

```zsh
curl -s http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3.1:8b",
    "messages": [
      {"role": "system", "content": "You are terse. One sentence only."},
      {"role": "user", "content": "Why is the sky blue?"}
    ],
    "max_tokens": 60,
    "temperature": 0
  }' | jq
```

**Checkpoint:** You get a JSON object with a `choices[0].message.content` string (the answer) and a `usage` object containing `prompt_tokens`, `completion_tokens`, and `total_tokens`. Note those token numbers — that is what you would be billed on against a paid provider.
**If not:** `curl: (7) Failed to connect` means the server isn't up — run `ollama serve` (Troubleshooting). An error JSON mentioning the model means you skipped `ollama pull llama3.1:8b`. If `jq` isn't found, `brew install jq` (a Month 2 tool).

### 2. The same call in Python, over raw HTTP — Stage 1: Worked example (I do)

This is the new skill of the lab: calling a model over raw HTTP and reading its reply. Study this complete, working example — you are not inventing anything yet, just running it and reading every line. Note that the structure is identical to a Month 4 API call: build a JSON body, POST, `raise_for_status()`, `.json()`, then index into the result.

Create `first_call.py`:

```python
import requests

resp = requests.post(
    "http://localhost:11434/v1/chat/completions",
    json={
        "model": "llama3.1:8b",
        "messages": [
            {"role": "system", "content": "You are terse. One sentence only."},
            {"role": "user", "content": "Why is the sky blue?"},
        ],
        "max_tokens": 60,
        "temperature": 0,
    },
    timeout=120,
)
resp.raise_for_status()
data = resp.json()

print("REPLY:", data["choices"][0]["message"]["content"])
print("USAGE:", data["usage"])
```

```zsh
uv run python first_call.py
```

**Checkpoint:** You see `REPLY:` followed by a one-sentence answer, then `USAGE:` with the three token counts. You have now called a model from your own code — the same `requests.post(...).json()` pattern from Month 4, pointed at a model instead of a weather API.
**If not:** a `ConnectionError` means Ollama isn't serving — see Troubleshooting (`Connection refused`). A `KeyError` on `choices` or `usage` usually means you hit `/api/chat` instead of `/v1/chat/completions`; check the URL. A `ReadTimeout` on first run is the model loading into memory — re-run.

### 3. Feel the knobs

Edit `first_call.py` and experiment, re-running after each change:

- Set `"max_tokens": 10`. **Checkpoint:** the reply is cut off mid-sentence — proof that `max_tokens` guillotines output, it does not make the model concise.
- Remove the `system` message and ask the same question. **Checkpoint:** the answer is longer and chattier — the system prompt was doing real work.
- Add `"stop": ["."]`. **Checkpoint:** generation halts at the first period.
- Set `"temperature": 1.5` and run three times. **Checkpoint:** answers vary run-to-run; at `0` they are (near-)identical.

**If not (any of the above):** if nothing changes when you edit the file, you may be running a stale copy — confirm you saved and re-ran the right path. If `stop` has no effect, the model may not have generated that character yet within `max_tokens`; raise the cap a little. Small models are sometimes still chatty at low `max_tokens` — that's expected variance, not a failure.

### 4. Tokens: count them yourself

The `usage` block told you the real count, but you should be able to *estimate* one offline. Add `tiktoken` and compare:

```zsh
uv add tiktoken
```

```python
# token_math.py
import tiktoken

text = "Why is the sky blue? Explain like I'm five."
enc = tiktoken.get_encoding("cl100k_base")   # a common BPE encoding; an approximation for any model
tokens = enc.encode(text)
print(f"{len(text)} chars, {len(text.split())} words, {len(tokens)} tokens")
```

```zsh
uv run python token_math.py
```

**Checkpoint:** tokens land between the word count and the character count — confirming the rule of thumb *1 token ≈ 4 chars ≈ ¾ word*. (This encoding is an approximation; each provider has its own tokenizer. The `usage` field is the source of truth for billing.)
**If not:** if `tiktoken` fails to import, re-run `uv add tiktoken` and use `uv run`. If the token count looks wildly off (e.g., equal to char count), you likely encoded an empty or wrong string — print `text` to confirm.

### 5. The dollar math

Create `cost.py` — a tiny module you will reuse all month:

```python
# cost.py
from dataclasses import dataclass

# Dollars per MILLION tokens. Ollama is free. Anthropic numbers are illustrative;
# check https://www.anthropic.com/pricing for current values.
PRICES = {
    "ollama":              {"in": 0.00, "out": 0.00},
    "claude-haiku":        {"in": 0.80, "out": 4.00},
    "claude-sonnet":       {"in": 3.00, "out": 15.00},
}

@dataclass
class Usage:
    model_class: str
    input_tokens: int
    output_tokens: int

    @property
    def dollars(self) -> float:
        p = PRICES[self.model_class]
        return (self.input_tokens / 1_000_000) * p["in"] + \
               (self.output_tokens / 1_000_000) * p["out"]


if __name__ == "__main__":
    u = Usage("claude-haiku", input_tokens=1500, output_tokens=500)
    print(f"{u.input_tokens} in + {u.output_tokens} out -> ${u.dollars:.6f}")
    free = Usage("ollama", input_tokens=1500, output_tokens=500)
    print(f"Same call on Ollama -> ${free.dollars:.6f}")
```

```zsh
uv run python cost.py
```

**Checkpoint:** the Haiku-class call prints `$0.003200` and the Ollama call prints `$0.000000`. You can now price any call.
**If not:** a `KeyError` means `model_class` doesn't match a key in `PRICES` (check spelling, e.g. `"claude-haiku"`). A wrong number almost always means the `/ 1_000_000` is missing or misplaced — re-derive it from the formula in README §3.

### 6. The reusable `call_model()` — Stage 2: Faded practice (we do)

Now wrap step 2's call into one reusable function that returns both the text *and* a `Usage`. The scaffolding is here, but the mechanical parts are yours to fill in — you already wrote each line's equivalent in steps 2 and 5. Create `model.py` and complete the four `# TODO`s:

```python
# model.py
import logging
import requests
from cost import Usage

logging.basicConfig(level=logging.INFO, format="%(levelname)s %(message)s")
log = logging.getLogger("model")

OLLAMA_URL = "http://localhost:11434/v1/chat/completions"

def call_model(
    messages: list[dict],
    model: str = "llama3.1:8b",
    max_tokens: int = 512,
    temperature: float = 0.0,
) -> tuple[str, Usage]:
    """Call a local Ollama model (OpenAI-compatible). Returns (text, Usage)."""
    resp = requests.post(
        OLLAMA_URL,
        json={
            "model": model,
            "messages": messages,
            "max_tokens": max_tokens,
            "temperature": temperature,
        },
        timeout=120,
    )
    resp.raise_for_status()                       # TODO 1: why is this line non-negotiable?
    data = resp.json()
    text = ...                                    # TODO 2: pull the reply text (see step 2)
    usage = Usage(
        model_class="ollama",
        input_tokens=...,                         # TODO 3: from data["usage"]["prompt_tokens"]
        output_tokens=...,                        # TODO 4: from data["usage"]["completion_tokens"]
    )
    log.info("tokens in=%d out=%d cost=$%.6f",
             usage.input_tokens, usage.output_tokens, usage.dollars)
    return text, usage


if __name__ == "__main__":
    text, usage = call_model([
        {"role": "system", "content": "You are terse. One sentence only."},
        {"role": "user", "content": "What is an API, in one sentence?"},
    ])
    print("REPLY:", text)
```

<details><summary>Check the four TODOs</summary>

1. The endpoint can return an HTTP error (model not pulled, bad body); `raise_for_status()` turns that into a clean exception instead of letting you index into an error body and crash with a confusing `KeyError`.
2. `text = data["choices"][0]["message"]["content"]`
3. `input_tokens=data["usage"]["prompt_tokens"]`
4. `output_tokens=data["usage"]["completion_tokens"]`
</details>

```zsh
uv run python model.py
```

**Checkpoint:** you see an `INFO tokens in=… out=… cost=$0.000000` log line (your Month 5 structured-logging habit, now reporting cost as a first-class metric) followed by the reply. This `call_model()` is the seed of the agent you build in Lab 4.
**If not:** a `TypeError: 'ellipsis' ...` or `AttributeError` means a `...` TODO is unfilled — complete all four. No log line at all means `logging.basicConfig` was overridden elsewhere; confirm it runs at import. A `KeyError: 'usage'` means you're on `/api/chat`, not `/v1/...`.

### 6b. Use it on a fresh prompt — Stage 3: Independent (you do)

No scaffolding now. Goal: from a *new* Python file (e.g., `try_it.py`), `from model import call_model`, send a two-message conversation of your own (a `system` instruction plus a `user` question), print only the reply text, and then print the cost using the returned `Usage`. Definition of done: it runs, prints a sensible answer, and prints a `$0.000000` cost line. You should now be able to call a model and price the call without copying any earlier code.

### 7. (Optional, paid) The same call against Anthropic

This step costs a fraction of a cent and is **entirely optional** — skip it and lose nothing required. It exists so you see the paid path with your own eyes.

Get a key from the [Anthropic console](https://console.anthropic.com), add a few dollars of credit, then:

```zsh
echo 'ANTHROPIC_API_KEY=sk-ant-...' >> .env   # never commit this; add .env to .gitignore
uv add anthropic
```

```python
# anthropic_call.py
import os
from dotenv import load_dotenv
from anthropic import Anthropic
from cost import Usage

load_dotenv()
client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

resp = client.messages.create(
    model="claude-3-5-haiku-latest",
    max_tokens=60,
    system="You are terse. One sentence only.",        # NOTE: a top-level field, not a message
    messages=[{"role": "user", "content": "Why is the sky blue?"}],
)
print("REPLY:", resp.content[0].text)
u = Usage("claude-haiku", resp.usage.input_tokens, resp.usage.output_tokens)
print(f"USAGE: in={u.input_tokens} out={u.output_tokens} cost=${u.dollars:.6f}")
```

```zsh
uv run python anthropic_call.py
```

**Checkpoint:** the reply prints and `USAGE:` shows a real, non-zero cost (a tiny fraction of a cent). Note the structural difference you read about: `system=` is a **separate field**, not a message in the list. That difference is exactly what `call_model()` exists to hide — you will unify both providers in Lab 4.
**If not:** an `authentication_error` means the key is wrong or `.env` wasn't loaded — see the Troubleshooting check. A `credit`/`billing` error means you haven't added credit in the console. This whole step is optional; skip it and lose nothing required.

## Definition of Done

- [ ] `ollama list` shows `llama3.1:8b` and `qwen2.5:7b`, and `curl http://localhost:11434/api/tags` returns JSON.
- [ ] `uv run python first_call.py` prints a reply and a `usage` block with token counts.
- [ ] You have observed `max_tokens` truncating output and the system prompt changing behavior.
- [ ] `uv run python cost.py` prints `$0.003200` for the Haiku-class call and `$0.000000` for Ollama.
- [ ] `uv run python model.py` logs `tokens in/out cost=$…` and prints a reply.
- [ ] You can state the cost of a 2,000-in / 1,000-out call at $1/M in, $5/M out without help. (Answer: $0.007.)

Self-verify in one line:

```zsh
uv run python -c "from cost import Usage; print(Usage('claude-haiku',2000,1000).dollars)"
# expect: 0.0056   (2000/1e6*0.80 + 1000/1e6*4.00 = 0.0016 + 0.004 = 0.0056 for haiku)
```

> The figure above uses your `PRICES` table (Haiku). The $0.007 self-check uses the hypothetical $1/$5 prices from the question — make sure you can do *both* by hand.

**Self-explain:** in one sentence, why is calling a language model just a special case of the Month-4 skill of POSTing JSON to an API and parsing the reply?

## Stretch Goals

1. **A second model.** Repeat step 2 with `qwen2.5:7b` and compare the answers and token counts on the same prompt.
2. **Streaming preview.** Add `"stream": true` to the curl call and watch the response arrive as a stream of `data:` events (you will build proper streaming in Lab 2).
3. **A running tally.** Make `call_model()` append each `Usage` to a module-level list and add a `total_cost()` function — the first step toward per-run cost tracking.
4. **Real tokenizer.** If you did the paid step, compare `tiktoken`'s estimate to Anthropic's reported `input_tokens` for the same prompt and note how far off the approximation is.

## Troubleshooting

- **`Connection refused` on localhost:11434.** Ollama's server is not running. Run `ollama serve` (or `brew services start ollama`) and retry.
- **First call is very slow.** The model loads into memory on first use; subsequent calls are fast. An 8B model needs ~6–8 GB of RAM free.
- **`KeyError: 'usage'`.** You hit `/api/chat` instead of `/v1/chat/completions`; only the OpenAI-compatible endpoint returns the `usage` block in this shape. Check the URL.
- **`requests.exceptions.ReadTimeout`.** Increase `timeout=` or use a smaller model; large prompts on an 8B model can be slow on first load.
- **Anthropic `authentication_error`.** The key in `.env` is wrong, or `load_dotenv()` did not find the file (run from the project root). Confirm with `uv run python -c "import os,dotenv; dotenv.load_dotenv(); print(bool(os.environ.get('ANTHROPIC_API_KEY')))"`.
- **Accidentally committed `.env`.** Add `.env` to `.gitignore` immediately, rotate the key in the Anthropic console, and remove it from history. Treat any leaked key as compromised.
