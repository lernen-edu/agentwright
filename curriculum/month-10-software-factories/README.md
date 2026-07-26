---
title: Month 10 — Software Factories (AI Developer Workflows)
parent: Curriculum
nav_order: 10
permalink: /curriculum/month-10-software-factories/
---

# Month 10 — Software Factories (AI Developer Workflows)

**Pillar 2 — Software Factories**

## Overview

For four months you have been building *one thing well*: a from-scratch agent loop (Month 6), made provider-agnostic with fallback (Month 7), given safe hands on the shell, filesystem, and network (Month 8), and finally promoted from agent to harness — a lead/worker/validator team welded to one domain (Month 9). You can now point a custom multi-agent harness at a known job and get a reliable result. This month you take that capability and aim it at the one domain that pays for all the others: **writing software itself.**

Here is the thesis, and it is the spine of Pillar 2: **stop building features; build the system that builds features.** Almost everyone using AI to code is still producing *outputs* — they prompt a model, get a chunk of code, paste it in, fix what's broken, and repeat. That is producing one feature at a time, by hand, with an expensive autocomplete. The move that separates an agentic engineer from everyone else is to stop producing outputs and start producing **a function that produces outputs.** A feature is an output. A *factory* is a function: feed it a spec, and it reliably emits working, tested, reviewed code — on spec, every time, with no human edits between the request and the pull request. This is "the system that builds the system" made concrete.

Once you see software as a compilation target, the architecture follows. A factory is a **pipeline** of stages — PLAN, SCOUT, BUILD, VALIDATE, TEST, REVIEW — and each stage is a small agent with its own prompt, its own tool stack, and its own evals. The heart of the whole machine is the **plan prompt**: a structured specification the factory *consumes*, the way a compiler consumes source. Get the spec format right and the rest of the pipeline becomes boring and predictable, which is exactly what you want from the stages that touch real code. The deepest lesson of the month is the shift in the question you ask. You stop asking "how do I get the model to do X?" and start asking "**what spec format makes X trivial to produce?**" — because the leverage was never in the prompt that builds one feature; it was in the spec format that makes every feature buildable.

The month ends with the **Mini Feature Factory** milestone: a CLI that takes a one-paragraph feature request for a real Python web app and runs it through all six stages to produce a PR with code, tests, and a changelog entry. The bar is demanding and deliberate — it must succeed on **five different feature requests, on the first try, with zero human edits to the produced PR** — and you will track dollars-per-feature, time-to-PR, and success rate, because a factory you cannot measure is a factory you cannot trust.

Here is the machine you are building, end to end — the central mental model for the whole month:

```mermaid
flowchart LR
    R["Fuzzy request"] --> P["PLAN: spec"]
    P --> S["SCOUT: read repo"]
    S --> B["BUILD: write code"]
    B --> V["VALIDATE: ruff + mypy"]
    V --> T["TEST: pytest + coverage"]
    T --> RV["REVIEW: diff vs spec"]
    RV --> PR["Open PR"]
```

*Notice: it is a one-way assembly line. A request enters as prose; a PR exits as the artifact. Each box is a small agent or a real tool, not one giant prompt — and the creative work happens at the front (PLAN), so everything after it can be boring and reliable.*

## Prerequisites

Coming in, you should be able to do everything from Months 1 through 9:

- Work fluently in zsh on macOS, use Git and `gh`, read HTTP/JSON, and call APIs from Python with timeouts, retries, and `.env`-loaded secrets (Months 1–2, 4).
- Write structured Python with `Protocol`-based interfaces, dependency injection, `pytest`, strict type hints, and structured logging (Month 5).
- Hand-write and explain the agent loop, run a tool-call round-trip, apply a working-directory jail, and write a JSONL trace (Month 6).
- Swap the model and tool layers behind interfaces with a config-driven fallback chain to local Ollama (Month 7).
- Invoke a CLI safely with `subprocess` (argument lists, timeouts), enforce an allowlist and a hardened jail, and rate tools by danger with a human gate (Month 8).
- Build a lead/worker/validator harness where sub-agents run in their own subprocesses with their own working directories and per-role models, and every run is traced and replayable (Month 9).

You do **not** need prior CI/CD or compiler experience. The pipeline is built from the orchestration patterns you already own.

## Warm-Up: Retrieve Before You Begin

Before reading on, answer these from memory — no peeking at earlier months. This pulls forward the prior skills this month builds on.

1. In Month 5 you used three tools as **quality gates** on your code. Name them and say what class of problem each catches.
2. In Month 6 you made the model emit a structured object instead of free prose, and you judged each run with an eval. Why is a *structured* output easier to gate on than a paragraph of text?
3. In Month 9 your harness had a lead that *decomposed* a job into worker tasks. If you froze that decomposition so it never changed — the same fixed sequence of specialized workers every time — what would you call the result?

<details><summary>Check your recall</summary>

1. **`ruff`** (lint + format — style and obvious errors), **`mypy`** (static type checking — type mismatches before runtime), and **`pytest`** (tests — behavior is actually correct). Month 5. These exact three become the VALIDATE and TEST stages this month.
2. A structured output (e.g. a Pydantic object or JSON schema) gives you *fields to assert on* — an eval can check "is `acceptance_criteria` non-empty?" deterministically, whereas prose forces a fuzzy judgment. Month 6. This month's PLAN stage emits a structured `Spec` for exactly this reason.
3. A **pipeline** — a fixed orchestration. Month 9's lead/worker/validator pattern, with the decomposition baked in instead of decided at runtime. The six-stage factory *is* this generalization.
</details>

