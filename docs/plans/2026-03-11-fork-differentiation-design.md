# Fork Differentiation Design: Deep Worker Integration

**Date:** 2026-03-11
**Status:** Approved
**Approach:** Method B — Skills-first, preserving OMC infrastructure

---

## Problem Statement

In upstream OMC, Codex and Gemini CLI workers are **second-class citizens**:
- Described as "one-shot tmux workers that don't participate in team communication"
- No guidance on *when* to choose codex vs gemini vs a Claude sub-agent
- No optimization for each tool's unique capabilities
- Generic prompts — no `--output-schema`, no `BEFORE YOU EXIT`, no ephemeral flags
- No per-project team configuration

Claude is given spawning capability but no decision-making framework.

---

## Goal

Give Claude Code a **decision framework and dispatch expertise** for external workers:
1. When to use codex vs gemini vs internal Claude sub-agents
2. How to construct optimal prompts for each tool
3. How to read per-project team config from CLAUDE.md
4. Sensible defaults per project type (web / ml / research / tool)

---

## What We're Building (Scope)

### 1. `skills/codex-dispatch/SKILL.md`

A dedicated skill Claude invokes when dispatching to Codex CLI. Contains:

**Invocation template (canonical):**
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

**Key flags Claude must always include:**
- `--ephemeral`: prevents session conflicts when running parallel workers
- `-a never`: worker mode, never prompts for approval
- `-s workspace-write`: standard sandbox for coding tasks
- `--color never`: clean output for log parsing
- `-o <file>`: capture final message to file

**Optional structured output:**
- Use `--output-schema <schema.json>` when task needs JSON result (analysis, test reports)

**AGENTS.md rules:**
- Max 32KB — keep under 500 lines
- Only project facts (commands, paths, off-limits files) — NOT behavioral rules
- Behavioral rules in AGENTS.md are ignored by the model

**Prompt structure (test-anchored pattern):**
```
## Task
<one-sentence goal>

## Context
Files: [explicit list]
Key files: [most relevant]

## Constraints
- Do not modify: [list]
- Do not install new dependencies

## Done When
`npm test -- -t '<test name>'` passes with 0 failures

## BEFORE YOU EXIT
Write to /tmp/codex-status-<task-id>.json:
{
  "status": "success|failed|partial",
  "files_modified": [...],
  "summary": "<one sentence>",
  "blockers": "<if failed, what blocked you>"
}
```

**Task types Codex handles best:**
- Targeted implementation with clear acceptance criteria
- Refactoring with explicit rules (rename X to Y, extract function)
- Test generation from existing patterns
- Bug fixes with reproduction steps
- Boilerplate / CRUD scaffolding

**Task types to NOT send Codex:**
- Open-ended design ("build me a system that…") → decompose first
- Multi-repository tasks
- Abstract quality improvements ("make this cleaner")
- Tasks requiring network access (sandbox blocks DNS by default)

**Model selection:**
- `o4-mini`: default, high-volume coding, iteration loops
- `o3`: complex reasoning, hard debugging, architecture decisions

---

### 2. `skills/gemini-dispatch/SKILL.md`

A dedicated skill Claude invokes when dispatching to Gemini CLI. Contains:

**Invocation template (canonical):**
```bash
gemini \
  --approval-mode yolo \
  -m gemini-2.5-pro \
  -p "$(cat task.md)" \
  --output-format stream-json
```

**Key flags:**
- `--approval-mode yolo`: fully autonomous (more reliable than `auto_edit`)
- `-p "<prompt>"`: non-interactive single-shot (NOT `-i` which requires TTY)
- `-m gemini-2.5-pro`: explicit model (prevents auto-switching mid-task)
- `--output-format stream-json`: structured output for parsing

**GEMINI.md rules (inject into project GEMINI.md):**
```markdown
## Worker Constraints
- Never use ask_user tool. Never pause for confirmation.
- Use read_many_files for batch loading — not iterative read_file calls
- Use narrow ripgrep patterns — never broad globs on large repos
- Specify output paths explicitly — do not ask where to write

## BEFORE YOU EXIT
You MUST write your output before exiting. Your final action must be writing to the output path specified in the task.
```

**Prompt structure (large-context analytical pattern):**
```
## Task
<one-sentence goal>

## Files to Load
Load these files using read_many_files:
- /path/to/file1
- /path/to/file2
[list explicitly — do not ask Gemini to discover them]

## Output
Write result to: /path/to/output.md
Format: markdown report

## Done When
Output file exists with content at the specified path

## BEFORE YOU EXIT
Write the output file. Do not ask for confirmation. Your final action must be writing to the output file.
```

