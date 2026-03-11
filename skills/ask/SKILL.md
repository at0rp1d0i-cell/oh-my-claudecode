---
name: ask
description: Ask Claude, Codex, or Gemini via local CLI and capture a reusable artifact
---

# Ask

Use OMC's canonical advisor skill to route a prompt through the local Claude, Codex, or Gemini CLI and persist the result as an ask artifact.

## Usage

```bash
/oh-my-claudecode:ask <claude|codex|gemini> <question or task>
```

Examples:

```bash
/oh-my-claudecode:ask codex "review this patch from a security perspective"
/oh-my-claudecode:ask gemini "suggest UX improvements for this flow"
/oh-my-claudecode:ask claude "draft an implementation plan for issue #123"
```

## Routing

Preferred path:

```bash
omc ask {{ARGUMENTS}}
```

## Requirements

- The selected local CLI must be installed and authenticated.
- Verify availability with the matching command:

```bash
claude --version
codex --version
gemini --version
```

## Artifacts

`omc ask` writes artifacts to:

```text
.omc/artifacts/ask/<provider>-<slug>-<timestamp>.md
```

Task: {{ARGUMENTS}}

## Optimal Invocation by Provider

When routing to `codex` or `gemini`, always use these exact flags:

### Codex
```bash
codex exec \
  -m gpt-5.3-codex \
  --full-auto \
  --ephemeral \
  -C "$PROJECT_DIR" \
  --color never \
  "$(cat prompt.txt)"
```
- `--full-auto`: workspace-write sandbox + on-request approval (no `-a` flag in exec mode)
- `--ephemeral`: prevents session conflicts when running parallel instances
- For complex tasks: use `-m gpt-5.4`
- For structured output: add `--output-schema /path/to/schema.json`

### Gemini
```bash
gemini \
  --approval-mode yolo \
  -m gemini-3 \
  -p "$(cat prompt.txt)" \
  --output-format stream-json
```
- Use `-p` not `-i` (`-i` requires TTY, not suitable for worker dispatch)
- `--approval-mode yolo` is more reliable than `auto_edit` for autonomous use
- Always specify `-m gemini-3` explicitly to prevent auto-switching mid-task

## Quick Routing Guide

| Task type | Route to |
|---|---|
| Read 20+ files, analyze entire codebase | `gemini` |
| Implement feature, write + test + fix loop | `codex` |
| Web research + synthesis | `gemini` |
| Architecture review across many files | `gemini` |
| Boilerplate, scaffolding, refactoring | `codex` |
| Design decision, brainstorming | Stay with Claude coordinator |

For full dispatch guides: invoke `codex-dispatch` or `gemini-dispatch` skills.
