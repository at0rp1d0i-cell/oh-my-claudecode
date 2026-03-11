# Deep Worker Integration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add deep Codex/Gemini CLI integration to the oh-my-claudecode fork via new skill files and template system, giving Claude an expert decision framework for external worker dispatch.

**Architecture:** Pure Markdown skill files — no TypeScript changes. New skills teach Claude optimal invocation patterns, routing decisions, and prompt structures for each CLI tool. A team-config skill reads per-project configuration from CLAUDE.md.

**Tech Stack:** Markdown (SKILL.md format), oh-my-claudecode skill loader, bash invocation patterns for codex/gemini CLIs.

**Design Doc:** `docs/plans/2026-03-11-fork-differentiation-design.md`

---

## Task 1: Create `skills/codex-dispatch/SKILL.md`

**Files:**
- Create: `skills/codex-dispatch/SKILL.md`

**Step 1: Create the skill file**

```bash
mkdir -p skills/codex-dispatch
```

Write `skills/codex-dispatch/SKILL.md` with this exact content:

```markdown
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
- `-a never` — worker mode, never prompts for approval; any other value may block the process indefinitely waiting for input that never comes
- `-s workspace-write` — standard sandbox for coding tasks
- `--color never` — clean output for log parsing
- `-C <absolute-path>` — always use absolute paths

**Optional structured output** (use when task needs JSON result):
```bash
codex exec ... --output-schema /path/to/schema.json "$(cat prompt.txt)"
```

## AGENTS.md Rules

AGENTS.md is injected as persistent project context. Constraints:
- **32 KB hard limit** — content beyond 32 KB is silently truncated with no warning; keep under 500 lines
- **Only project facts** — commands, file paths, off-limits directories, naming conventions
- **NOT behavioral rules** — abstract rules like "write clean code" or "no duplication" are ignored by the model; repeat these inline in each task prompt

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
<one-sentence goal — be specific, not generic>

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

**The `BEFORE YOU EXIT` section is mandatory.** Without it, Codex completes the task but never writes status, causing the orchestrator to hang waiting for a signal that never comes.

## Task Types: Send to Codex

- Targeted implementation with explicit acceptance criteria
- Refactoring with clear rules (rename X to Y, extract function, convert class)
- Test generation when given existing code and test patterns
- Bug fixes with reproduction steps
- Boilerplate generation (CRUD endpoints, component scaffolding)
- File format conversions (JSON → TypeScript types, SQL schema → ORM)
- Adding features to existing well-structured code

## Task Types: Do NOT Send to Codex

| Anti-pattern | Reason | Alternative |
|---|---|---|
| "Build me a system that…" | Too open-ended, no done criteria | Decompose first, then dispatch |
| Multi-repository tasks | Codex works in one working root | Split into per-repo tasks |
| Tasks requiring `git push` or network | Sandbox blocks DNS/SSH by default | Use `--dangerously-bypass-approvals-and-sandbox` only in CI |
| "Make this code cleaner" | Abstract quality rule, not implementable | Write a specific refactoring rule |
| Architecture decisions | Requires Claude-level reasoning | Keep with coordinator (Claude) |

## Model Selection

| Model | Use when |
|---|---|
| `o4-mini` | Default — high-volume coding, iteration, boilerplate |
| `o3` | Hard debugging, complex reasoning, architecture analysis |

## Structured Output (Advanced)

For tasks where you need structured data back from Codex (e.g., test failure list, code analysis), use `--output-schema`:

```bash
codex exec --output-schema /tmp/schema.json "Analyze tests/ and list all failing tests"
```

Schema example:
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
- API quota: start sequential, add parallelism incrementally
```

**Step 2: Verify the file**

```bash
ls -la skills/codex-dispatch/SKILL.md
grep -c "##" skills/codex-dispatch/SKILL.md
head -5 skills/codex-dispatch/SKILL.md
```

Expected: file exists, at least 10 `##` section headers, frontmatter starts with `---`

**Step 3: Commit**