## Learning Objectives

By the end of this month you can:

1. **Explain** the mindset shift from producing outputs to producing a function that produces outputs, and **articulate** why a factory beats hand-prompting one feature at a time.
2. **Design** a plan-prompt / `SPEC.md` format that turns a fuzzy one-paragraph request into a structured specification a pipeline can consume deterministically.
3. **Build** the six-stage pipeline — PLAN, SCOUT, BUILD, VALIDATE, TEST, REVIEW — each as a stage with its own prompt, tool stack, and eval.
4. **Decide** determinism vs. creativity per stage, and **justify** why planning is allowed to be creative while building should be boring and predictable.
5. **Wire** real engineering tools into the pipeline — `ruff`, `mypy`, and `pytest` via `uv` — so VALIDATE and TEST run actual checks, not model opinions.
6. **Implement** an independent REVIEW stage where a second agent checks the diff against the spec before merge, and **explain** why the builder cannot be its own reviewer.
7. **Instrument** every stage with structured telemetry so each run is queryable and **cost-per-produced-artifact** is tracked and reported.
8. **Ship** a Mini Feature Factory that produces a PR (code + tests + changelog) from a one-paragraph request, succeeding on five varied features on the first try with zero human edits.
9. **Measure** a factory: report dollars-per-feature, time-to-PR, and success rate, and **explain** how the spec format — not the prompt — drives first-try success.

## Tech Stack (free, macOS)

| Tool | Install | Why |
|---|---|---|
| Python 3.12+ via uv | `brew install uv`; `uv python install 3.12` | From Month 3. The factory is a `uv` project; each stage and the target app run under `uv run`. |
| Ollama + two models | `brew install ollama`; `ollama pull qwen2.5-coder:7b` and `ollama pull qwen2.5:3b` | The **free** model layer. A coder model builds; a small model does cheap classification/review. Whole pipeline runs at **$0**. |
| Your Month 7 `llm` package | (from Month 7) | Pluggable providers + fallback. Each stage names a provider/model and inherits the fallback chain. |
| Your Month 8 guardrails | (from Month 8) | The jail, allowlisted `subprocess` runner, and danger levels — reused so BUILD/VALIDATE/TEST touch only the target repo. |
| Your Month 9 orchestration | (from Month 9) | The lead/worker/validator and per-run trace patterns — a pipeline is a *fixed* orchestration; the stages are specialized workers. |
| `ruff` | `uv add --dev ruff` | The VALIDATE stage: fast lint + format check. Real static analysis, not a model's opinion of style. |
| `mypy` | `uv add --dev mypy` | The VALIDATE stage: static type checking. Catches a whole class of build errors before tests run. |
| `pytest` + `pytest-cov` | `uv add --dev pytest pytest-cov` | The TEST stage: run existing tests, run generated ones, enforce a coverage threshold. |
| `git` + `gh` | `brew install git gh` | The output: a real branch, commit, and PR. `gh pr create` produces the artifact the factory exists to make. |
| `pydantic` | `uv add pydantic` | Parse and validate the `SPEC.md` / plan object so a malformed spec fails loudly at stage 1, not at BUILD. |
| FastAPI target app | `uv add fastapi` (in the *target* repo) | A small, real Python web app the factory builds features *into* — the workpiece, not part of the factory. |
| `anthropic` / `openai` (optional, paid) | `uv add anthropic` | Only if you want a frontier model for BUILD or REVIEW. A local coder model substitutes for **$0**; the cost is always shown. |

**Cost summary.** This month is **$0-completable.** Two local Ollama models — a coder model for BUILD and a small model for PLAN/SCOUT/REVIEW classification — cover the whole pipeline, including the five-feature milestone, for free. The honest caveat, stated plainly: **first-try success rate is the metric this month lives and dies by, and a more capable model raises it.** A frontier model on the BUILD and REVIEW stages will clear five varied features on the first try more reliably than a 7B local model will. That is exactly why telemetry is mandatory — you report dollars-per-feature *and* success rate on whichever model you ran, so the cost/quality tradeoff is visible instead of hidden. The free path completes the milestone; the paid path (labeled wherever it appears) buys a higher first-try rate for a few cents per feature.

## Weekly Breakdown

Budget ~8–12 hours per week: roughly a third on the Core Concepts and the spec design, the rest building stages and running features through them.

### Week 1 — The mindset shift and the plan prompt
**Warm-start (do this first):** before any new material, re-open your Month 9 harness and run it once on a known job, then read your own `RunTrace` output. You will reuse that exact trace machinery for per-stage telemetry this month — re-running it now keeps the orchestration-and-trace skill live and reminds you what a per-step event log looks like.
**Focus:** internalize "build the factory, not the feature," then design the spec format that is the factory's source language.
**Topics:** output vs. function (a feature is an output; a factory is a function that produces outputs); **spec-driven development** (the spec is the source of truth, code is a compilation target); the **plan prompt** as the heart of the factory; what a good `SPEC.md` contains (intent, acceptance criteria, touchpoints, constraints, out-of-scope) and what makes one machine-consumable; **determinism vs. creativity** (planning is the creative step; everything downstream should be predictable); the question shift — from "how do I prompt X" to "what spec format makes X trivial."
**Reading:** Core Concepts §1–§3.
**Build:** Lab 1 — design a `SPEC.md` format, build the **PLAN** stage that turns a fuzzy one-paragraph request into a validated structured spec (parsed with `pydantic`), and write three example specs by hand to pressure-test the format.

