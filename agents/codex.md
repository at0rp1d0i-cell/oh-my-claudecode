# AGENTS.md Template for Codex Workers

> **Usage:** When onboarding a project, append the filled-in version of this template
> to the project's `AGENTS.md`. Fill in the `[PLACEHOLDER]` values from the actual project.
> This file is injected as persistent context into every Codex session.
>
> **32 KB hard limit.** Keep under 500 lines. Only project facts — not behavioral rules.
> Behavioral rules (e.g., "write clean code") are ignored by the model; put those in task prompts.

---

```markdown
# Project: [PROJECT_NAME]

## What This Is
[One sentence describing what this project does and its current focus.]
Example: "REST API for user authentication — currently adding OAuth2 support."

## Build & Test
- Build: `[command]`
- Test: `[command]`
- Lint: `[command]`
- Type check: `[command, or omit if N/A]`

## Layout
- Source: `[src path]`
- Tests: `[tests path]`
- Config: `[config path]`
- Generated / do NOT edit: `[dist/, generated/, __pycache__/, etc.]`

## Off-Limits Files
- Never modify: `[list critical files, e.g., config/production.json, .env, migration files]`
- Never delete: `[list]`

## Naming Conventions
- [e.g., "Files use kebab-case", "Functions use camelCase", "Tests colocated with source"]

## Output Convention
Write task completion status to: `/tmp/codex-<task-id>-status.json`
Format:
{
  "status": "success" | "failed" | "partial",
  "files_modified": ["relative/path"],
  "summary": "one sentence",
  "blockers": "if failed: what stopped you"
}

## Orchestrator Rules
- Do not commit — leave changes staged for orchestrator review
- Do not push to remote
- Do not install new dependencies without noting them in the status summary
```
