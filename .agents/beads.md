# Beads (bd) Issue Tracking Guide

> **Purpose**: This guide teaches agents how to use beads (`bd`) for persistent issue tracking in this project. beads provides graph-based dependency tracking and survives context compaction.

## Why beads?

- **Persistent memory**: Issues survive context compaction and session boundaries
- **Dependency graphs**: Track blockers, parent-child, and discovered work relationships
- **Git-backed**: Auto-syncs to JSONL, shared across machines via git
- **Agent-optimized**: `--json` output, ready work detection, batch operations

## When to Use beads vs TodoWrite

| Use beads when... | Use TodoWrite when... |
|---|---|
| Work spans multiple sessions | Task completes this session |
| Complex dependencies exist | Linear step-by-step execution |
| Context might get compacted | All context in conversation |
| Need to resume after days/weeks | Simple checklist for now |

**Rule of thumb**: If resuming after 2 weeks without bd would be hard, use bd.

---

## Quick Start

### Initialize (one-time per project)

```bash
bd init --quiet  # Non-interactive, auto-installs git hooks
```

### Core Workflow

```bash
# 1. Find available work
bd ready --json

# 2. Claim a task
bd update <id> --status in_progress --json

# 3. Work on it...

# 4. Discover new work while coding? File it immediately
bd create "Found bug in X" -d "Details about the bug" -t bug -p 1 --deps discovered-from:<current-id> --json

# 5. Complete the work
bd close <id> --reason "Implemented X with Y" --json

# 6. ALWAYS sync at session end
bd sync
```

---

## Essential Commands

### Finding Work

```bash
bd ready --json                    # Issues with no blockers (includes all priorities)
bd ready --priority 0,1,2,3 --json # Skip backlog (P4) — USE THIS BY DEFAULT
bd ready --priority 1 --json       # Only P1 issues
bd blocked --json                  # See what's stuck and why
bd stale --days 30 --json          # Forgotten issues
bd list --status in_progress --json # Currently active work
```

**Important**: When picking work autonomously, use `--priority 0,1,2,3` to skip backlog (P4) items.

### Creating Issues

```bash
# Basic issue
bd create "Title" -d "Description" -t <type> -p <priority> --json

# With discovered-from link (one command)
bd create "Found bug" -d "Details" -t bug -p 1 --deps discovered-from:<parent-id> --json

# Types: bug | feature | task | epic | chore
# Priorities: 0 (critical) | 1 (high) | 2 (medium) | 3 (low) | 4 (backlog)
```

**Always include descriptions!** Future agents need context:

```bash
# Good
bd create "Fix auth token refresh" \
  -d "Tokens aren't refreshing before expiry, causing 401s on long sessions. Discovered during user testing." \
  -t bug -p 1 --json

# Bad - no context for future resumption
bd create "Fix auth bug" -t bug -p 1 --json
```

### Updating Issues

```bash
bd update <id> --status in_progress --json
bd update <id> --priority 0 --json
bd update <id> --assignee claude --json

# Batch updates
bd update <id1> <id2> <id3> --priority 1 --json
```

#### Valid Statuses

| Status | Meaning |
|--------|---------|
| `open` | Not started |
| `in_progress` | Currently being worked on |
| `closed` | Completed |

**Note**: Use `bd close <id>` (preferred) or `bd update <id> --status closed` to close issues. The status `done` is **not valid**.

### Closing Issues

```bash
bd close <id> --reason "Completed: implemented X with Y" --json

# Batch close
bd close <id1> <id2> --reason "Completed in session" --json
```

### Viewing Issues

```bash
bd show <id> --json                # Full details
bd show <id1> <id2> --json         # Multiple issues
bd dep tree <id>                   # Dependency visualization
bd list --status open --json       # All open issues
bd stats --json                    # Project health overview
```

### Filtering

```bash
bd list --status open --priority 1 --json
bd list --label-any urgent,critical --json
bd list --type bug --json
bd list --title-contains "auth" --json
bd list --created-after 2024-01-01 --json
```

---

## Dependencies

### Types

| Type | Purpose | Affects `bd ready`? |
|------|---------|---------------------|
| `blocks` | Hard dependency - X blocks Y | Yes |
| `related` | Soft link - connected but not blocking | No |
| `parent-child` | Epic/subtask hierarchy | No |
| `discovered-from` | Provenance - found during work on parent | No |

### Adding Dependencies

```bash
# A depends on B (B must be done first, B blocks A)
# Think: "A depends on B" → "B blocks A"
bd dep add <dependent> <blocker> --type blocks

# Example: Phase 2 depends on Phase 1 (Phase 1 must complete first)
bd dep add phase-2-id phase-1-id --type blocks

# Mark as discovered during parent work
bd dep add <child-id> <parent-id> --type discovered-from

# Or create with dependency in one command (preferred)
bd create "New issue" -d "Details" --deps discovered-from:<parent-id> --json
```

