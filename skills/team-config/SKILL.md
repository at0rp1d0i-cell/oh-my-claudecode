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
3. Surface which workers are enabled and what routing rules apply
4. If no config found, offer to run onboarding

## Reading the Config

Look for this section in CLAUDE.md:

```markdown
## AI Team

### Workers
| Worker | Enabled | Strength |
|--------|---------|---------|
| codex  | yes     | implementation, test-fix loops |
| gemini | no      | codebase analysis, large context |

### Routing Rules
- Architecture analysis → gemini
- Feature implementation → codex
- Design decisions → stay with Claude (coordinator)
```

## If Config Found

Report to the user:
```
AI Team configured for this project:
- codex: enabled (implementation, test-fix loops)
- gemini: disabled

Routing: features → codex, design → coordinator
Note: gemini is disabled — large-context analysis will stay with coordinator.
```

Then apply these routing rules throughout the session.

## Worker Enable/Disable

Check the `Enabled` column for each worker:

- `yes` — worker is available; route tasks to it per routing rules
- `no` — worker is not installed or disabled for this project; skip entirely

**Fallback rules when a worker is disabled:**

| Disabled | Fallback |
|----------|---------|
| codex    | Use Claude internal sub-agent (executor role) for implementation tasks |
| gemini   | Keep large-context analysis with Claude coordinator; do not route to codex |
| both     | Full Claude-only mode; warn user that external worker orchestration is unavailable |

If Gemini is disabled but a task clearly needs large-context analysis, say:
> "Gemini is disabled for this project. I'll handle this analysis directly, but quality may be lower for very large codebases."

## If Config Not Found

```
No AI Team config found in CLAUDE.md.

Would you like to set up a team for this project?
Options:
- /oh-my-claudecode:team-config setup web      → web frontend/backend project
- /oh-my-claudecode:team-config setup ml       → machine learning / data science
- /oh-my-claudecode:team-config setup research → research / documentation heavy
- /oh-my-claudecode:team-config setup tool     → CLI tool / library
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
| Worker | Enabled | Strength |
|--------|---------|---------|
| codex  | yes     | [project-specific codex tasks] |
| gemini | yes     | [project-specific gemini tasks] |

### Routing Rules
- [Task type] → [worker]
- Design decisions → stay with Claude (never outsource)

### Codex Config
Model: gpt-5.3-codex

### Gemini Config
Model: gemini-3
```
