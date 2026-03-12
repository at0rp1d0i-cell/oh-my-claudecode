<!-- WHEN TO USE: Dispatch this template AFTER a worker completes a task — Codex acts as coworker, evaluating the produced changes against the original intent. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before dispatching -->
<!-- ROLE: Coworker (informed reviewer, not executor — do NOT modify any files) -->

## Task
Review the changes produced by a worker and assess whether they meet the original task intent, follow project conventions, and introduce no regressions.

## Original Task Intent
[PLACEHOLDER: the goal that was given to the executor worker]

## Changes To Review
Modified files (from executor's status file):
- [PLACEHOLDER: path/to/modified-file-1]
- [PLACEHOLDER: path/to/modified-file-2]

Diff baseline: `git diff [PLACEHOLDER: base-commit-sha]..HEAD -- <files above>`

## Reference Files (for context)
- [PLACEHOLDER: path to project conventions or patterns the changes should follow]
- [PLACEHOLDER: path to related tests]
- [PLACEHOLDER: path to upstream/unmodified code for comparison]

## What To Evaluate

### Correctness
- Do the changes actually achieve the stated task intent?
- Are there logic errors, off-by-one mistakes, or missed branches?

### Scope Discipline
- Did the worker stay within the requested scope?
- Are there unnecessary changes (formatting, unrelated refactors, added dependencies)?

### Convention Compliance
- Do the changes match existing code style, naming, and patterns in this project?
- Are new patterns introduced where existing ones would suffice?

### Test Coverage
- Are new/modified behaviors covered by tests?
- Do existing tests still pass? Any tests that should have been updated but weren't?

### Risks
- Could these changes break something outside the modified files?
- Are there concurrency, performance, or security concerns?

## Constraints
- **Read-only** — do not modify any files
- Read the actual diff, don't rely on the worker's self-reported summary
- Compare against project conventions in existing code, not abstract best practices

## Done When
- All 5 evaluation dimensions (Correctness, Scope Discipline, Convention Compliance, Test Coverage, Risks) addressed with specific file references
- `findings_by_severity` addressed and verdict supported by evidence in strengths/findings_by_severity

## Output Format
Write to <project-root>/.ai-team/outputs/codex-[PLACEHOLDER: task-id]-status.json:
```json
{
  "status": "success",
  "files_modified": [],
  "verdict": "looks-good | needs-discussion | needs-rework",
  "strengths": [
    "what the implementation gets right (with file:line evidence)"
  ],
  "findings_by_severity": {
    "high": [
      {
        "issue": "specific problem found in the diff",
        "evidence": "file:line or concrete observation",
        "impact": "why this matters"
      }
    ],
    "medium": [
      {
        "issue": "important but non-blocking concern",
        "evidence": "file:line or concrete observation",
        "impact": "why this matters"
      }
    ],
    "low": [
      {
        "issue": "minor concern or convention mismatch",
        "evidence": "file:line or concrete observation",
        "impact": "why this matters"
      }
    ]
  },
  "suggestions": [
    {
      "what": "concrete improvement to the implementation",
      "why": "benefit and cost",
      "where": "affected files/sections"
    }
  ],
  "files_reviewed": [
    "path/to/file-or-diff-the-review-actually-read"
  ],
  "summary": "one-paragraph overall assessment"
}
```

## BEFORE YOU EXIT
Write the JSON above. If you cannot complete the review, set verdict to "needs-discussion" and explain why in summary.
