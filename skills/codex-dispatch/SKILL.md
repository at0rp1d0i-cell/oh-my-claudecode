---
name: codex-dispatch
description: Expert dispatch guide for Codex CLI workers — optimal flags, prompt structure, task routing
---

# Codex Dispatch

Use this skill whenever you are about to dispatch a task to the Codex CLI worker. It contains the canonical invocation template, prompt structure, task suitability guidelines, and critical pitfalls.

## Canonical Invocation

```bash
codex exec \
  -m o4-mini \
  -s workspace-write \
  -a never \
  --ephemeral \
  -C "$PROJECT_DIR" \
  --color never \
  -o "$STATUS_DIR/last-message.txt" \
  "$(cat "$TASK_DIR/prompt.txt")"
```

**Every invocation MUST include:**
- `--ephemeral` — prevents session conflicts when running parallel workers (without this, concurrent instances corrupt each other's sessions)
- `-a never` — worker mode, never prompts for approval; any other value may block the process indefinitely
- `-s workspace-write` — standard sandbox for coding tasks
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

## Prompt Structure (Test-Anchored Pattern)

Always use this structure for Codex tasks:

```
## Task
<one-sentence goal — be specific>

## Context
Files to look at:
- src/path/to/relevant-file.ts
- tests/path/to/test-file.test.ts

## Constraints
- Do not modify: tests/, config/production.json
- Do not install new dependencies
- Do not commit — leave changes staged

## Done When
`npm test -- -t '<exact test name>'` passes with 0 failures

## BEFORE YOU EXIT
Write to /tmp/codex-status-<task-id>.json:
{
  "status": "success" | "failed" | "partial",
  "files_modified": ["list of relative paths"],
  "summary": "one sentence describing what was done",
  "blockers": "if failed: what prevented completion"
}
```

**The `BEFORE YOU EXIT` section is mandatory.** Without it, Codex completes the task but never writes status, causing the orchestrator to hang.

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
| `o4-mini` | Default — high-volume coding, iteration, boilerplate |
| `o3` | Hard debugging, complex reasoning, architecture analysis |

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

## Multi-Agent Safety

- Always `--ephemeral` for parallel Codex instances
- Each instance needs a unique status file path (use task-id in filename)
- Set 5-minute heartbeat timeout — Codex has no built-in progress reporting
- Start sequential, add parallelism incrementally to avoid quota drain

## Done When
File exists at skills/codex-dispatch/SKILL.md and starts with the frontmatter block (--- name: codex-dispatch ---).

## BEFORE YOU EXIT
Write to /tmp/codex-task1-status.json:
{
  "status": "success" or "failed",
  "files_modified": ["skills/codex-dispatch/SKILL.md"],
  "summary": "created codex-dispatch skill file"
}