```bash
git add skills/codex-dispatch/SKILL.md
git commit -m "feat: add codex-dispatch skill with expert invocation guide"
```

---

## Task 2: Create `skills/gemini-dispatch/SKILL.md`

**Files:**
- Create: `skills/gemini-dispatch/SKILL.md`

**Step 1: Create the skill file**

```bash
mkdir -p skills/gemini-dispatch
```

Write `skills/gemini-dispatch/SKILL.md` with this exact content:

```markdown
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
- `--approval-mode yolo` — fully autonomous, all tools run without confirmation; more reliable than `auto_edit` (known bug: `auto_edit` ignores "allow" rules for shell tools)
- `-p "<prompt>"` — non-interactive single-shot mode. **NEVER use `-i` (`--prompt-interactive`) for worker dispatch** — it requires a TTY and drops into interactive mode after the initial prompt
- `-m gemini-2.5-pro` — always specify model explicitly; without this, Gemini may auto-switch from Pro to Flash mid-task, degrading quality silently (issue #8186)
- `--output-format stream-json` — structured JSONL output; watch for `{"type":"result"}` event to detect task completion

**With additional directories:**
```bash
gemini \
  --approval-mode yolo \
  -m gemini-2.5-pro \
  --include-directories /path/to/project,/path/to/deps \
  -p "$(cat task.md)" \
  --output-format stream-json
```

## GEMINI.md Configuration

Add this block to the project's GEMINI.md before dispatching Gemini workers:

```markdown
## Worker Constraints

You are an autonomous worker. No user is watching. Follow these rules:

- **Never use ask_user tool.** Never pause for confirmation. If ambiguous, make a reasonable assumption and note it in the output.
- **Use read_many_files for batch loading** — not iterative read_file calls. Load all needed files in one operation.
- **Use narrow search patterns** — never use broad globs like `**/*.ts` on large repos (risks context overflow). Use ripgrep with specific patterns and file filters.
- **Write output to specified paths** — do not ask where to write; the task always specifies output paths.

## BEFORE YOU EXIT

You MUST write your output before exiting. Your absolute final action must be writing to the output file path specified in the task. Do not summarize to stdout and skip the file write.
```

**Why both GEMINI.md AND prompt?** GEMINI.md can be lost after `/clear` commands or context resets. Repeat the `BEFORE YOU EXIT` instruction in every task prompt as a safety net.

## Prompt Structure (Large-Context Analytical Pattern)

Always use this structure for Gemini tasks:

```
## Task
<one-sentence goal>

## Files to Load
Load ALL of these files at once using read_many_files before starting analysis:
- /absolute/path/to/file1.ts
- /absolute/path/to/file2.ts
- /absolute/path/to/file3.md
[list every file explicitly — do not ask Gemini to discover them]

## Search Patterns (if needed)
If searching is needed, use ripgrep with narrow patterns:
- Pattern: `specific_function_name`
- Glob: `--glob "src/**/*.ts"`
- Do NOT use: `glob("**/*")` or other broad patterns

## Output
Write result to: /absolute/path/to/output.md
Format: markdown report with sections

## Done When
1. Output file exists at the specified path
2. <specific acceptance criterion>

## BEFORE YOU EXIT
Your final action MUST be writing the output file. Do not ask for confirmation. Do not summarize and skip the file write.
```

**Explicit file paths, not discovery:** Gemini is capable of discovering files via glob/grep, but this costs tokens and risks context overflow. Always provide exact paths when known.

## Large Context Window: Practical Patterns

Gemini's ~1M token context enables loading entire codebases at once, but there are constraints:

**Use `read_many_files` for batch loading:**
```
Load all files at once using read_many_files:
- /project/src/auth/login.ts
- /project/src/auth/register.ts
- /project/src/auth/middleware.ts
```
This is far more efficient than iterative `read_file` calls.

**Avoid broad patterns that overflow context:**
```
# BAD — returns all 50k file paths, overflows context
glob("**/*.ts")

