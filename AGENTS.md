# AGENTS.md

Instructions for AI coding agents (Claude Code, Codex, etc.) working in this repository.

## Project Overview

This repo is the canonical source for agent workflow guidelines and the ExecPlan template.

---

## Test driven development

Almost always write tests first, and have me approve them. Unit tests like that, integration tests ask my permission but we generally want those also unless a service being tested has side effects that are unsafe (I can help decide this).

---

## Workflow: Beads + Exec Plans

This project uses two coordinated systems:

| System | Purpose | Source of Truth For |
|--------|---------|---------------------|
| **Beads** (`bd` CLI) | Task queue | What to do, status, dependencies |
| **Exec Plans** (`.agents/plans/`) | Living documentation | How to do it, progress, decisions |

**Detailed beads reference**: See `.agents/beads.md` for comprehensive bd CLI usage.

### Starting Work

1. **Pick a task:**
   ```bash
   bd ready # Show unblocked tasks (skip P4 backlog)
   bd show <issue-id>          # Review details
   bd update <id> --status in_progress
   ```

2. **Create an exec plan:**
   - Copy `.agents/template/PLAN.md` to `.agents/plans/<issue-id>/PLAN.md`
   - Fill in Purpose, Context, Plan of Work and all sections
   - Use mermaid diagrams to clarify flow or scope if helpful
   - If issue has label `needs-plan-review`: **STOP and wait for human approval**
   - Otherwise: proceed to execution

3. **Work:**
   - Update Progress section in PLAN.md as you complete steps
   - Log decisions and surprises in the plan
   - Commit incrementally with `<issue-id>` in commit messages

4. **Complete:**
   ```bash
   bd close <issue-id> --reason "Completed"
   ```

### Labels

| Label | Meaning |
|-------|---------|
| `needs-plan-review` | Draft plan, then STOP and wait for human approval |
| `needs-ack` | Waiting on human decision |
| `blocked` | Can't proceed (see dependencies or notes) |

---

## Oracle: Consulting Other AI Models

When stuck, need a second opinion, or reviewing complex code, use Oracle to consult other models.

**Run once per session:**
```bash
npx -y @steipete/oracle --help
```

### Gemini 3 Pro (API - fast, huge context)

Best for: summarizing large files, parsing PDFs, broad codebase questions

```bash
npx -y @steipete/oracle --model gemini-3-pro --file <files> --prompt "<prompt>"
```

### GPT-5.2-Pro (Browser - requires `--engine browser`)

Best for: deep reasoning, architectural decisions, debugging complex issues

```bash
npx -y @steipete/oracle --engine browser --model gpt-5.2-pro --file <files> --prompt "<prompt>"
```

**Tips:**
- Use `gemini-3-pro` NOT `gemini-2.5-pro` (2.5 is terrible)
- GPT-5.2-Pro requires `--engine browser` flag
- Run GPT in background with `&` for parallel queries
- If browser runs fail to attach (ECONNREFUSED / No inspectable targets), use a fixed DevTools port and attach with `--remote-chrome` instead of re-running manual login.

**Oracle browser attach (fixed port)**
```bash
pkill -f "@steipete/oracle" || true
pkill -f "oracle-browser-" || true

# Start and login once (keep window open)
npx -y @steipete/oracle --engine browser --browser-manual-login --browser-keep-browser --browser-port 9223 -p "hello"

# Run the real command against the same browser
npx -y @steipete/oracle --engine browser --remote-chrome 127.0.0.1:9223 --browser-model-strategy current --model gpt-5.2-pro ...
```

---

## Rules

1. **Never delete files** without explicit human permission
2. **No destructive git commands** (`reset --hard`, `clean -fd`, `rm -rf`) without explicit approval
3. **Always create an exec plan** when starting substantive work

---

## Communication Style (very important)

- Be blunt + decisive. Default to statements, not hedged possibilities.
- Avoid hedging words: "maybe", "might", "could", "likely", "it depends" (unless you truly can't know).
- Lead with the answer / decision in the first sentence.
- If you're unsure, say "I don't know" + the single fastest way to verify.
- Don't add extra context or options unless asked.
- Prefer short numbered questions (yes/no) over long explanations.
- Mirror the user's wording/tone; keep it terse (aim <10 lines).