**Task types Gemini handles best:**
- Whole-codebase analysis (load 50+ files at once)
- Large document processing and synthesis
- Architecture understanding across many files
- Migration planning (analyze legacy → produce plan)
- Security audits (code + dependencies + patterns simultaneously)
- Research synthesis (web_search + web_fetch + analysis)
- Producing markdown reports

**Task types to NOT send Gemini:**
- Iterative code generation with test-fix loops → use Codex
- High-frequency short tool call tasks → overhead too high
- Stateful multi-step debugging → history cost grows exponentially

**Model selection:**
- `gemini-2.5-pro`: deep reasoning, complex multi-step, large codebase review
- `gemini-2.5-flash`: structured extraction, fast summarization, web ops

**Critical pitfalls to avoid:**
- Never use broad globs (glob `**/*.ts` on large repo = context overflow)
- Always specify `-m` explicitly (auto-switching degrades quality mid-task)
- One-shot prompts > multi-turn (history re-sent every turn = exponential cost)
- Must include `BEFORE YOU EXIT` or workers analyze but fail to write output

---

### 3. `skills/team-config/SKILL.md`

A skill for reading project CLAUDE.md team configuration and applying it.

**What it does:**
- Reads `[ai-team]` or similar section from project's CLAUDE.md
- Surfaces available workers and their configuration to Claude
- Gives Claude project-specific routing rules

**CLAUDE.md team config format (proposed):**
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
```

**Session-start behavior:**
On session start, Claude checks for this section. If found, it announces available workers and routing preferences.

---

### 4. `templates/teams/` — Project Type Templates

Pre-built CLAUDE.md team config sections for common project types:

- `templates/teams/web.md` — Frontend/backend web projects
- `templates/teams/ml.md` — Machine learning / data science
- `templates/teams/research.md` — Research / documentation heavy
- `templates/teams/tool.md` — CLI tools / libraries

Each template includes:
- Recommended worker roster
- Default routing rules for that project type
- Example task decompositions

**Onboarding flow:**
```
User enters project without AI Team config
→ Claude detects project type (web/ml/research/tool)
→ Claude proposes matching template
→ User confirms
→ Template appended to project CLAUDE.md
```

---

### 5. Enhancements to `skills/team/SKILL.md`

Add a **Worker Routing Decision Tree** section:

```
Task incoming
  ├─ Requires reading 20+ files? → gemini-dispatch
  ├─ Requires write-test-fix loop? → codex-dispatch
  ├─ Requires web research? → gemini-dispatch
  ├─ Requires architectural decision? → analyst/architect (Claude sub-agent)
  ├─ Mixed (analyze then implement)? → gemini-dispatch → codex-dispatch (sequential)
  └─ Default → codex-dispatch (for coding) or executor (for Claude native)
```

Add **Project Config Check** at team invocation start:
- Run `team-config` skill first
- Apply project-specific routing overrides if found

---

## What We Are NOT Changing

- TypeScript source (`src/`) — no changes needed for Method B
- OMC's tmux infrastructure — it works, we're not touching it
- Existing Claude sub-agents in `agents/` — they stay as-is
- Package name, version, release process

---

## Division of Labor (Permanent)

| Task | Handler |
|------|---------|
| Design, brainstorming, task decomposition | Claude (coordinator, never outsource) |
| Skill file writing (Markdown) | Codex (clear spec, verifiable output) |
| Large codebase analysis, reports | Gemini |
| Iterative implementation + tests | Codex |
| Template design | Claude |
| Template writing | Codex |

---

## File Inventory (What Gets Created/Modified)

### New files
```
skills/codex-dispatch/SKILL.md     ← comprehensive codex dispatch guide
skills/gemini-dispatch/SKILL.md    ← comprehensive gemini dispatch guide
skills/team-config/SKILL.md        ← CLAUDE.md-driven team config reader
templates/teams/web.md             ← web project team template
templates/teams/ml.md              ← ML project team template
templates/teams/research.md        ← research project team template
templates/teams/tool.md            ← CLI tool project team template
```

### Modified files
```
skills/team/SKILL.md               ← add worker routing decision tree + team-config hook
skills/ask/SKILL.md                ← add optimal invocation flags for codex/gemini
```

### Unchanged
```
src/                               ← no TypeScript changes
agents/                            ← no changes to Claude sub-agents
```

---

## Success Criteria

1. `codex-dispatch` skill produces prompts that consistently result in `BEFORE YOU EXIT` status files being written
2. `gemini-dispatch` skill prevents context overflow (no broad globs, explicit file paths)
3. `team-config` skill correctly reads and applies project CLAUDE.md team configuration
4. Claude can be observed making correct routing decisions: gemini for analysis, codex for implementation
5. A new web project can be onboarded with team config in under 2 minutes via template selection