# GOOD — narrow and targeted
grep --pattern "authenticate" --glob "src/**/*.ts"
```

**One-shot preferred over multi-turn:**
Every conversation turn re-sends the ENTIRE history (APIs are stateless). In a 10-turn conversation on a large codebase, the 10th turn sends 10x the tokens of the first. Use one comprehensive prompt instead of building up through conversation.

## Task Types: Send to Gemini

- Whole-codebase analysis ("find all places where X pattern is used across src/")
- Large document processing and synthesis (200-page PDFs, long API specs)
- Architecture understanding across many files ("explain data flow from HTTP request to DB across these 30 files")
- Migration planning (analyze legacy codebase → produce structured migration plan)
- Security audits (load all code + dependencies + vulnerability patterns simultaneously)
- Research synthesis (web_search + web_fetch + analysis in one pass)
- Cross-file refactoring analysis (identify all callsites of a function)
- Producing markdown reports and structured analysis documents

## Task Types: Do NOT Send to Gemini

| Anti-pattern | Reason | Alternative |
|---|---|---|
| Iterative code generation with test-fix loops | High history cost per turn | Use Codex |
| High-frequency short tool calls | Overhead too high (full history each call) | Use Codex |
| Stateful multi-step debugging | History grows exponentially | Use Codex |
| Real-time shell interaction | Codex is more optimized for this | Use Codex |

## Model Selection

| Model | Use when |
|---|---|
| `gemini-2.5-pro` | Complex reasoning, architectural analysis, large codebase review, precision matters |
| `gemini-2.5-flash` | Structured extraction, fast summarization, web operations, cost-sensitive |

## Critical Pitfalls

1. **Context overflow from broad search** — `glob("**/*.ts")` on a 50k-file repo dumps all paths into context. Always specify narrow patterns with file type filters.
2. **Worker exits without writing output** — the #1 cause of silent failures. Always include `BEFORE YOU EXIT` in prompt.
3. **`ask_user` blocking** — even in yolo mode, Gemini may use `ask_user` if the task is ambiguous. Forbid it in GEMINI.md.
4. **Model auto-switching mid-task** — specify `-m` explicitly, always.
5. **GEMINI.md lost after `/clear`** — repeat critical constraints in every task prompt.
6. **Stdin duplication bug** — use `-p "prompt"` rather than piping stdin to avoid content duplication.

## Typical Collaboration Pattern

```
[Claude coordinator]
    → gemini-dispatch: "Analyze all 45 files in src/auth/ and produce a security audit report"
    ← reads /tmp/gemini-audit.md (output file)
    → codex-dispatch: "Fix the 3 vulnerabilities identified in /tmp/gemini-audit.md"
    ← reads codex status file
    → gemini-dispatch: "Review the diff produced by Codex and verify all issues are resolved"
```
```

**Step 2: Verify the file**

```bash
ls -la skills/gemini-dispatch/SKILL.md
grep -c "##" skills/gemini-dispatch/SKILL.md
grep "BEFORE YOU EXIT" skills/gemini-dispatch/SKILL.md | wc -l
```

Expected: file exists, at least 10 `##` headers, at least 3 mentions of "BEFORE YOU EXIT"

**Step 3: Commit**

```bash
git add skills/gemini-dispatch/SKILL.md
git commit -m "feat: add gemini-dispatch skill with large-context expert guide"
```

---

## Task 3: Create `skills/team-config/SKILL.md`

**Files:**
- Create: `skills/team-config/SKILL.md`

**Step 1: Create the skill file**

```bash
mkdir -p skills/team-config
```

Write `skills/team-config/SKILL.md` with this exact content:

```markdown
---
name: team-config
description: Read project AI Team configuration from CLAUDE.md and surface worker availability to coordinator
---

# Team Config

Read the current project's CLAUDE.md for AI Team configuration. Run this at the start of any session or before invoking `/team` to know which workers are available and what routing rules apply.

## Usage

```
/oh-my-claudecode:team-config
```

## What This Skill Does

1. Read `CLAUDE.md` in the current working directory
2. Find the `## AI Team` section (if present)
3. Surface worker availability and routing rules to you (Claude coordinator)
4. If no config found, offer to run onboarding

