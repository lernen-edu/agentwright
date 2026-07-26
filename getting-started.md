---
title: Getting Started
nav_order: 2
permalink: /getting-started/
---

# Getting Started

What you need before Month 1 begins, the one-time setup, and the AI tools the tutor supports.

---

## You'll need

- A Mac (Apple Silicon or Intel) running macOS.
- A free [GitHub](https://github.com/) account.
- About 8–12 hours per week if you're following the 12-month pace. (Or 15–20 hrs/week for the 8-month compressed pace, or full-time for 5–6 months.)
- One AI coding tool installed — see the [supported tools](#supported-ai-tools) below. You can use any of them, including a $0 local-model setup.

You do **not** need to know how to code. The first five months teach you that.

---

## Clone the course

```bash
git clone https://github.com/lernen-edu/agentwright.git
cd agentwright
```

That's it — no setup script to run. Open the directory in your AI coding tool of choice (see below) and the tutor activates automatically.

---

## Install the macOS baseline

Every month uses the same free tooling. Install Homebrew first, then the rest in one go:

```bash
# Homebrew (if you don't have it):
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Core stack
brew install git gh uv httpie tmux
brew install --cask visual-studio-code

# Free LLM (used in every AI month)
brew install ollama
```

Don't worry about installing everything up front — each month tells you what new tool it needs.

---

## Supported AI tools

The Agentwright tutor activates automatically when you open this directory in any of these:

- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** — Anthropic's CLI. Free tier exists; the $20/mo Claude.ai plan also includes Claude Code. Reads `CLAUDE.md`.
- **[OpenAI Codex CLI](https://github.com/openai/codex)** — OpenAI's CLI for code. Reads `AGENTS.md`.
- **[Gemini CLI](https://github.com/google-gemini/gemini-cli)** — Google's CLI. Free tier available. Reads `GEMINI.md`.
- **[Opencode](https://opencode.ai/)** — open-source coding agent. Reads `AGENTS.md`.
- **Pi Coding Agent** — reads `AGENTS.md`.

You can also bring your own local model setup via Ollama and a wrapper tool. The tutor instructions live in plain markdown files at the root of this repo, so any tool that reads agent-context conventions will pick them up.

---

## What the tutor will and won't do

Read [How the Tutor Works](./tutor-reference.md) before you start. The short version: it's Socratic by default. It asks you questions, points you at course sections, and gives hints with a gap. It will not write your lab code or run your commands. That is the point.

If you ever need a quick factual answer, say "answer me directly" — you'll get one direct answer, then the tutor returns to Socratic mode.

---

## Track your progress

The tutor reads `.tutor/progress.md` at the start of every session to calibrate how much scaffolding to give you. Update it as you move through the course. The lab work is yours; the progress log is also yours.

---

## When you're ready

Open [the curriculum index](./curriculum/) and start at Month 1, Lab 1. Work top to bottom. Hit each month's Month-End Assessment before moving on.

Good luck.
