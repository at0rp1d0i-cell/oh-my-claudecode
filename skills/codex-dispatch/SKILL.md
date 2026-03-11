---
name: codex-dispatch
description: Expert dispatch guide for Codex CLI workers — optimal flags, prompt structure, task routing
---

# Codex Dispatch

Use this skill whenever you are about to dispatch a task to the Codex CLI worker. It contains the canonical invocation template, prompt structure, task suitability guidelines, and critical pitfalls.

## Output Directory Convention

All Codex worker status files go to `<project-root>/.ai-team/outputs/`:

```
<project-root>/
└── .ai-team/
    └── outputs/
        └── codex-<task-id>-status.json
```

- Add `.ai-team/` to `.gitignore` — outputs are transient, not source
- Coordinator always reads from this directory after worker completes
- Task prompt files (`/tmp/task-<id>/`) stay in `/tmp` — coordinator-side temp only


## Pre-Dispatch Checklist

**Before dispatching:** Verify the filled prompt contains no remaining `[PLACEHOLDER: ...]` strings. A task sent with unfilled placeholders will silently fail or produce irrelevant output.


## Canonical Invocation

```bash
codex exec \
  -m gpt-5.3-codex \
  --full-auto \
  --ephemeral \
  -C "$PROJECT_DIR" \
  --color never \
  -o "$STATUS_DIR/last-message.txt" \
  "$(cat "$TASK_DIR/prompt.txt")"
```