### Week 2 — The pipeline: six stages, each with prompt, tools, and evals
**Focus:** build the assembly line — PLAN → SCOUT → BUILD → VALIDATE → TEST → REVIEW — wiring real tools and per-stage evals.
**Topics:** the **stage** abstraction (each stage has a prompt, a tool stack, an eval, and emits telemetry); **SCOUT** (read the target codebase, find touchpoints, prior art, and conventions before building); **BUILD** (implement the spec honoring Scout's conventions — the boring, deterministic stage); **VALIDATE** (`ruff` + `mypy` as a hard gate); **TEST** (`pytest` + coverage, generate new tests); **REVIEW** (an independent agent checks the diff against the spec); per-stage **telemetry** (every stage emits structured logs; cost-per-artifact is tracked); failure handling (a stage that fails its eval re-runs or fails the run loudly).
**Reading:** Core Concepts §4–§7.
**Build:** Lab 2 — implement all six stages as composable units over a scaffolded FastAPI app, each with its own prompt/tools/eval and structured telemetry, and run a single feature request through the whole pipeline to a diff.

### Week 3 — The Mini Feature Factory (milestone)
**Focus:** close the loop to a real PR and prove it on five varied features with metrics.
**Topics:** turning a passing pipeline run into a **PR** (branch, commit, `gh pr create`) with code, tests, and a changelog entry, reusing Month 8 git safety; the **metrics table** (dollars/feature, time-to-PR, success rate) and how to compute it from the trace; designing the five feature requests across a complexity gradient; what "first try, zero human edits" actually disciplines you to build; iterating on the **spec format** when a feature fails — fix the spec, not the prompt.
**Reading:** Core Concepts §8–§9.
**Build:** Lab 3 — the **Mini Feature Factory** milestone: a CLI that takes a one-paragraph request and emits a PR; run it on five features of varying complexity, link the five PRs, and produce the metrics table and the finalized `SPEC.md`.

### Week 4 — Hardening, the spec retrospective, and the defense
**Focus:** raise the first-try success rate by improving the spec format and the evals, and write up what you learned.
**Topics:** reviewing the factory through the five lenses (is each stage's blast radius bounded to the target repo? is every run queryable? is cost-per-artifact reported?); the **spec retrospective** (for each feature that failed first-try, was the fix a better spec or a better stage?); tightening REVIEW so it rejects spec-violations it currently passes; writing the short defense — why this is a factory and not a fancy autocomplete.
**Reading:** re-read §1–§9 as a checklist; review your `METRICS.md` and `SPEC.md`.
**Build:** finish and harden Lab 3; raise success rate toward 5/5; write `RETRO.md` capturing the spec changes that moved the metric.

## Core Concepts

### §1 — A feature is an output; a factory is a function that produces outputs

This is the whole month in one sentence, so sit with it. When you prompt a model to "add a rate limiter to the login endpoint" and paste the result in, you have produced **one output**. Tomorrow's feature starts from zero: a new prompt, a new paste, a new round of fixing. You are the assembly line, and you do not scale. The agentic engineer's move is to climb one level of abstraction: instead of producing the feature, you build **the function whose input is a feature request and whose output is a merged-quality PR.** Run it a hundred times and it produces a hundred features, each on spec, without you hand-holding any of them.

This is the same climb you have made before, now applied to your own work. In Month 6 you stopped *being* the loop and built the loop. In Month 9 you stopped *being* the team and built the team. Now you stop *being* the developer-who-uses-AI and build the developer. The economic consequence is the point of the entire pillar: a feature's value is linear (one feature, once), but a factory's value compounds (every feature, forever, at near-zero marginal cost). When the factory is good, your job stops being "write this feature" and becomes "improve the factory so this *class* of features writes itself." If you only take one idea from Month 10, take this: **the leverage is in the function, not the output.**

> **Common misconception.** A factory is just a long, well-engineered prompt — if you stuff enough instructions into one mega-prompt, you get the same thing.
> **Reality.** A factory is *staged agents plus real code plus deterministic gates*. The leverage comes from breaking the work into stages with their own tools and evals — `ruff` exits non-zero, `pytest` fails, REVIEW rejects — none of which a single prompt can do. A long prompt is still one output; it cannot gate, retry a single stage, or report cost-per-artifact. The mega-prompt is tempting because it feels like "more prompt = more capable," but capability here comes from *structure between* model calls, not from a bigger call.

### §2 — Spec-driven development: the spec is source, code is a compilation target

If a factory is a function, what is its input language? It is the **specification.** This reframes the relationship between specs and code that most developers carry around. Normally a spec is a vague document you read once and then drift from as you code; the code becomes the real source of truth and the spec rots. In a factory the polarity flips: **the spec is the source of truth, and the code is a compilation target generated from it.** You don't edit the produced code by hand any more than you'd edit compiler-emitted assembly by hand — if the output is wrong, you fix the *spec* (or the stage that compiles it) and re-run.

> **Common misconception.** If the model is good enough, you don't really need a spec — a strong model will just figure out what you meant from the one-paragraph request.
> **Reality.** The spec *is* the source of truth, and a better model does not remove the need for it — it just compiles a good spec more reliably. With no spec there is nothing for TEST to generate tests from, nothing for REVIEW to check the diff against, and no way to tell "the feature is wrong" from "the model is wrong." The belief is tempting because frontier models often produce *plausible* code from a vague ask — but plausible is not "on spec," and you cannot prove on-spec without a spec. (This is also why a more capable model raises your *success rate* but never lets you skip Lab 1.)

This is not a metaphor you adopt for fun; it is a *discipline that makes the milestone's "zero human edits" bar achievable.* The instant you allow yourself to hand-tweak the produced PR, you are back to producing outputs — you've smuggled yourself back onto the assembly line. The "zero edits" rule is what forces the spec and the pipeline to carry the full weight. A practical consequence: a good spec is **complete enough to compile.** A request like "make it faster" cannot compile, because the factory has nothing concrete to honor. "Add a Redis-backed cache to `GET /products` with a 60-second TTL, returning the cached body unchanged, and a `/products?fresh=true` bypass" *can* compile — it names the touchpoint, the behavior, the acceptance criteria. Learning to recognize the difference, and to design a spec format that pushes requests toward the compilable end, is most of Lab 1.

### §3 — The plan prompt: the heart of the factory

> **Heavy concept ahead.** Slow down here; this is the load-bearing idea of the month. Everything in Labs 1–3 either feeds this stage or consumes its output. If the spec format is wrong, no downstream stage can save the run.

The **plan prompt** is the single most important artifact in the factory, because it is the stage that converts a *fuzzy human request* into the *structured spec* every later stage consumes. Everything downstream is only as good as the spec PLAN emits. A good plan prompt does three jobs: it **extracts intent** from a loose paragraph, it **fills gaps with explicit, reasonable defaults** (and records them, so they are reviewable rather than silent), and it **emits a rigid, validated structure** — not prose. That structure is your `SPEC.md` schema. A workable starting schema:

```python
from pydantic import BaseModel

class Spec(BaseModel):
    title: str                       # one-line feature name
    intent: str                      # what and why, 1-3 sentences
    touchpoints: list[str]           # files/modules likely to change (SCOUT refines this)
    acceptance_criteria: list[str]   # testable statements -> become tests in TEST
    constraints: list[str]           # "no new deps", "keep response schema", etc.
    out_of_scope: list[str]          # explicit non-goals; bounds the blast radius
    assumptions: list[str]           # defaults PLAN chose to fill gaps (reviewable)
```

Notice what this schema *forces*. `acceptance_criteria` must be **testable**, which is what lets TEST generate tests from the spec instead of guessing. `out_of_scope` and `constraints` bound what BUILD is allowed to touch, which is both a quality and a blast-radius decision. `assumptions` makes the model's gap-filling **visible** — the most dangerous spec is the one that silently invented a requirement. The plan prompt's job is to produce one of these, fully populated, every time. This is where the month's central question lands: you will spend Lab 1 not asking "how do I prompt the model to build features" but "**what spec format makes features trivial to build**" — because once PLAN reliably emits a compilable `Spec`, BUILD's job collapses from "be clever" to "honor this."

### §4 — The pipeline: stages with their own prompt, tools, and evals

A factory is a **pipeline** of stages, and the design rule that keeps it sane is that **each stage is a small, single-purpose agent with three things of its own: a prompt, a tool stack, and an eval.** This is your Month 9 worker pattern in a *fixed* arrangement — instead of a lead dynamically decomposing, the decomposition is the pipeline itself, baked in. The six stages and their distinct jobs:

1. **PLAN** — fuzzy request → validated `Spec` (§3). Tools: none (pure reasoning). Eval: does the output parse as a `Spec` with non-empty acceptance criteria?
2. **SCOUT** — read the target codebase; find the real touchpoints, prior art, and conventions (§5). Tools: read-only `read_file`, `grep`, `list_dir`. Eval: did it return concrete file paths that exist?
3. **BUILD** — implement the spec, honoring Scout's conventions (§6). Tools: `read_file`, `write_file` (jailed to the repo). Eval: does the working tree still import / parse?
4. **VALIDATE** — static analysis (§7). Tools: `ruff check`, `mypy`. Eval: both exit `0`.
5. **TEST** — run existing tests, generate new tests from acceptance criteria, enforce coverage (§7). Tools: `write_file` (tests only), `pytest --cov`. Eval: tests pass and coverage ≥ threshold.
6. **REVIEW** — an independent agent checks the diff against the spec before merge (§8). Tools: `git diff`, `read_file`. Eval: verdict `approve`, with cited reasons; a `reject` fails the run.

```
request ─▶ PLAN ─▶ SCOUT ─▶ BUILD ─▶ VALIDATE ─▶ TEST ─▶ REVIEW ─▶ PR
            spec    map      code     ruff/mypy   pytest  diff vs spec
            (creative)      (boring/deterministic)  (real tools)  (independent)
```

A stage that fails its eval does not get to pass its output downstream. It either **re-runs** (with the failure fed back as context — "mypy says line 12 is untyped, fix it") up to a small retry budget, or it **fails the whole run loudly** with the stage and reason recorded. This is the factory analog of "fail the build" — and it is what makes "first try, zero human edits" mean something: the human never patches a stage's output, the *stage* fixes itself or the run is declared a failure to be debugged at the spec level.

### §5 — SCOUT: read before you build

The stage most beginners skip, and the one that most determines whether BUILD produces code that *fits*, is **SCOUT.** Its job is to read the target codebase *before* a line is written and answer three questions: **where** does this feature touch (the real files, refining PLAN's guess), **what prior art** exists (is there already a cache helper, an auth dependency, a response model to reuse?), and **what are the conventions** (does this repo use `async def` routes, Pydantic response models, a particular error pattern?). SCOUT exists because a model with no codebase context writes plausible code that violates every local convention — it imports a library the repo doesn't use, re-implements a helper that already exists, returns a dict where every other route returns a model. That code "works" and still fails review.

SCOUT is read-only by design — it touches `read_file`, `grep`, and `list_dir` and **nothing that writes.** Its output is a small structured **conventions brief** that becomes part of BUILD's context: the files to edit, the patterns to follow, the helpers to reuse. This is Month 9's context engineering applied to code: BUILD does not get the whole repo dumped into its context (which would degrade quality via lost-in-the-middle); it gets PLAN's spec plus SCOUT's tight brief — exactly the few files and conventions that matter. A good SCOUT brief is what turns BUILD from a creative act into a boring one.

### §6 — Determinism vs. creativity: planning is creative, building is boring

A factory mixes two opposite temperaments, and assigning them to the right stages is a core design decision. **PLAN is allowed to be creative** — it interprets a fuzzy request, fills gaps, proposes a reasonable shape. You *want* some divergent thinking here; this is the one place where the model earns its keep by being clever. **BUILD should be boring and predictable** — given a complete spec and a conventions brief, implementing it should be a near-mechanical translation, not an invention. The instant BUILD gets creative, it starts adding features you didn't ask for, choosing libraries the repo doesn't use, and "improving" things the spec didn't mention — and every one of those is a way the PR fails the "zero edits, on spec" bar.

> **Common misconception.** Building is the creative, interesting part — you want the model to be smart and inventive when it writes the code.
> **Reality.** Building should be *boring and deterministic*; planning is where creativity belongs. A creative BUILD adds unrequested features, swaps in libraries the repo doesn't use, and "improves" things the spec never mentioned — every one of which fails the "on spec, zero edits" bar. You front-load the creativity into PLAN (interpret the fuzzy request, fill gaps) so that by the time you reach the stage that *writes to your repo*, there is nothing left to invent. The misconception is natural because writing code feels like the creative act — but in a factory the cleverness has already been spent upstream.

You control this temperament through three levers. **Temperature**: PLAN runs warm (room to interpret), BUILD runs cold (temperature 0 where the model supports it) so it sticks to the spec. **Prompt framing**: PLAN's prompt invites interpretation ("infer reasonable defaults and record them"); BUILD's prompt forbids it ("implement *exactly* the acceptance criteria; add nothing not in the spec; if the spec is ambiguous, fail and say what's missing"). **The spec itself** is the strongest lever: the more compilable the spec (§2), the less room BUILD has to improvise, the more boring and reliable it becomes. This is why the month's leverage lives in the spec format — a great spec makes the *most dangerous stage the least creative*, which is exactly the safety property you want from the stage that writes to your repo.

### §7 — VALIDATE and TEST: real tools, not model opinions

The stages that decide whether the produced code is *actually good* must run **real engineering tools**, not ask a model whether the code looks fine. This is non-negotiable, and it is what separates a factory from a clever prompt. **VALIDATE** runs `ruff check` (lint + format) and `mypy` (static types) as a hard gate — both must exit `0`. These are deterministic, fast, and unfoolable; a model can be talked into thinking buggy code is clean, but `mypy` cannot. **TEST** runs `pytest` against the existing suite (did the feature break anything?), generates *new* tests from the spec's acceptance criteria (does the feature do what was specified?), and enforces a **coverage threshold** so the generated tests actually exercise the new code.

```python
# VALIDATE and TEST are subprocess calls to real tools (Month 8's safe runner), not model calls.
def validate_stage(repo: Path) -> StageResult:
    ruff = run(["uv", "run", "ruff", "check", "."], cwd=repo)        # Month 8 allowlisted runner
    mypy = run(["uv", "run", "mypy", "."], cwd=repo)
    ok = ruff.returncode == 0 and mypy.returncode == 0
    return StageResult(ok=ok, detail=ruff.stdout + mypy.stdout)     # fed back to BUILD on failure

def test_stage(repo: Path) -> StageResult:
    res = run(["uv", "run", "pytest", "--cov", "--cov-fail-under=80", "-q"], cwd=repo)
    return StageResult(ok=res.returncode == 0, detail=res.stdout[-2000:])
```

The crucial pattern is the **feedback loop**: when VALIDATE fails, its output (`mypy`'s exact error) is fed back into BUILD's context for a retry, so the factory *self-corrects* the way a developer would — read the error, fix the line, re-run. A bounded retry budget (say, 3) keeps a stuck run from looping forever; exhausting it fails the run loudly. The model writes the code; the *tools* decide if it's acceptable. That division of labor — creative generation gated by deterministic verification — is the architecture of every reliable factory.

### §8 — REVIEW: an independent agent, because the builder cannot grade its own work

The last gate before a PR is **REVIEW**, and the one rule that makes it worth having is that it must be **independent of BUILD.** A reviewer that is the same agent, with the same context, that just wrote the code will rubber-stamp it — it already believes the code is correct, because it wrote it that way. REVIEW gets a *fresh* context: the `git diff` and the original `Spec`, and *not* BUILD's reasoning. Its job is adversarial in the productive sense — check the diff **against the spec**: does it satisfy every acceptance criterion? does it stay inside `out_of_scope`? did it honor the `constraints` (no new deps, schema unchanged)? did it sneak in changes the spec never asked for? A `reject` fails the run with cited reasons; an `approve` lets the PR through.

This is the same defense-in-depth posture as Month 9's validator, specialized for code. The validator there checked a worker's structured result; REVIEW here checks a code diff against the spec that authorized it. Using a *different model* than BUILD (or at least a different prompt and a cold, skeptical framing) gives you a genuine second opinion. The payoff is that REVIEW catches the failure mode VALIDATE and TEST can't: code that lints clean, types clean, and passes tests, but **does the wrong thing** — implements a feature the spec didn't ask for, or quietly violates a constraint. Tools verify the code is *well-formed*; REVIEW verifies it is *the right code*. Both gates exist because they catch different failures, and a factory that ships unreviewed diffs is one confident hallucination away from merging the wrong feature.

### §9 — Telemetry: every run is queryable and cost-per-artifact is reported

A factory you cannot measure is a factory you cannot trust, improve, or defend — so **telemetry is a first-class stage requirement, not an afterthought.** Every stage emits structured events (reusing Month 9's `RunTrace`): which stage, which model, how many tokens in/out, how long it took, whether its eval passed, and how many retries it burned. Written per run to `runs/<feature>/trace.jsonl`, this makes every run **queryable** — you can ask "which stage fails most often?", "which feature was most expensive?", "did REVIEW ever reject?" — and it is what lets you compute the metric that defines the whole pillar: **cost-per-produced-artifact.**

```python
trace.emit(stage="build", model="qwen2.5-coder:7b", served_by="ollama",
           tokens_in=3140, tokens_out=890, ms=8200, eval_ok=True, retries=1)
# ...later, the metrics table is just an aggregation over all runs:
# | feature | $/feature | time-to-PR | result   |
# | cache   | $0.00     | 94s        | success  |   (Ollama)
# | auth    | $0.013    | 71s        | success  |   (frontier model on BUILD)
```

Three metrics matter for the milestone: **dollars-per-feature** (sum of each stage's token cost; `$0.00` on Ollama, a few cents on a paid model — report whichever you ran), **time-to-PR** (wall-clock from request to PR), and **success rate** (first-try, zero-edit successes out of attempts). These are not vanity numbers; they are how you *engineer the factory*. If success rate is 3/5, the trace tells you which stage failed on the two misses, and §2's discipline tells you the fix is almost always a better spec, not a better prompt. The metrics table turns "I built a thing that works sometimes" into "here is a function with a known success rate and a known cost per output" — which is the difference between a demo and a factory.

## Labs

| Lab | Title | Time | Difficulty |
|---|---|---|---|
| [Lab 1](lab-1-the-plan-prompt-and-spec-driven-development.md) | The Plan Prompt and Spec-Driven Development | ~3.5 hrs | Core |
| [Lab 2](lab-2-build-the-six-stage-pipeline.md) | Build the Six-Stage Pipeline (Plan → Scout → Build → Validate → Test → Review) | ~5 hrs | Core |
| [Lab 3](lab-3-mini-feature-factory-milestone.md) | The Mini Feature Factory (Milestone) | ~6 hrs | Core / Stretch |

## Checkpoints & Self-Assessment

Run these against yourself at the end of each week. You are on track if you can do them without looking them up.

- **Week 1:** Explain "a feature is an output, a factory is a function" in one sentence, and why the factory's value compounds. Take a vague request ("make login better") and rewrite it as a *compilable* spec. State why "the spec is source, code is a compilation target" makes the "zero human edits" bar achievable. Show your `Spec` schema and point to which field makes TEST able to generate tests.
- **Week 2:** Name all six stages and, for each, its prompt's job, its tool stack, and its eval. Explain why SCOUT is read-only and why BUILD runs cold while PLAN runs warm. Show VALIDATE failing on a type error and feeding `mypy`'s output back into a BUILD retry. Point to where in the trace each stage's tokens and timing are recorded.
- **Week 3:** Run one feature end-to-end to a PR (`gh pr view`). Open `runs/<feature>/trace.jsonl` and read off the dollars-per-feature and time-to-PR. Explain why REVIEW must be independent of BUILD with a concrete example of what it catches that TEST can't.
- **Week 4:** Read your metrics table — what's the success rate, and for each failure, was the fix a better spec or a better stage? Recite the blast radius of the BUILD and TEST stages (write-jailed to the target repo). Deliver the two-minute defense: why this is a factory, not a fancy autocomplete.

## Reflect

Spend ten minutes on these in your learning log (writing, not just thinking):

- **Explain it back:** In two or three sentences, explain "a feature is an output; a factory is a function that produces outputs" to a peer who just finished Month 9 — and why that means you fix the spec, never the produced PR.
- **Connect:** Month 9 gave you a lead that decomposed a job into workers at runtime. How does this month's *fixed* six-stage pipeline change or extend that pattern, and what did you gain by freezing the decomposition?
- **Monitor:** Which idea this month is still fuzzy — determinism-vs-creativity per stage, why REVIEW must be independent, or how cost-per-artifact is computed from the trace? Name it precisely, and write the one question that would clear it up.

## Month-End Assessment

**Deliverable:** the **Mini Feature Factory** — a CLI that takes a **one-paragraph feature request** for a real Python web app (a scaffolded FastAPI app, or a small borrowed open-source project) and runs it through the **full six-stage pipeline** (PLAN → SCOUT → BUILD → VALIDATE → TEST → REVIEW), producing a **PR with code, tests, and a changelog entry**. The factory must succeed on **at least five different feature requests of varying complexity, on the first try, with zero human edits to the produced PR.** Every stage emits structured telemetry; every run is queryable. You submit: the **factory codebase**; the **five produced PRs** (linked from the README); a **metrics table** reporting dollars-per-feature, time-to-PR, and success rate; and a **`SPEC.md`** defining the plan-prompt / spec format you settled on. Done means you stop asking "how do I get the model to do X?" and start asking "**what spec format makes X trivial to produce?**"

**Rubric**

- **Passing:** The CLI accepts a one-paragraph request and runs all six stages, with PLAN producing a validated structured spec, SCOUT reading the repo, BUILD writing code jailed to the target repo, VALIDATE running `ruff` + `mypy` as a hard gate, TEST running `pytest` with a coverage threshold, and an **independent** REVIEW stage checking the diff against the spec. A successful run produces a real PR (branch + commit + `gh pr create`) containing code, tests, and a changelog entry. The factory succeeds on **at least five** varied feature requests **first-try with zero human edits to the PR**. Each run writes a `trace.jsonl`; a **metrics table** reports dollars-per-feature, time-to-PR, and success rate; the whole milestone is **$0-completable on local Ollama** (cost shown either way). A `SPEC.md` defines the spec format. Model fallback (Month 7) is wired per stage.
- **Excellent:** All of the above, plus: the five features span a **real complexity gradient** (a one-line change, a new endpoint, a cross-cutting concern) and **all five pass first-try**; the spec format is documented with a **rationale** for each field and at least one **iteration recorded** (a feature that failed, the spec-format change that fixed it — not a prompt tweak); VALIDATE/TEST failures **feed back** into a bounded BUILD retry loop visible in the trace; REVIEW uses a **different model/prompt** than BUILD and the README cites a concrete spec-violation it caught that TEST passed; telemetry is genuinely **queryable** (a small script answers "which stage fails most" / "cost per feature"); and a `RETRO.md` argues, from the metrics, that the leverage came from the **spec format**, not the prompts. The cost story is honest: dollars-per-feature and success rate are reported for whichever model ran each stage, making the cost/quality tradeoff explicit.

The real definition of done is behavioral: **you have stopped trying to make the model build a feature and started designing the spec format that makes a whole class of features trivial to produce.** If you find yourself hand-editing a produced PR, the factory isn't done — the spec or a stage is.

## Common Pitfalls

- **Building features, not the factory.** If you find yourself prompting for one feature and pasting the result, you've reverted to producing outputs. The deliverable is the *function*, proven on five inputs — not five hand-built features.
- **Hand-editing the produced PR.** The instant you patch the output by hand, "zero human edits" is broken and you're back on the assembly line. If the PR is wrong, fix the *spec* or the *stage* and re-run. This rule is the whole discipline.
- **A spec that won't compile.** "Make it faster" gives the factory nothing to honor. A good spec names touchpoints, testable acceptance criteria, and explicit out-of-scope. If BUILD has to invent requirements, your spec format is too loose.
- **Skipping SCOUT.** BUILD with no codebase context writes plausible code that violates every local convention — wrong imports, re-implemented helpers, wrong return types. It passes nothing. SCOUT is what makes BUILD boring.
- **Letting BUILD be creative.** A warm, "improve things" BUILD adds unrequested features and swaps libraries, failing the "on spec" bar. Run BUILD cold, forbid invention in its prompt, and lean on a compilable spec.
- **VALIDATE/TEST as model opinions.** Asking a model "is this code good?" is not validation. Run `ruff`, `mypy`, and `pytest` as real subprocesses with exit-code gates. The tools decide, not the model.
- **A REVIEW that's really the builder.** A reviewer sharing BUILD's context and conviction rubber-stamps everything. REVIEW gets a fresh context (diff + spec only), ideally a different model, and an adversarial prompt.
- **No telemetry, so no factory.** Without per-stage logs you cannot compute dollars-per-feature or success rate, cannot find which stage fails, and cannot prove the spec format (not luck) drives success. Instrument every stage from day one.

## Knowledge Check

Answer from memory first, then check. Questions marked ⟲ are spaced callbacks to earlier months — they are supposed to feel like a stretch.

1. State the month's thesis in one sentence, and explain why a factory's value compounds while a feature's value is linear.
2. A request says "make the products endpoint faster." Why can't this spec "compile," and what is the smallest set of additions that would make it compilable?
3. Name the six stages in order and, for each, give its one-job and its eval (what the stage must prove before its output passes downstream).
4. Predict the result: BUILD runs warm (temperature 0.9) with a vague spec. What two failure modes show up, and which downstream stage catches each?
5. Spot the risk: a teammate sets REVIEW to use the *same* model and context as BUILD to "save tokens." What goes wrong, and what does REVIEW then fail to catch that VALIDATE and TEST also miss?
6. Why are VALIDATE and TEST `subprocess` calls to real tools rather than model calls? What property do `ruff`/`mypy`/`pytest` have that a model judging "is this code good?" does not?
7. Your factory hits 3/5 first-try. The trace shows two failures at the TEST stage on vague acceptance criteria. Per this month's discipline, what do you fix — the prompt, the produced code, or the spec format — and why?
8. From a `trace.jsonl`, which three metrics define the milestone, and how is dollars-per-feature computed for an all-Ollama run?
9. ⟲ (Month 5) VALIDATE gates on `ruff` and `mypy` exiting `0`. What distinct class of defect does each catch, and why isn't one enough?
10. ⟲ (Month 6) PLAN emits a Pydantic `Spec` rather than prose. Tie this to what you learned about structured outputs and evals — why does structure make the PLAN eval trivial?
11. ⟲ (Month 9) The pipeline reuses your `RunTrace` and the worker-as-subprocess pattern. In one sentence, how is a pipeline related to the lead/worker/validator harness?
12. Why must BUILD's writes be jailed to the target repo, and which earlier month gave you that jail?

<details><summary>Answer key</summary>

1. Stop building features; build the system that builds features. A feature is produced once (linear value); a factory produces every feature thereafter at near-zero marginal cost (compounding value).
2. "Faster" names no touchpoint, no testable behavior, and no bound — BUILD would have to invent the requirement. Make it compilable by adding a touchpoint, a measurable acceptance criterion (e.g. "a second request within 60s does not hit the data layer"), and out-of-scope/constraints.
3. PLAN (fuzzy→`Spec`; eval: parses as a `Spec` with non-empty acceptance criteria) → SCOUT (read repo for touchpoints/conventions; eval: returns real file paths) → BUILD (implement honoring conventions; eval: tree still parses) → VALIDATE (`ruff`+`mypy`; eval: both exit 0) → TEST (`pytest --cov`; eval: pass + coverage ≥ floor) → REVIEW (diff vs spec; eval: verdict `approve`).
4. A warm BUILD on a vague spec (a) invents unrequested behavior — caught by REVIEW (out-of-scope / not in acceptance_criteria), and (b) produces inconsistent code across runs / type or style slips — caught by VALIDATE (`ruff`/`mypy`). TEST may also fail if behavior doesn't match the criteria.
5. A reviewer sharing BUILD's context and conviction rubber-stamps the diff. It fails to catch code that lints clean, types clean, and passes tests but **does the wrong thing** — implements an unrequested feature or violates a constraint. REVIEW needs a fresh context (diff + spec only), ideally a different model.
6. They are deterministic and unfoolable — a model can be talked into believing buggy code is clean, but `mypy` cannot. Real tools gate on exit codes, giving a true/false signal instead of an opinion.
7. Fix the **spec format** (e.g. require each acceptance criterion to name an observable). Patching the code voids "zero human edits"; tweaking the prompt rarely fixes a whole class. A spec-format rule fixes every "vague verb" feature at once. (§2, §9.)
8. Dollars-per-feature, time-to-PR, and first-try success rate. Dollars-per-feature = sum over stages of (tokens × per-token price); on Ollama the price is `$0`, so it's `$0.00` — but you still report it.
9. ⟲ `ruff` catches lint/format/style and obvious errors; `mypy` catches type mismatches (wrong argument types, missing returns) before runtime. They catch disjoint defect classes, so passing one says nothing about the other. (Month 5.)
10. ⟲ A structured output gives named fields to assert on, so the eval is a deterministic check ("is `acceptance_criteria` non-empty / do criteria parse?") instead of a fuzzy read of prose — the same structured-output + eval discipline from Month 6.
11. ⟲ A pipeline is a lead/worker/validator harness with the decomposition *frozen*: the six stages are the fixed dec, and the per-run trace is the same `RunTrace`. (Month 9.)
12. So a buggy or adversarial edit cannot write outside the workpiece — bounding blast radius. The path-resolution jail and allowlisted `subprocess` runner came from Month 8.
</details>

## Further Reading

- [Anthropic — "Building effective agents"](https://www.anthropic.com/research/building-effective-agents) — the prompt-chaining / pipeline pattern (each stage's output feeds the next, with programmatic gates between) is exactly the factory's spine.
- [GitHub — "spec-driven development" (spec-kit)](https://github.com/github/spec-kit) — a concrete take on treating the spec as the primary artifact and generating code from it; useful counterpoint to your own `SPEC.md` design.
- [Ruff documentation](https://docs.astral.sh/ruff/) and [mypy documentation](https://mypy.readthedocs.io/) — the real static-analysis tools the VALIDATE stage gates on; read what they actually check so you can wire them as hard gates.
- [pytest — coverage and `pytest-cov`](https://pytest-cov.readthedocs.io/) — running existing tests, generating new ones, and enforcing a coverage threshold in the TEST stage.
- [GitHub CLI — `gh pr create`](https://cli.github.com/manual/gh_pr_create) — turning a passing pipeline run into the artifact the factory exists to produce.
- [Martin Fowler — "Specification by Example"](https://martinfowler.com/bliki/SpecificationByExample.html) — why acceptance criteria phrased as testable examples are the bridge between a spec and a generated test suite (§3, §7).

## Author's Notes

This month operationalizes Pillar 2's thesis — build the system that builds the system — by making the learner construct a real six-stage pipeline rather than read about one. Three calibration decisions worth naming. First, **scope of the target app**: the milestone builds features *into* a small scaffolded FastAPI app rather than a large open-source project, because a sprawling codebase makes SCOUT and "zero human edits" dominated by repo-archaeology rather than factory design; the stretch goal points learners at a borrowed OSS project once the factory works. Second, **the free-vs-paid honesty**: the spec demands the month be $0-completable, and it is on local Ollama, but I refuse to pretend a 7B model clears five varied features first-try as reliably as a frontier model — so the Free-LLM mandate is honored by making the pipeline run end-to-end on Ollama while telemetry forces the learner to *report* dollars-per-feature and success rate on whichever model they ran, making the cost/quality tradeoff explicit instead of hidden. Third, **reuse over reinvention**: VALIDATE/TEST deliberately call Month 8's allowlisted `subprocess` runner and the whole pipeline reuses Month 9's `RunTrace` and worker-as-subprocess patterns, so the new conceptual load is the *spec format and stage design*, not new infrastructure. The unresolved tension I'm flagging for the convergence pass: "first try, zero human edits" is a genuinely hard bar on free models, and some learners will hit 3–4/5 locally; the rubric treats 5/5 as the Excellent tier and a working factory with honest metrics as Passing, which keeps the month achievable on $0 without lowering the conceptual bar.
