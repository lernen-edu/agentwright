---
title: Curriculum
nav_order: 10
permalink: /curriculum/
---

# Agentwright — A 12-Month Curriculum

A complete, lab-driven course that takes someone with **zero coding experience** to **deploying always-on, value-generating AI systems**. The destination is the ability to build *the system that builds the system*: custom agent harnesses, software factories that produce results on spec, code that survives the churn of changing AI models, agents with safe programmatic access to the world, and agents that run 24/7 and generate value while you sleep.

- **Pace:** ~8–12 hours/week (evenings + weekends). Compress to 8 months at 15–20 hrs/week, or 5–6 months full-time.
- **Cost:** designed to be completable on **$0**. Every AI month uses local models via [Ollama](https://ollama.com) for free, and labels the optional paid path (Anthropic/OpenAI) with its actual dollar cost.
- **Platform:** everything runs on **macOS** (Apple Silicon and Intel), using free, open tools.

---

## How the course is built

This curriculum was authored and reviewed by five expert personas, and every month is checked through all five lenses before shipping:

1. **The Curriculum Architect** — pedagogy, measurable objectives, cognitive load, assessment.
2. **The Foundations Engineer** — CLI, Git, HTTP, Python, software-engineering correctness.
3. **The Agentic Systems Engineer** — agent loops, harnesses, factories, extensible design.
4. **The Infra / Security / DevOps Engineer** — sandboxing, secrets, deployment, cost, blast radius.
5. **The Lab Designer** — step-by-step labs, checkpoints, verification, troubleshooting.

Each month is its own folder with a `README.md` (the full course) and 3–4 hands-on lab files. The work products chain: the agent you hand-build in Month 6 is refactored, secured, orchestrated, and deployed across the pillar months, ending in a single integrated capstone system.

### How each month is designed to be learned

Every month is built on eight evidence-based learning principles. Concretely, in each month you will find:

- **A Warm-Up that makes you retrieve before you receive** — a few from-memory questions that pull forward the prior months a topic builds on (retrieval practice + spaced review).
- **New skills taught as worked → faded → independent** — you first study a complete worked example, then fill in a faded one, then solve a fresh one alone (the worked-example effect / gradual release).
- **Diagrams paired with the words** — 68 Mermaid diagrams across the course turn loops, pipelines, and architectures into pictures (dual coding).
- **Misconception callouts** that name the wrong belief and correct it before it sets.
- **A recovery path on every checkpoint** — each "you should see X" is followed by "if not, here's why and the fix" (formative feedback).
- **Reflection prompts** that ask you to explain ideas back and monitor your own understanding (metacognition).
- **An end-of-month Knowledge Check** that interleaves the new material with spaced callbacks (marked ⟲) to earlier months.

Difficulty ramps with you: the hands-on labs read at roughly an 8th-grade level, and the conceptual reading climbs only as your sophistication does.

---

## The arc

| Phase | Months | Theme |
|---|---|---|
| Foundations | 1–5 | Command line, Git, HTTP, Python, software engineering |
| The first agent | 6 | AI APIs and the hand-built agent loop |
| Pillar 3 | 7 | Extensible Software |
| Pillar 5 | 8 | Agentic Access (and security) |
| Pillar 1 | 9 | Agent Harnesses |
| Pillar 2 | 10 | Software Factories |
| Pillar 4 | 11 | Always On Agents |
| Capstone | 12 | Integrate all five pillars into a deployed system |

The pillars are intentionally out of numerical order: **Extensible Software (3)** and **Agentic Access (5)** come before **Harnesses (1)** and **Factories (2)**, because a harness built on brittle code with unsafe tool access is worse than no harness. **Always On Agents (4)** is last because it is the deployment phase.

---

## Months

### Foundations

**[Month 1 — Command Line & Git](month-01-command-line-and-git/README.md)** · *Foundations*
How computers actually work: the macOS/Unix shell, terminal editing, Git, and the GitHub workflow — typed by hand, with AI assistance switched off.
Milestone: a `dotfiles` repo with a `bin/` of shell scripts and 25+ atomic commits.
Labs: [Terminal & filesystem](month-01-command-line-and-git/lab-1-terminal-and-filesystem-navigation.md) · [Shell scripting](month-01-command-line-and-git/lab-2-shell-scripting-build-the-bin-directory.md) · [Git & GitHub](month-01-command-line-and-git/lab-3-git-and-github-workflow.md) · [Vim survival](month-01-command-line-and-git/lab-4-vim-survival.md)

**[Month 2 — HTTP & JSON](month-02-http-and-json/README.md)** · *Foundations*
The web's plumbing: the HTTP request/response model, status codes, headers, JSON literacy, `jq`, auth concepts, and reading API docs — all by hand, no code yet.
Milestone: an "API Explorer's Notebook" documenting three real public APIs.
Labs: [HTTP anatomy](month-02-http-and-json/lab-1-http-anatomy-with-curl-and-httpie.md) · [JSON & jq](month-02-http-and-json/lab-2-json-literacy-and-jq-slicing.md) · [Explore real APIs](month-02-http-and-json/lab-3-explore-real-apis-and-build-the-notebook.md)

**[Month 3 — Python Fluency](month-03-python-fluency/README.md)** · *Foundations*
From running commands to writing programs: Python's standard library, data structures, file I/O, JSON/CSV, error handling, and project management with `uv`.
Milestone: a "Toolbelt" of packaged Python CLI utilities.
Labs: [Language core](month-03-python-fluency/lab-1-language-core-and-data-structures.md) · [Files, JSON, CSV & errors](month-03-python-fluency/lab-2-files-json-csv-and-errors.md) · [Build the Toolbelt CLI](month-03-python-fluency/lab-3-build-the-toolbelt-cli.md)

**[Month 4 — Python & APIs](month-04-python-and-apis/README.md)** · *Foundations*
Where HTTP and Python fuse: `requests`, `.env` secrets, hand-rolled retries with backoff and jitter, rate limits, and pagination.
Milestone: "GitHub Pulse," an installable CLI that reports a user's activity.
Labs: [requests basics](month-04-python-and-apis/lab-1-requests-get-post-headers-and-auth.md) · [Resilience](month-04-python-and-apis/lab-2-resilience-timeouts-retries-and-rate-limits.md) · [Build & package GitHub Pulse](month-04-python-and-apis/lab-3-build-and-package-github-pulse.md)

**[Month 5 — Software Engineering Principles](month-05-software-engineering-principles/README.md)** · *Foundations*
What separates a scripter from an engineer: OOP, Protocols/interfaces, dependency injection, SOLID (with deep focus on Open/Closed and Single Responsibility), pytest, strict types, and structured logging.
Milestone: the "Refactor Crucible" — rebuild GitHub Pulse with pluggable providers and 80%+ test coverage.
Labs: [OOP, interfaces & DI](month-05-software-engineering-principles/lab-1-oop-interfaces-and-dependency-injection.md) · [pytest & strict types](month-05-software-engineering-principles/lab-2-pytest-deep-dive-and-strict-types.md) · [Refactor Crucible](month-05-software-engineering-principles/lab-3-refactor-crucible-pluggable-providers.md)

### The first agent

**[Month 6 — AI APIs & the First Agent Loop](month-06-ai-apis-and-the-first-agent-loop/README.md)** · *Bridge to the pillars*
The mystery, removed: an agent is a `while` loop around an API call. The Messages API, tokens and cost math, prompt engineering, streaming, tool use — all hand-built, no frameworks.
Milestone: a single-file "From-Scratch Agent" with file/shell tools, a working-directory jail, and JSONL tracing.
Labs: [First call & cost math](month-06-ai-apis-and-the-first-agent-loop/lab-1-first-model-call-and-cost-math.md) · [Prompting, structured output & evals](month-06-ai-apis-and-the-first-agent-loop/lab-2-prompting-structured-output-and-evals.md) · [Tool use by hand](month-06-ai-apis-and-the-first-agent-loop/lab-3-tool-use-by-hand.md) · [From-Scratch Agent](month-06-ai-apis-and-the-first-agent-loop/lab-4-from-scratch-agent.md)

### The Five Pillars

**[Month 7 — Extensible Software](month-07-extensible-software/README.md)** · *Pillar 3*
Software that survives model churn: a single `LLMClient` interface, Strategy/Registry/Plugin patterns, config-as-code, and fallback chains — "open to extension, closed to modification."
Milestone: a "Provider-Agnostic Core" where swapping models is a config change, with graceful failover to local Ollama.
Labs: [LLMClient Protocol & providers](month-07-extensible-software/lab-1-llmclient-protocol-and-providers.md) · [Registries, Strategy & prompt versioning](month-07-extensible-software/lab-2-registries-strategy-and-prompt-versioning.md) · [Provider-agnostic core & failover](month-07-extensible-software/lab-3-provider-agnostic-core-and-failover.md)

**[Month 8 — Agentic Access](month-08-agentic-access/README.md)** · *Pillar 5*
Safe hands in the outside world: `subprocess` done right, a minimal MCP server, webhooks with FastAPI, and the non-negotiable security guardrails (allowlists, jails, egress limits, read-only DB roles, human-in-the-loop gates).
Milestone: a "Safe-Hands Toolkit" with danger-rated tools, a containerized sandbox, and a documented threat model.
Labs: [Subprocess, allowlist & jail](month-08-agentic-access/lab-1-subprocess-cli-allowlist-and-jail-hardening.md) · [Minimal MCP server & client](month-08-agentic-access/lab-2-build-a-minimal-mcp-server-and-client.md) · [Webhooks & secrets hygiene](month-08-agentic-access/lab-3-webhooks-with-fastapi-and-secrets-hygiene.md) · [Safe-Hands Toolkit](month-08-agentic-access/lab-4-safe-hands-toolkit-milestone.md)

**[Month 9 — Agent Harnesses](month-09-agent-harnesses/README.md)** · *Pillar 1*
Custom, domain-specific agent environments beyond the default tools: context engineering, sub-agent delegation, team-lead/worker/validator orchestration, sandboxing, per-role model routing, and trace/replay.
Milestone: a "Triage Harness" with three roles, three recorded runs, and a postmortem.
Labs: [Harness anatomy & domain modeling](month-09-agent-harnesses/lab-1-harness-anatomy-and-domain-modeling.md) · [Sub-agent orchestration & model routing](month-09-agent-harnesses/lab-2-sub-agent-orchestration-and-model-routing.md) · [Triage Harness](month-09-agent-harnesses/lab-3-triage-harness-milestone.md)

**[Month 10 — Software Factories](month-10-software-factories/README.md)** · *Pillar 2*
Stop building features; build the system that builds features on spec. The plan prompt, and the six-stage pipeline: Plan → Scout → Build → Validate → Test → Review, with telemetry and cost-per-artifact.
Milestone: a "Mini Feature Factory" that turns a one-paragraph request into a finished PR, proven on five features.
Labs: [The plan prompt & spec-driven dev](month-10-software-factories/lab-1-the-plan-prompt-and-spec-driven-development.md) · [Build the six-stage pipeline](month-10-software-factories/lab-2-build-the-six-stage-pipeline.md) · [Mini Feature Factory](month-10-software-factories/lab-3-mini-feature-factory-milestone.md)

**[Month 11 — Always On Agents](month-11-always-on-agents/README.md)** · *Pillar 4*
Token arbitrage and AFK deployment: the token-economics ladder, scheduling with `launchd`/cron, durable state in SQLite, hard spend caps, circuit breakers, tested kill switches, and free 24/7 hosting.
Milestone: a "First AFK Agent" that runs unattended for 7 days with a runbook and a real incident log.
Labs: [Scheduling & durable state](month-11-always-on-agents/lab-1-scheduling-with-launchd-and-durable-state-in-sqlite.md) · [Safety rails](month-11-always-on-agents/lab-2-safety-rails-spend-cap-circuit-breaker-kill-switch-alerting.md) · [Deploy to a free cloud substrate](month-11-always-on-agents/lab-3-deploy-to-a-free-cloud-substrate-and-failure-semantics.md) · [First AFK Agent](month-11-always-on-agents/lab-4-first-afk-agent-milestone-seven-day-run.md)

### Capstone

**[Month 12 — The Integrated Agentic System](month-12-capstone/README.md)** · *Capstone — All Five Pillars*
Combine everything into one deployed, value-generating system, with a pillar-coverage matrix and a demo guide.
Milestone: the "AFK Value Generator" — a custom harness, maintained by a factory, with pluggable providers, secured access, deployed always-on, cost-tracked, and run unattended for 14 days.
Labs: [Kickoff: scope, architecture & spec](month-12-capstone/lab-1-capstone-kickoff-scope-architecture-and-spec.md) · [Build & integrate the five pillars](month-12-capstone/lab-2-build-and-integrate-the-five-pillars.md) · [Deploy, harden & retrospect](month-12-capstone/lab-3-deploy-harden-and-retrospect.md)

---

## Cross-cutting practices (every month)

- **A daily learning log** committed to a public repo — for forced reflection, not vanity.
- **A weekly demo**, even if only to a webcam — explain what you built, out loud.
- **One paper or substantial blog post per week** from the AI-engineering canon.
- **A `RUNBOOK.md`** for every deployed agent from Month 6 onward.
- **The assistance rule (from Month 6 on):** you may use AI to build, but never to skip understanding — you must be able to defend every line if asked.

---

## Getting started

1. Install the baseline macOS stack: Homebrew, `uv`, Git, `gh`, and Ollama — all free. The [Getting Started](../getting-started.md) page walks you through each one. Don't worry about installing everything up front; each month tells you what new tool it needs.
2. Open [Month 1](month-01-command-line-and-git/README.md) and work top to bottom: read the README, then do the labs in order.
3. Hit each month's **Month-End Assessment** before moving on — the milestones are load-bearing for later months.