## Reading the Config

Look for this section in CLAUDE.md:

```markdown
## AI Team

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | implementation, test-fix loops |
| gemini | yes       | codebase analysis, large context |

### Routing Rules
- Architecture analysis → gemini
- Feature implementation → codex
- Design decisions → stay with Claude (coordinator)

### Notes
[any project-specific overrides]
```

## If Config Found

Report to the user:
```
AI Team configured for this project:
- codex: available (implementation, test-fix loops)
- gemini: available (codebase analysis, large context)

Routing: architecture → gemini, features → codex, design → coordinator
```

Then apply these routing rules throughout the session.

## If Config Not Found

```
No AI Team config found in CLAUDE.md.

Would you like to set up a team for this project?
Options:
- /oh-my-claudecode:team-config setup web     → web frontend/backend project
- /oh-my-claudecode:team-config setup ml      → machine learning / data science
- /oh-my-claudecode:team-config setup research → research / documentation heavy
- /oh-my-claudecode:team-config setup tool    → CLI tool / library
```

## Setup Flow (when user chooses a template)

1. Read the matching template from `templates/teams/<type>.md`
2. Show the user what will be added to CLAUDE.md
3. Ask for confirmation
4. Append the `## AI Team` section to CLAUDE.md
5. Confirm: "AI Team config added to CLAUDE.md"

## CLAUDE.md Format for AI Team Section

```markdown
## AI Team

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | [project-specific codex tasks] |
| gemini | yes       | [project-specific gemini tasks] |

### Routing Rules
- [Task type] → [worker]
- Design decisions → stay with Claude (never outsource)

### Codex Config
Model: o4-mini
Sandbox: workspace-write

### Gemini Config
Model: gemini-2.5-pro
```

## Task: {{ARGUMENTS}}
```

**Step 2: Verify the file**

```bash
ls -la skills/team-config/SKILL.md
grep "## AI Team" skills/team-config/SKILL.md
```

Expected: file exists, contains `## AI Team` section reference

**Step 3: Commit**

```bash
git add skills/team-config/SKILL.md
git commit -m "feat: add team-config skill for CLAUDE.md-driven team configuration"
```

---

## Task 4: Create Project Type Templates

**Files:**
- Create: `templates/teams/web.md`
- Create: `templates/teams/ml.md`
- Create: `templates/teams/research.md`
- Create: `templates/teams/tool.md`

**Step 1: Create templates directory**

```bash
mkdir -p templates/teams
```

**Step 2: Write `templates/teams/web.md`**

```markdown
## AI Team

> Web project (frontend + backend). Gemini for architecture/review, Codex for feature implementation.

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | React components, API endpoints, test-fix loops |
| gemini | yes       | Full-stack architecture review, dependency audits, large codebase analysis |

### Routing Rules
- Feature implementation (components, endpoints, hooks) → codex
- Architecture review, security audit, large refactoring analysis → gemini
- Test generation for specific functions → codex
- Whole-repo analysis ("find all places where X is used") → gemini
- Design decisions, API design, brainstorming → stay with Claude (coordinator)

### Codex Config
Model: o4-mini
Sandbox: workspace-write
Done criteria: always include a specific test command

### Gemini Config
Model: gemini-2.5-pro
Preferred tasks: one-shot analysis, producing structured reports
```

**Step 3: Write `templates/teams/ml.md`**

```markdown
## AI Team

> Machine learning / data science project. Gemini for research and large codebase analysis, Codex for training scripts and utilities.

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | Training scripts, data pipelines, utility functions, experiment configs |
| gemini | yes       | Research synthesis, paper analysis, large dataset schema review, model architecture comparison |

### Routing Rules
- Implement training loop, data loader, evaluation metrics → codex
- Research synthesis, paper cross-referencing, literature review → gemini
- Analyze experiment results across many log files → gemini
- Implement specific model architecture from paper → codex
- Design decisions, experiment strategy → stay with Claude (coordinator)

### Codex Config
Model: o3 (preferred for complex algorithm implementation)
Sandbox: workspace-write
Note: ML tasks often need longer iteration; use o3 for non-trivial implementations

### Gemini Config
Model: gemini-2.5-pro
Preferred tasks: loading entire src/ for architecture overview, research document analysis
```

