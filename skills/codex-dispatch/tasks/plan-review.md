<!-- WHEN TO USE: Dispatch this template BEFORE executing a plan — Codex acts as coworker, evaluating the proposed approach against the actual codebase. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before dispatching -->
<!-- ROLE: Coworker (informed reviewer, not executor — do NOT modify any files) -->

## Task
Evaluate the proposed plan below against the current codebase. Identify strengths, risks, and gaps the Coordinator may have missed.

## Proposed Plan
[PLACEHOLDER: paste the full plan/approach that Coordinator is about to execute]

## Project Goal
[PLACEHOLDER: one sentence — what is this project trying to achieve]

## Key Files To Read
- [PLACEHOLDER: path to files most affected by this plan]
- [PLACEHOLDER: path to files the plan depends on]
- [PLACEHOLDER: path to tests or specs that define current behavior]

## What To Evaluate

### Feasibility
- Can this plan be implemented with the current codebase structure?
- Are there hidden dependencies or coupling the plan doesn't account for?

### Completeness
- What scenarios does the plan NOT address?
- Are there edge cases, error paths, or integration points being overlooked?

### Design Fit
- Does the plan follow existing project conventions and patterns?
- Does it introduce unnecessary complexity or deviation from current architecture?

### Risk
- What could go wrong during execution?
- What's the blast radius if something fails?

## Constraints
- **Read-only** — do not modify any files
- Base your evaluation on actual code, not assumptions
- If the plan references files that don't exist, flag it

## Done When
- All 4 evaluation dimensions (Feasibility, Completeness, Design Fit, Risk) addressed with specific file references
- Verdict is set and supported by evidence in strengths/concerns

## Output Format
Write to <project-root>/.ai-team/outputs/codex-[PLACEHOLDER: task-id]-status.json:
```json
{
  "status": "success",
  "files_modified": [],
  "verdict": "looks-good | needs-discussion | needs-rework",
  "strengths": [
    "what the plan gets right (with file:line evidence)"
  ],
  "concerns": [
    {
      "issue": "specific problem",
      "evidence": "file:line or concrete observation",
      "severity": "high | medium | low"
    }
  ],
  "suggestions": [
    {
      "what": "concrete improvement",
      "why": "benefit and cost",
      "where": "affected files/sections"
    }
  ],
  "summary": "one-paragraph overall assessment"
}
```

## BEFORE YOU EXIT
Write the JSON above. If you cannot complete the review, set verdict to "needs-discussion" and explain why in summary.
