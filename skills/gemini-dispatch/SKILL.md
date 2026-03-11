---
name: gemini-dispatch
description: Expert dispatch guide for Gemini CLI workers — optimal flags, prompt structure, large-context patterns
---

# Gemini Dispatch

Use this skill whenever you are about to dispatch a task to the Gemini CLI worker. It contains the canonical invocation template, GEMINI.md configuration, large-context best practices, and critical pitfalls.

## Canonical Invocation

```bash
gemini \
  --approval-mode yolo \
  -m gemini-2.5-pro \
  -p "$(cat task.md)" \
  --output-format stream-json
```

**Critical flags:**
- `--approval-mode yolo` — fully autonomous; more reliable than `auto_edit` (known bug: `auto_edit` ignores allow-rules for shell tools)
- `-p "<prompt>"` — non-interactive single-shot. **NEVER use `-i`** (`--prompt-interactive`) for worker dispatch — it requires a TTY
- `-m gemini-2.5-pro` — always specify model explicitly; without this, Gemini may auto-switch from Pro to Flash mid-task (issue #8186)
- `--output-format stream-json` — JSONL output; watch for `{"type":"result"}` event for completion

## GEMINI.md Configuration

Add this block to the project's GEMINI.md before dispatching Gemini workers:

```markdown
## Worker Constraints

You are an autonomous worker. No user is watching. Follow these rules:

- **Never use ask_user tool.** If ambiguous, make a reasonable assumption and note it in output.
- **Use read_many_files for batch loading** — not iterative read_file calls.
- **Use narrow search patterns** — never broad globs like `**/*.ts` on large repos (context overflow risk).
- **Write output to specified paths** — do not ask where to write.

## BEFORE YOU EXIT
Your absolute final action must be writing to the output file path specified in the task.
```

**Why both GEMINI.md AND prompt?** GEMINI.md can be lost after `/clear` or context resets. Repeat `BEFORE YOU EXIT` in every task prompt as a safety net.

## Prompt Structure (Large-Context Analytical Pattern)

```
## Task
<one-sentence goal>

## Files to Load
Load ALL of these files at once using read_many_files before starting:
- /absolute/path/to/file1.ts
- /absolute/path/to/file2.ts
[list explicitly — do not ask Gemini to discover them]

## Search Patterns (if needed)
Use ripgrep with narrow patterns:
- Pattern: `specific_function_name`
- Glob: `--glob "src/**/*.ts"`
- Do NOT use: broad glob("**/*")

## Output
Write result to: /absolute/path/to/output.md
Format: markdown report

## Done When
Output file exists at the specified path

## BEFORE YOU EXIT
Your final action MUST be writing the output file. Do not ask for confirmation.
```

**Explicit file paths, not discovery:** Always provide exact paths — Gemini discovering files costs tokens and risks context overflow.

## Large Context: Practical Patterns

**Use read_many_files for batch loading:**
Load all files at once — far more efficient than iterative read_file calls.

**Avoid patterns that overflow context:**
```
# BAD — returns all file paths, overflows context
glob("**/*.ts")

# GOOD — narrow and targeted
grep --pattern "authenticate" --glob "src/**/*.ts"
```

**One-shot preferred over multi-turn:**
Every conversation turn re-sends the ENTIRE history (APIs are stateless). A 10-turn conversation sends 10x the tokens of turn 1. Use one comprehensive prompt.

## Task Types: Send to Gemini

- Whole-codebase analysis ("find all places where X pattern is used")
- Large document processing and synthesis
- Architecture understanding across many files
- Migration planning (analyze legacy → produce structured plan)
- Security audits (load all code + dependencies simultaneously)
- Research synthesis (web_search + web_fetch + analysis in one pass)
- Producing markdown reports and structured analysis documents

## Task Types: Do NOT Send to Gemini

| Anti-pattern | Reason | Alternative |
|---|---|---|
| Iterative code generation with test-fix loops | High history cost per turn | Use Codex |
| High-frequency short tool calls | Full history overhead each call | Use Codex |
| Stateful multi-step debugging | History grows exponentially | Use Codex |
| Real-time shell interaction | Codex is more optimized | Use Codex |

## Model Selection

| Model | Use when |
|---|---|
| `gemini-2.5-pro` | Complex reasoning, architecture review, precision matters |
| `gemini-2.5-flash` | Structured extraction, fast summarization, web operations |

## Critical Pitfalls

1. **Context overflow from broad search** — `glob("**/*.ts")` on a large repo dumps all paths into context. Always use narrow patterns.
2. **Worker exits without writing output** — #1 cause of silent failures. Always include `BEFORE YOU EXIT`.
3. **`ask_user` blocking** — even in yolo mode, Gemini may use `ask_user` if task is ambiguous. Forbid it in GEMINI.md.
4. **Model auto-switching mid-task** — specify `-m` explicitly, always.
5. **GEMINI.md lost after `/clear`** — repeat critical constraints in every task prompt.
6. **Stdin duplication bug** — use `-p "prompt"` rather than piping stdin.

## Typical Collaboration Pattern

```
[Claude coordinator]
    → gemini-dispatch: "Analyze all files in src/auth/ and produce a security audit report"
    ← reads /tmp/gemini-audit.md
    → codex-dispatch: "Fix the 3 vulnerabilities identified in /tmp/gemini-audit.md"
    ← reads codex status file
    → gemini-dispatch: "Review the diff produced by Codex and verify all issues are resolved"
```

## Done When
File exists at skills/gemini-dispatch/SKILL.md and starts with the frontmatter block.

## BEFORE YOU EXIT
Write to /tmp/codex-task2-status.json:
{
  "status": "success" or "failed",
  "files_modified": ["skills/gemini-dispatch/SKILL.md"],
  "summary": "created gemini-dispatch skill file"
}