**Every invocation MUST include:**
- `--ephemeral` — prevents session conflicts when running parallel workers (without this, concurrent instances corrupt each other's sessions)
- `--full-auto` — fully autonomous worker mode (workspace-write sandbox + never prompts for approval; replaces `-a` and `-s` flags)
- `--color never` — clean output for log parsing
- `-C <absolute-path>` — always use absolute paths

**Optional structured output** (use when task needs JSON result):
```bash
codex exec ... --output-schema /path/to/schema.json "$(cat prompt.txt)"
```

## AGENTS.md Rules

AGENTS.md is injected as persistent project context. Hard constraints:
- **32 KB hard limit** — content beyond 32 KB is silently truncated; keep under 500 lines
- **Only project facts** — commands, file paths, off-limits directories, naming conventions
- **NOT behavioral rules** — abstract rules like "write clean code" are ignored by the model; repeat these inline in each task prompt

Recommended structure:
```markdown
# Project: <name>

## Build & Test
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`

## Layout
- Source: `src/`
- Tests: `tests/`
- Generated: `dist/` (do not edit)

## Constraints
- Never modify: `config/production.json`
- Do not commit; leave changes staged for orchestrator review
```

## Prompt Philosophy

**Give Codex intent, not steps. Trust it to decide HOW.**

Codex has 200k context and full reasoning capability. A good prompt tells it:
- **What** — one-sentence goal
- **Where** — starting files (not exhaustive; Codex can explore from there)
- **Boundary** — what not to touch
- **Done When** — verifiable acceptance criteria

A bad prompt writes out every step ("then open file X, then find function Y, then change line Z"). If your prompt contains the word "then", you're doing Codex's job for it.

## Prompt Structure

```
## Task
<one-sentence goal>

## Context
Starting files:
- src/path/to/relevant-file.ts
- tests/path/to/test-file.test.ts

## Constraints
- Do not modify: tests/, config/production.json
- Do not install new dependencies
- Leave changes staged, do not commit

## Done When
`npm test -- -t '<test name>'` passes

## BEFORE YOU EXIT
Write to <project-root>/.ai-team/outputs/codex-<task-id>-status.json:
{
  "status": "success" | "failed" | "partial",
  "files_modified": ["relative paths"],
  "summary": "one sentence",
  "blockers": "if failed: what blocked you"
}
```

**`BEFORE YOU EXIT` is mandatory.** Without it, Codex completes silently and the orchestrator hangs waiting for a status file that never arrives.

## Task Types: Send to Codex

- Targeted implementation with explicit acceptance criteria
- Refactoring with clear rules (rename X to Y, extract function)
- Test generation from existing patterns
- Bug fixes with reproduction steps
- Boilerplate generation (CRUD endpoints, scaffolding)
- Adding features to existing well-structured code

## Task Types: Do NOT Send to Codex

| Anti-pattern | Reason | Alternative |
|---|---|---|
| "Build me a system that…" | Too open-ended, no done criteria | Decompose first, then dispatch |
| Multi-repository tasks | Codex works in one working root | Split into per-repo tasks |
| Tasks requiring `git push` or network | Sandbox blocks DNS/SSH by default | Plan around offline execution |
| "Make this code cleaner" | Abstract quality rule, not implementable | Write a specific refactoring rule |
| Architecture decisions | Requires coordinator-level reasoning | Keep with Claude coordinator |

## Model Selection

| Model | Use when |
|---|---|
| `gpt-5.3-codex` | Default — high-volume coding, iteration, boilerplate |
| `gpt-5.4` | Hard debugging, complex reasoning, architecture analysis |

## Structured Output (Advanced)

For tasks needing structured data back from Codex (e.g., test failure list, analysis), use `--output-schema`:

Schema example for test failure report:
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "file": {"type": "string"},
      "testName": {"type": "string"},
      "failureReason": {"type": "string"}
    },
    "required": ["file", "testName", "failureReason"]
  }
}
```

## Worker Lifecycle (Coordinator's View)

Codex runs as a foreground process that exits when done. The coordinator's job:

```bash
# 1. Write task
echo "..." > /tmp/task-<id>/prompt.txt

# 2. Launch with timeout
timeout 300 codex exec -m gpt-5.3-codex --full-auto --ephemeral \
  -C "$PROJECT_DIR" --color never \
  "$(cat /tmp/task-<id>/prompt.txt)"
EXIT=$?

# 3. Read result
STATUS_FILE="<project-root>/.ai-team/outputs/codex-<id>-status.json"
if   [ $EXIT -eq 0 ]   && [ -f "$STATUS_FILE" ]; then  # success
elif [ $EXIT -eq 124 ]                                  ; then  # hung — timeout killed it
elif [ $EXIT -ne 0 ]   && [ -f "$STATUS_FILE" ]; then  # self-reported failure
else                                                            # crash — no status written
fi
```

**Recovery rules:**
| Condition | Meaning | Action |
|---|---|---|
| Exit 0 + status file | Completed normally | Read status, continue |
| Exit 0 + no status file | Forgot BEFORE YOU EXIT | Treat as partial, check git diff |
| Exit 124 | Hung (timeout killed it) | Report to user, do not retry automatically |
| Non-zero exit + status file | Self-reported failure | Read `blockers`, escalate to user |
| Non-zero exit + no status file | Crashed mid-task | Report exit code, escalate to user |

**Parallelism:**
- Always `--ephemeral` for concurrent instances
- Each needs a unique status file path (`<project-root>/.ai-team/outputs/codex-<task-id>-status.json`)
- Start with 1 worker, add parallelism only after confirming single-worker success

## Foreground vs Background Mode

**Choose based on estimated task duration:**

| Mode | When | How |
|---|---|---|
| Foreground | < 2 min, simple targeted task | `timeout 120 codex exec ...` — Claude blocks and waits |
| Background | > 2 min, parallel work possible | tmux session — Claude continues other work, polls status |

**Background launch (tmux):**
```bash
WORKER_ID="codex-$(date +%s)"
tmux new-session -d -s "$WORKER_ID" \
  "timeout 600 codex exec -m gpt-5.3-codex --full-auto --ephemeral \
   -C '$PROJECT_DIR' --color never \
   '$(cat /tmp/task-prompt.txt)'"
```

**Polling completion (check every ~30s):**
```bash
# Check if tmux session still alive
tmux has-session -t "$WORKER_ID" 2>/dev/null && echo "running" || echo "exited"

# Check for status file
ls <project-root>/.ai-team/outputs/codex-<task-id>-status.json 2>/dev/null
```

The session exits when Codex finishes. Status file appears if BEFORE YOU EXIT was followed.
OMC's tmux infrastructure handles session naming and cleanup — reuse it rather than rolling your own.
