---
name: gemini-dispatch
description: Expert dispatch guide for Gemini CLI workers — optimal flags, prompt structure, large-context patterns
---

# Gemini Dispatch

Use this skill whenever you are about to dispatch a task to the Gemini CLI worker. It contains the canonical invocation template, GEMINI.md configuration, large-context best practices, and critical pitfalls.

## Output Directory Convention

All Gemini worker output files go to `<project-root>/.ai-team/outputs/`:

```
<project-root>/
└── .ai-team/
    └── outputs/
        └── gemini-<task-id>-output.md
```

- Add `.ai-team/` to `.gitignore` — outputs are transient, not source
- Coordinator always reads from this directory after worker completes
- Gemini sandbox blocks `/tmp` writes; this path is always safe


## Pre-Dispatch Checklist

**Before dispatching:**
- Build a dispatch manifest using `skills/team/dispatch-manifest.schema.md`.
- Verify the manifest and the filled prompt contain no remaining `[PLACEHOLDER: ...]` strings.
- Use manifest fields, not ad hoc shell literals, when assembling the Gemini command.

## Pre-Dispatch: Build Manifest

Before the canonical invocation, the coordinator writes a dispatch manifest for this worker. The manifest is the source of truth for:

- `task_id`
- `provider=gemini`
- `mode=executor` or `mode=coworker`
- `model`
- `reasoning_effort`
- `input_files`
- `output_path`
- `timeout_seconds`
- `verification_command`
- `template_used`

The coordinator must validate that every manifest field is filled, every referenced path is absolute, and the prompt generated from `template_used` contains no placeholders before launching Gemini.


## Canonical Invocation

```bash
gemini \
  --approval-mode yolo \
  -m "$MODEL" \
  -p "$(cat task.md)" \
  --output-format stream-json
```

**Critical flags:**
- `--approval-mode yolo` — fully autonomous; more reliable than `auto_edit` (known bug: `auto_edit` ignores allow-rules for shell tools)
- `-p "<prompt>"` — non-interactive single-shot. **NEVER use `-i`** (`--prompt-interactive`) for worker dispatch — it requires a TTY
- `-m "$MODEL"` — pass the exact manifest-selected model explicitly (without this, Gemini may auto-switch mid-task)
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

## Gemini's Actual Strength

Gemini's advantage is **loading the right files at once**, not "unlimited context".

| Zone | File count | Quality |
|---|---|---|
| Sweet spot | 20–50 explicitly named files | Excellent |
| Acceptable | 50–100 files | Degrades noticeably |
| Avoid | 100+ files / broad discovery | Unreliable |

Long context does not mean better results — it means Gemini can handle more than other models before degrading. **Do not treat Gemini as a "dump everything in" solution.**

**Practical rules:**
- Provide explicit file paths — do not ask Gemini to discover what it needs
- Use `read_many_files` for batch loading, not iterative `read_file` calls
- One-shot prompt: every turn re-sends the full history (stateless API). A 10-turn conversation costs 10x turn-1 tokens.
- Narrow ripgrep patterns when search is unavoidable: `grep --pattern "functionName" --glob "src/**/*.ts"`

**What breaks Gemini:**
```
# BAD — dumps all paths into context, causes overflow
glob("**/*.ts")

# GOOD — targeted load of known relevant files
read_many_files(["src/auth/jwt.ts", "src/auth/middleware.ts", "tests/auth.test.ts"])
```

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
| `gemini-3` | Default — complex reasoning, architecture review, large-context analysis |
| `gemini-2.5` | Fallback — structured extraction, fast summarization |

## Critical Pitfalls

1. **Context overflow from broad search** — `glob("**/*.ts")` on a large repo dumps all paths into context. Always use narrow patterns.
2. **Worker exits without writing output** — #1 cause of silent failures. Always include `BEFORE YOU EXIT`.
3. **`ask_user` blocking** — even in yolo mode, Gemini may use `ask_user` if task is ambiguous. Forbid it in GEMINI.md.
4. **Model group selectors only work interactively** — `-m gemini-3` in `-p` mode causes ModelNotFoundError. Either omit `-m` (uses default auto model) or verify the exact API model ID first.
5. **Model auto-switching mid-task** — when specifying `-m`, use an exact API model ID, not a group selector.
6. **GEMINI.md lost after `/clear`** — repeat critical constraints in every task prompt.
7. **Stdin duplication bug** — use `-p "prompt"` rather than piping stdin.
8. **`/tmp` write restriction** — Gemini sandbox blocks writes to `/tmp`. Use the project workspace for output files (e.g. `.ai-team/outputs/`) or Gemini's own temp dir (`~/.gemini/tmp/<project>/`).

## Worker Lifecycle (Coordinator's View)

Gemini `-p` mode always exits — it has no "hung" state. The only failure mode is writing output or not.

```bash
# 0. Build dispatch manifest
# manifest.task_id=<id>
# manifest.model=<model>
# manifest.output_path=<project-root>/.ai-team/outputs/gemini-<id>-output.md
# manifest.timeout_seconds=180
# manifest.template_used=skills/gemini-dispatch/tasks/<template>.md

# 1. Write task from the filled template referenced by manifest.template_used

# 2. Verify manifest + prompt have no placeholders, then launch with timeout
timeout "$TIMEOUT_SECONDS" gemini --approval-mode yolo -m "$MODEL" \
  -p "$(cat task.md)" --output-format stream-json
EXIT=$?

# 3. Read result from manifest.output_path and run manifest.verification_command
OUTPUT_FILE="$OUTPUT_PATH"
if   [ $EXIT -eq 0 ]   && [ -f "$OUTPUT_FILE" ]; then  # success
elif [ $EXIT -eq 124 ]                                  ; then  # hung (rare in -p mode)
else                                                            # failed / no output written
fi
```

**Recovery rules:**
| Condition | Meaning | Action |
|---|---|---|
| Exit 0 + output file | Completed normally | Read output, continue |
| Exit 0 + no output file | Forgot BEFORE YOU EXIT | Task likely done, output lost — retry with stricter prompt |
| Exit 124 | Hung (prompt too complex) | Decompose task further, retry |
| Non-zero exit | API error or crash | Check error message, retry once |

## Typical Collaboration Pattern

```
[Claude coordinator]
    → gemini-dispatch: "Analyze all files in src/auth/ and produce a security audit report"
    ← reads <project-root>/.ai-team/outputs/gemini-audit.md
    → codex-dispatch: "Fix the 3 vulnerabilities identified in <project-root>/.ai-team/outputs/gemini-audit.md"
    ← reads codex status file
    → gemini-dispatch: "Review the diff produced by Codex and verify all issues are resolved"
```