**Step 4: Write `templates/teams/research.md`**

```markdown
## AI Team

> Research / documentation-heavy project. Gemini for reading and synthesizing large documents, Codex for tooling and automation.

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | Build scripts, automation tools, document processing utilities |
| gemini | yes       | Literature synthesis, large document analysis, cross-referencing, report generation |

### Routing Rules
- Synthesize multiple long documents → gemini
- Build automation scripts, document converters → codex
- Cross-reference sources across many files → gemini
- Web research and synthesis (web_search + web_fetch) → gemini
- Writing decisions, argument structure, editorial choices → stay with Claude (coordinator)

### Codex Config
Model: o4-mini
Sandbox: workspace-write

### Gemini Config
Model: gemini-2.5-pro
Note: Use read_many_files to load all source documents at once before analysis
```

**Step 5: Write `templates/teams/tool.md`**

```markdown
## AI Team

> CLI tool / library project. Codex-primary with Gemini for API design review and docs analysis.

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | Core implementation, test-fix loops, CLI command scaffolding, refactoring |
| gemini | yes       | API design review, large dependency analysis, compatibility audits |

### Routing Rules
- Implement new commands, flags, core logic → codex
- Review entire public API for consistency → gemini
- Analyze all dependencies for vulnerabilities → gemini
- Test generation for specific functions → codex
- API design decisions, breaking changes → stay with Claude (coordinator)

### Codex Config
Model: o4-mini
Sandbox: workspace-write
Done criteria: always include `npm test` or equivalent

### Gemini Config
Model: gemini-2.5-pro
Preferred tasks: loading entire src/ + test/ for API consistency review
```

**Step 6: Verify all templates**

```bash
ls -la templates/teams/
for f in templates/teams/*.md; do echo "=== $f ===" && head -3 "$f"; done
```

Expected: 4 files, each starts with `## AI Team`

**Step 7: Commit**

```bash
git add templates/teams/
git commit -m "feat: add project-type team templates (web, ml, research, tool)"
```

---

## Task 5: Enhance `skills/team/SKILL.md`

**Files:**
- Modify: `skills/team/SKILL.md`

**Step 1: Find the insertion point**

Read `skills/team/SKILL.md` and find the end of the "Architecture" section (around line 80) and the beginning of the "Staged Pipeline" section.

**Step 2: Insert the Worker Routing section**

Find the line containing `## Staged Pipeline (Canonical Team Runtime)` and insert the following block BEFORE it:

```markdown
## Worker Routing Decision Tree

Before spawning workers, apply this routing logic to select the right worker type:

```
Incoming task
  ├─ Requires reading 20+ files or entire codebase? → use :gemini worker (gemini-dispatch skill)
  ├─ Requires write + test + fix iteration loop? → use :codex worker (codex-dispatch skill)
  ├─ Requires web research (web_search + web_fetch)? → use :gemini worker
  ├─ Requires architectural decision or design? → use analyst/architect Claude sub-agent (NOT external worker)
  ├─ Mixed: analyze then implement?
  │     → Step 1: gemini-dispatch for analysis → write report to /tmp/
  │     → Step 2: codex-dispatch for implementation using report as context
  └─ Default coding task → use :codex worker
```

**When NOT to use external workers (codex/gemini):**
- Brainstorming, design decisions → stay with Claude coordinator
- Tasks requiring team communication protocol → use internal Claude sub-agents (executor, debugger, etc.)
- Tasks spanning multiple repositories → split and sequence

## Project Team Config Check

At the start of any `/team` invocation, check for project team configuration:

1. Look for `## AI Team` section in `CLAUDE.md`
2. If found: apply routing rules from config (project rules override defaults above)
3. If not found: proceed with defaults; optionally suggest `/oh-my-claudecode:team-config` setup