**Common mistake**: The argument order is `<dependent> <blocker>`, NOT `<blocker> <dependent>`. If Phase 1 must complete before Phase 2 can start, run `bd dep add phase-2 phase-1`.

### Visualizing Dependencies

```bash
bd dep tree <id>      # Show dependency tree
bd dep cycles         # Detect circular dependencies
```

---

## Hierarchical Issues (Epics)

For large features, use hierarchical IDs:

```bash
# Create epic
bd create "Auth System" -t epic -p 1 --json
# Returns: bd-a3f8e9

# Child tasks auto-number within epic
bd create "Login UI" -p 1 --json         # bd-a3f8e9.1
bd create "Password reset" -p 1 --json   # bd-a3f8e9.2
bd create "OAuth flow" -p 1 --json       # bd-a3f8e9.3
```

---

## Issue Types & Priorities

### Types

- `bug` - Something broken
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature with subtasks
- `chore` - Maintenance (dependencies, tooling)

### Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (default)
- `3` - Low (polish, optimization)
- `4` - **Backlog** (future ideas, phase-2 work) — **agents should skip P4 by default**

---

## Session Workflow

### Session Start

```bash
# Check for active and ready work
bd ready --json
bd list --status in_progress --json

# If in_progress exists, read context
bd show <id> --json
```

### During Work

- **Proactively file issues** for bugs, TODOs, and follow-up work discovered
- Use `discovered-from` to maintain context chains
- Update status when starting work on an issue

### Session End (Critical!)

```bash
# ALWAYS run before ending session
bd sync
```

This exports changes, commits, pulls, imports updates, and pushes. **Never skip this.**

### Landing the Plane

When asked to "land the plane" or wrap up:

1. File issues for remaining work
2. Run quality gates (tests, linting) if code changed
3. Close completed issues
4. **PUSH to remote** (mandatory):
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # Verify "up to date with origin"
   ```
5. Clean up: `git stash clear && git remote prune origin`
6. Recommend next issue for follow-up session

---

## Duplicate Detection

```bash
# Check for duplicates before creating
bd list --json | grep -i "search term"

# Find all duplicates
bd duplicates

# Auto-merge duplicates
bd duplicates --auto-merge

# Manual merge
bd merge <source1> <source2> --into <target> --json
```

---

## Notes for Compaction Survival

When updating issues, write notes that survive context loss:

```bash
bd update <id> --notes "
COMPLETED: JWT validation with RS256 (12 rounds)
KEY DECISION: Chose RS256 over HS256 for key rotation
IN PROGRESS: Rate limiting implementation
BLOCKERS: Need user input on rate limit thresholds
NEXT: Implement 5 req/min limit once confirmed
"
```

**Format**:
- **COMPLETED**: Specific deliverables (not "made progress")
- **KEY DECISIONS**: Important context with WHY
- **IN PROGRESS**: Current state + next step
- **BLOCKERS**: What's preventing progress
- **NEXT**: Immediate next action

---

## Cleanup & Maintenance

```bash
# Delete closed issues older than N days
bd cleanup --older-than 90 --force --json

# Compaction (summarize old closed issues)
bd compact --analyze --json
bd compact --apply --id <id> --summary summary.txt

# Restore compacted issue from git history
bd restore <id>
```

---

## Common Patterns

### Pattern 1: Bug Discovery During Feature Work

```bash
# Working on bd-abc feature, discover a bug
bd create "Found: login doesn't handle special chars" \
  -d "Password field truncates at # character. Discovered during OAuth testing." \
  -t bug -p 1 --deps discovered-from:bd-abc --json

# Continue with original work
```

### Pattern 2: Multi-Session Project Resume

```bash
# Start of new session
bd ready --json                           # What's available?
bd list --status in_progress --json       # What was I doing?
bd show <in-progress-id> --json           # Read notes for context
bd update <id> --status in_progress       # Resume work
```

### Pattern 3: Epic Breakdown

```bash
# Create parent epic
bd create "V4 Frontend Integration" -t epic -d "Hook up V4 orchestrator to UI" -p 1 --json

# Break into tasks
bd create "Add V4 API routes" -d "..." -p 1 --json
bd create "Update interview state handlers" -d "..." -p 1 --json
bd create "Add V4-specific error handling" -d "..." -p 1 --json

# Add blocking relationships
bd dep add <task1> <task2> --type blocks
```

---

## Key Takeaways

1. **Always use `--json`** for programmatic output
2. **Always include descriptions** with context
3. **Always run `bd sync`** at session end
4. **Use `discovered-from`** to track where issues came from
5. **Check `bd ready`** before asking "what's next?"
6. **Write notes for future-you** - context may be lost to compaction
