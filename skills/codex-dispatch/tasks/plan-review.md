<!-- WHEN TO USE: Dispatch this template BEFORE executing a plan — Codex acts as coworker, evaluating the proposed approach against the actual codebase. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before dispatching -->
<!-- ROLE: Coworker (informed reviewer, not executor — do NOT modify any files) -->

## Task
Evaluate the dispatch manifest below against the current codebase. Identify strengths, risks, and gaps the Coordinator may have missed before sending it to an external worker.

## Dispatch Manifest
```json
[PLACEHOLDER: paste the completed dispatch manifest JSON]
```

## Project Goal
[PLACEHOLDER: one sentence — what is this project trying to achieve]

## Key Files To Read
- [PLACEHOLDER: path to files listed in manifest.input_files that are most critical]
- [PLACEHOLDER: path to files the manifest may have missed but the review should inspect]
- [PLACEHOLDER: path to tests or specs that define current behavior]

## What To Evaluate

### Feasibility
- Can this manifest be executed with the current codebase structure?
- Are `input_files`, `template_used`, `timeout_seconds`, and `verification_command` realistic for the work described?
- Are there hidden dependencies or coupling the manifest doesn't account for?

### Completeness
- What scenarios does the manifest NOT address?
- Are there edge cases, error paths, or integration points being overlooked?

### Design Fit
- Does the dispatch choice follow existing project conventions and patterns?
- Does it introduce unnecessary complexity or deviation from current architecture?

### Risk
- What could go wrong during execution?
- What's the blast radius if something fails?
- Is the selected `provider`, `mode`, `model`, or `reasoning_effort` mismatched for this task?

## Constraints
- **Read-only** — do not modify any files
- Base your evaluation on actual code, not assumptions
- If the manifest references files that don't exist, flag it

## Done When
- All 4 evaluation dimensions (Feasibility, Completeness, Design Fit, Risk) addressed with specific file references
- Manifest-specific fields (`provider`, `mode`, `input_files`, `output_path`, `verification_command`, `template_used`) are either validated or challenged
- `findings_by_severity` addressed and verdict supported by evidence in strengths/findings_by_severity

## Output Format
Write to <project-root>/.ai-team/outputs/codex-[PLACEHOLDER: task-id]-status.json:
```json
{
  "status": "success",
  "files_modified": [],
  "verdict": "looks-good | needs-discussion | needs-rework",
  "recommendation": "approve | revise | reject",
  "strengths": [
    "what the plan gets right (with file:line evidence)"
  ],
  "findings_by_severity": {
    "high": [
      {
        "issue": "specific problem",
        "evidence": "file:line, manifest field, or concrete observation",
        "impact": "why this matters"
      }
    ],
    "medium": [
      {
        "issue": "important but non-blocking concern",
        "evidence": "file:line, manifest field, or concrete observation",
        "impact": "why this matters"
      }
    ],
    "low": [
      {
        "issue": "minor concern or cleanup item",
        "evidence": "file:line, manifest field, or concrete observation",
        "impact": "why this matters"
      }
    ]
  },
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