```bash
grep -A 20 "## AI Team" CLAUDE.md 2>/dev/null || echo "No team config found"
```

```

**Step 3: Verify the modification**

```bash
grep -n "Worker Routing Decision Tree" skills/team/SKILL.md
grep -n "Project Team Config Check" skills/team/SKILL.md
```

Expected: both section headers found with line numbers

**Step 4: Commit**

```bash
git add skills/team/SKILL.md
git commit -m "feat: add worker routing decision tree and team config check to team skill"
```

---

## Task 6: Enhance `skills/ask/SKILL.md`

**Files:**
- Modify: `skills/ask/SKILL.md`

**Step 1: Read the current file**

Read `skills/ask/SKILL.md` and find the end of the file.

**Step 2: Append the optimal invocation section**

Add this block at the END of `skills/ask/SKILL.md`:

```markdown

## Optimal Invocation by Provider

When routing to `codex` or `gemini`, use these flags:

### Codex
```bash
codex exec \
  -m o4-mini \
  -s workspace-write \
  -a never \
  --ephemeral \
  -C "$PROJECT_DIR" \
  --color never \
  "$(cat prompt.txt)"
```
Key: `--ephemeral` prevents session conflicts; `-a never` prevents blocking on approval prompts.

### Gemini
```bash
gemini \
  --approval-mode yolo \
  -m gemini-2.5-pro \
  -p "$(cat prompt.txt)" \
  --output-format stream-json
```
Key: use `-p` not `-i` (interactive requires TTY); `--approval-mode yolo` is more reliable than `auto_edit`.

## Quick Routing Guide

| Task type | Route to |
|---|---|
| Read many files, analyze codebase | `gemini` |
| Implement feature, fix tests | `codex` |
| Web research + synthesis | `gemini` |
| Boilerplate, scaffolding | `codex` |
| Design decision, brainstorming | `claude` (or stay in coordinator) |

For full dispatch guides: invoke `codex-dispatch` or `gemini-dispatch` skills.
```

**Step 3: Verify**

```bash
grep "Optimal Invocation by Provider" skills/ask/SKILL.md
grep "\-\-ephemeral" skills/ask/SKILL.md
grep "\-\-approval-mode yolo" skills/ask/SKILL.md
```

Expected: all three grep commands return matches

**Step 4: Commit**

```bash
git add skills/ask/SKILL.md
git commit -m "feat: add optimal CLI invocation flags and routing guide to ask skill"
```

---

## Task 7: Final Verification

**Step 1: Check all new files exist**

```bash
echo "=== New skill files ===" && \
ls -la skills/codex-dispatch/SKILL.md \
       skills/gemini-dispatch/SKILL.md \
       skills/team-config/SKILL.md && \
echo "=== Templates ===" && \
ls -la templates/teams/ && \
echo "=== Modified files ===" && \
grep -c "Worker Routing Decision Tree" skills/team/SKILL.md && \
grep -c "Optimal Invocation by Provider" skills/ask/SKILL.md
```

Expected output:
```
=== New skill files ===
[3 files with sizes]
=== Templates ===
[4 .md files]
=== Modified files ===
1
1
```

**Step 2: Check git log**

```bash
git log --oneline -7
```

Expected: 6 commits matching the feature commits from Tasks 1-6

**Step 3: Final commit if anything missed**

If any files were not committed in previous steps:
```bash
git status
git add -A
git commit -m "chore: ensure all fork differentiation files are committed"
```

---

## Summary

| Task | Files | Type |
|---|---|---|
| 1 | `skills/codex-dispatch/SKILL.md` | New |
| 2 | `skills/gemini-dispatch/SKILL.md` | New |
| 3 | `skills/team-config/SKILL.md` | New |
| 4 | `templates/teams/*.md` (×4) | New |
| 5 | `skills/team/SKILL.md` | Modified |
| 6 | `skills/ask/SKILL.md` | Modified |
| 7 | Verification | — |

**No TypeScript changes.** All work is Markdown skill files — pure content, no build step needed.
