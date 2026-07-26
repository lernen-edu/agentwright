---
title: How the Tutor Works
nav_order: 3
permalink: /tutor-reference/
---

# How the Tutor Works

When you open the Agentwright repository in a supported AI coding tool, the tool reads the tutor instructions in this repo and switches into Socratic mode. Here's what that means in practice, so nothing about it is surprising.

---

## The stance

The tutor's first response to "help me with this lab" is **a question**, **a hint with a gap**, or **a pointer to a course section**. Not a solution.

You do the typing. You do the running. You do the diagnosing. The tutor's job is to keep you on the productive path and surface what you almost saw on your own.

This matches the course's own [Assistance Rule](./curriculum/) from Month 6 onward: AI may be used to build, but never to skip understanding. You must be able to defend every line if asked.

---

## The four-tier interaction ladder

In order of preference, the tutor will:

1. **Question back.** "What error did you actually see?" "What's the smallest version of this you could test?" "Which Month 3 concept does this look like?"
2. **Point to a course section.** "Re-read the misconception callout in Month 5 about dependency injection — does that apply here?" You open the file.
3. **Hint with a gap.** "The fix involves changing how `requests` handles a `429`. Which response header tells you how long to wait?"
4. **Worked walkthrough (last resort).** Only after several genuine attempts, and only on the *concept* using a different example — not on your code. Then asks you to apply it.

---

## What the tutor will refuse

- "Write the lab for me." → No.
- "Generate the test stubs." → No.
- "Just give me the working code." → No.
- "Paste the answer to the Knowledge Check." → No — those exist so you retrieve from memory.
- "Run these commands for me." → No — you type them.

The refusals are brief and warm, then the tutor offers the Socratic alternative.

---

## The opt-out, for quick facts

When you genuinely need a fact, not a lesson:

- "Just answer me directly."
- "Skip the Socratic thing this once."
- "I need a quick fact, not a lesson."

You'll get **one direct answer**, then on the next turn the tutor returns to Socratic mode. Legitimate uses:

- "What's the curl flag to follow redirects?" → `curl -L`.
- "Is HTTPie free?" → Yes.
- "Does the course recommend Colima or Docker Desktop?" → Colima.

The opt-out does not cover lab solutions. "Just answer me directly, what's the code for lab 2?" remains off-limits.

---

## How the tutor calibrates to where you are

The tutor adjusts its level of scaffolding based on which month you're on. You don't have to maintain anything for this to work — the tutor will ask you on a new session ("what month and lab are you on?") and remember between sessions in a small file it maintains itself (`.tutor/progress.md`). If you ever want to jump, just tell it in conversation: *"let's jump to Month 6, Lab 2"*, and the tutor will pick up there.

The calibration ramp, roughly:

- **Months 1–3.** Slow, lots of scaffolding. Jargon defined the first time. Frequent "we covered this in Month X" callbacks.
- **Months 4–5.** Real code. Expect to read tracebacks. Push to write a tiny test before a fix.
- **Month 6.** The Assistance Rule activates. You must explain every line.
- **Months 7–8.** Treated as a junior engineer. Security guardrails surface first whenever your code touches shell, filesystem, network, secrets, or a database.
- **Months 9–11.** Pushed back like a senior engineer. "Why didn't you delegate that to a sub-agent?" "What's your spend cap?" "What's the kill switch?"
- **Month 12.** Review and challenge. The tutor uses the course's five expert lenses to critique your capstone.

Be honest when the tutor asks where you are. Telling it Month 9 when you're really Month 3 means you'll get less help than you need.

---

## Bring any model

The tutor instructions live in plain markdown files at the root of the repo (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, with the shared body in `.tutor/tutor-core.md`). Any AI coding tool that reads agent-context conventions will pick them up. That means:

- **Claude Code** (free or paid) — works.
- **OpenAI Codex CLI** — works.
- **Gemini CLI** (free tier or paid) — works.
- **Opencode + Ollama (a fully local, $0 setup)** — works.
- **Pi Coding Agent** — works.

The Socratic stance doesn't require a frontier model. Any model that can read these instructions and follow them is enough.

---

## If the tutor misbehaves

If the AI starts writing your code, paste-debugging, or otherwise breaking the Socratic stance:

1. Remind it: "Please re-read `.tutor/tutor-core.md` and stay Socratic."
2. If that doesn't work, close the session and reopen the repo. A fresh load of the agent-context files usually fixes it.
3. If a particular tool consistently misbehaves, open an issue on the course repo — the tutor instructions can be tightened.

You can also at any time say "answer me directly" for a one-shot direct answer if the Socratic mode is getting in the way of a legitimate quick lookup.
