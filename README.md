<!--
This README is what students see when they land on github.com/lernen-edu/agentwright.
Keep it short and action-oriented. The Pages site at lernen-edu.github.io/agentwright
is where the rich nav and reading experience lives.
-->

# Agentwright

A 12-month, lab-driven agentic-engineering course from zero coding experience to deploying always-on, value-generating AI systems. Free on macOS. Free models supported throughout. Authored by [lernen-edu](https://github.com/lernen-edu).

**Site:** <https://lernen-edu.github.io/agentwright/>

---

## Start here

```bash
git clone https://github.com/lernen-edu/agentwright.git
cd agentwright
```

Then open this directory in **Claude Code**, **Codex CLI**, **Gemini CLI**, **Opencode**, or **Pi Coding Agent**. The Socratic tutor activates automatically.

Begin at [`curriculum/README.md`](./curriculum/) — Month 1, Lab 1.

---

## How this course is different

- **Built on eight evidence-based learning principles.** Retrieval before reception. Worked examples that fade into independence. Spaced callbacks across months.
- **Authored through five expert lenses.** Curriculum architect, foundations engineer, agentic systems engineer, infra/security/devops, lab designer. Every month passes all five.
- **The tutor is Socratic.** It will not write your lab code. It will not run your commands. It will not paste-debug your errors. It asks back, points to course sections, and gives hints with a gap. See [Tutor reference](./tutor-reference.md) for the full behavior contract.
- **Free.** Local models via Ollama, free hosted tiers, paid APIs labeled with their dollar cost. The whole course is completable on $0.

---

## What's in this repo

```
.
├── _config.yml               Just the Docs config (GitHub Pages site)
├── index.md                  Site landing page
├── README.md                 You are here
├── getting-started.md        Setup and supported AI tools
├── tutor-reference.md        What the tutor will and won't do
├── CLAUDE.md                 Tutor entry-point for Claude Code
├── AGENTS.md                 Tutor entry-point for Codex / Opencode / Pi
├── GEMINI.md                 Tutor entry-point for Gemini CLI
├── .tutor/
│   ├── tutor-core.md         Shared tutor behavior (source of truth)
│   └── progress.md           Your progress through the course (you update this)
├── curriculum/               12 months of the course
│   ├── README.md             Master curriculum index
│   ├── month-01-.../         Each month: README + labs
│   └── ...
├── LICENSE-CONTENT           CC BY 4.0 (curriculum prose)
└── LICENSE-CODE              MIT (tutor files, hooks, scripts, Jekyll config)
```

---

## License

Two licenses, because the work is two things:

- **Course content** (everything under `curriculum/`, `index.md`, `getting-started.md`, `tutor-reference.md`) is licensed under **Creative Commons Attribution 4.0** — see [`LICENSE-CONTENT`](./LICENSE-CONTENT). You may remix, share, and build on it, including commercially, with attribution.
- **Code components** (the tutor files, hooks, scripts, Jekyll config) are licensed under the **MIT License** — see [`LICENSE-CODE`](./LICENSE-CODE).

When in doubt, the file's intent governs: prose under CC BY 4.0, code under MIT.

---

## A note on AI assistance

This course teaches you to use AI well. The tutor is intentionally Socratic — it refuses to write your code. That is the point. You will defend every line you commit. See [Tutor reference](./tutor-reference.md) for the full contract.
