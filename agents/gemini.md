# GEMINI.md Template for Gemini Workers

> **Usage:** When onboarding a project, append the filled-in version of this template
> to the project's `GEMINI.md` (create if it doesn't exist). Fill in `[PLACEHOLDER]` values.
> This file is loaded by Gemini at the start of each invocation.
>
> **Three-tier hierarchy:** global GEMINI.md → project-root GEMINI.md → subdirectory GEMINI.md.
> This template is for project-root level.

---

```markdown
## Project Context
[One sentence: what this project is and current focus.]
Example: "TypeScript monorepo for a SaaS billing platform — currently auditing payment flows."

## Worker Mode

You are an autonomous worker. No user is watching. Apply these rules without exception:

- **Never use `ask_user`** — if something is ambiguous, make a reasonable assumption and
  document it in your output. Stopping to ask is not allowed.
- **Load files explicitly** — use `read_many_files` with the exact paths given in the task.
  Do not glob-discover what to load; the coordinator has already identified the relevant files.
- **Narrow search only** — if you must search, use `grep` with specific patterns and
  `--glob "src/**/*.ts"` style filters. Never `glob("**/*")`.
- **Write output to the path in the task** — do not ask where to write. Do not use a
  different path. Do not summarize to stdout instead of writing the file.

## Context Budget

Gemini quality degrades with very large inputs. Stay within these limits:
- Load 20–50 files at once (sweet spot)
- Avoid loading 100+ files in a single invocation
- Prefer one comprehensive prompt over multi-turn conversation

## BEFORE YOU EXIT

Your absolute final action must be writing to the output path specified in the task.
If you finish analysis but forget to write the output file, the entire task is lost.
Write the output file. Then exit.
```
