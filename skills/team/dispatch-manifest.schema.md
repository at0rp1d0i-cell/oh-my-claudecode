# Dispatch Manifest

The coordinator builds one dispatch manifest before every external worker launch. This is a lightweight planning artifact, not a formal validator schema.

## Example

```json
{
  "task_id": "auth-audit-01",
  "provider": "gemini",
  "mode": "coworker",
  "model": "gemini-2.5-pro",
  "reasoning_effort": "high",
  "input_files": [
    "/home/torpedo/Workspace/claude-exploring/oh-my-claudecode/skills/team/SKILL.md",
    "/home/torpedo/Workspace/claude-exploring/oh-my-claudecode/CLAUDE.md"
  ],
  "output_path": "/home/torpedo/Workspace/claude-exploring/oh-my-claudecode/.ai-team/outputs/gemini-auth-audit-01-output.md",
  "timeout_seconds": 300,
  "verification_command": "test -f /home/torpedo/Workspace/claude-exploring/oh-my-claudecode/.ai-team/outputs/gemini-auth-audit-01-output.md",
  "template_used": "skills/gemini-dispatch/tasks/codebase-analysis.md"
}
```

## Fields

| Field | Required | Description |
|---|---|---|
| `task_id` | Yes | Stable task identifier used in filenames, logs, and status tracking. |
| `provider` | Yes | External worker target: `codex` or `gemini`. |
| `mode` | Yes | Dispatch intent: `executor` for code-changing work or `coworker` for review-only work. |
| `model` | Yes | Exact model the coordinator intends to pass to the CLI. Read this from project config or user instructions. |
| `reasoning_effort` | Yes | Coordinator-selected reasoning level for the worker, for example `medium`, `high`, or `xhigh`. |
| `input_files` | Yes | Absolute paths the worker must read before starting. Keep this list explicit and minimal. |
| `output_path` | Yes | Absolute path where the worker must write its result or status file. For team workflows this should usually be under `<project-root>/.ai-team/outputs/`. |
| `timeout_seconds` | Yes | Hard timeout for the worker process. The coordinator uses this when launching the CLI. |
| `verification_command` | Yes | Exact command the coordinator runs after completion to confirm success, usually a file existence check or targeted test command. |
| `template_used` | Yes | Filled task template used to produce the worker prompt, such as `skills/codex-dispatch/tasks/plan-review.md`. |

## Coordinator Rules

- Build the manifest before writing the final worker prompt.
- Verify no placeholder text remains in the manifest or in the filled template.
- Construct the CLI command from manifest fields rather than ad hoc values.
- Keep the manifest lightweight and task-specific. If a field is unclear, resolve it before dispatch instead of letting the worker guess.
