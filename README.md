# ExecPlan Template for Codex

AGENTS.md and ExecPlan markdown templates for programming efficiently with AI coding assistants.

**"ExecPlan"** becomes shared vocabulary between you and your agent—a code word for task-scoped work plans that structure how features get built.

**Inspired by [Aaron Friel](https://x.com/AaronFriel)'s OpenAI 2025 DevDay presentation:** ["Shipping with Codex"](https://youtu.be/Gr41tYOzE20?t=1050)

Maintained by [Aditya Advani](https://x.com/aditya_advani)

---

## The Workflow

1. **Create `.agent/template/PLAN.md`** — a structured markdown template for feature planning
2. **Configure `AGENTS.md`** — instructs your AI agent to create a new ExecPlan for each task by copying the template into a new feature folder (e.g., `.agent/orchestrator/PLAN.md`)
3. **Say "do the plan"** — the agent executes methodically, usually one-shotting or completing in 2-3 iterations

### Why It Works

- **Structured thinking:** Forces the agent (and you) to clarify purpose, context, and steps upfront
- **Living document:** Progress, decisions, and surprises stay in the plan
- **Clean workspace:** Each feature gets its own folder; option to delete when merged
- **Repeatability:** The template encodes best practices—TDD, decision logs, verification steps

---

## Quick Start

1. Copy this repo's `.agent/template/PLAN.md` and `AGENTS.md` into your project
2. **(Optional)** Add to `.gitignore` to keep plans local:
   ```
   .agent/*
   !.agent/template/
   ```
3. Tell your agent: "Create an ExecPlan for [feature] following the template"
4. Execute: "Do the plan"

---

## Usage Notes

- **One ExecPlan = One Task:** Each plan maps to a task (feature, refactor, bug fix, etc), sometimes with subtasks tracked as well in the Progress section
- **TDD first:** `AGENTS.md` includes test-driven development guidance
- **Concurrency:** Running multiple plans in parallel is possible but requires orchestration (still figuring this out myself)
- **Cleanup:** Delete feature folders post-merge if working in a shared codebase

---

## Contributing

If you've adapted this workflow or have insights on running multiple concurrent agents (especially in Codex web or as codex cli background jobs in sandboxes), please share!

---

## License

MIT
