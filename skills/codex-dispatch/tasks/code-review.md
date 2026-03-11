<!-- WHEN TO USE: Dispatch this template when Codex should review the code quality of a specific module, PR, or changeset without making modifications. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before dispatching -->
<!-- ROLE: Coworker (informed reviewer, not executor — do NOT modify any files) -->

## Task
Review the code quality of [PLACEHOLDER: module/PR/changeset] and assess whether the implementation is correct, consistent with project patterns, adequately tested, and safe to ship.

## Review Scope
- Review target: [PLACEHOLDER: module name, PR title/number, or changeset description]
- Intended behavior or requirements:
  - [PLACEHOLDER: expected behavior or acceptance criterion]
  - [PLACEHOLDER: additional requirement or invariant]
- Files in scope:
  - [PLACEHOLDER: path/to/file-1]
  - [PLACEHOLDER: path/to/file-2]
- Diff baseline (if reviewing a changeset): `git diff [PLACEHOLDER: base-commit-sha]..[PLACEHOLDER: head-commit-sha-or-ref] -- <files above>`

## Reference Files (for context)
- [PLACEHOLDER: path to similar or canonical implementation]
- [PLACEHOLDER: path to related tests/specs]
- [PLACEHOLDER: path to conventions, README, or architecture note]

## What To Evaluate

### Code Correctness
- Does the implementation satisfy the intended behavior in the Review Scope?
- Are there logic errors, incorrect assumptions, or missed branches?

### Patterns and Naming
- Does the code follow existing project patterns, abstractions, and naming conventions?
- Are new helpers or abstractions justified, or is the code inventing unnecessary structure?

### Test Coverage
- Are the important behavior changes or code paths covered by tests?
- Are there missing regression tests for the risky paths in scope?

### Edge Cases
- Are boundary conditions, invalid inputs, empty states, or failure paths handled correctly?
- Are there hidden integration risks at module boundaries?

### Security
- Could the code introduce authorization, input validation, secret handling, or injection risks?
- Are there trust-boundary assumptions that should be challenged?

## Constraints
- **Read-only** — do not modify any files
- Read the actual code and diff if one is provided; do not rely on summaries alone
- Evaluate against project conventions in the repository, not generic best practices

## Done When
- All 5 evaluation dimensions (Code Correctness, Patterns and Naming, Test Coverage, Edge Cases, Security) are addressed with specific file references
- Findings are grouped by severity and supported by evidence
- Verdict is set and supported by strengths/findings/suggestions

## Output Format
Write to <project-root>/.ai-team/outputs/codex-[PLACEHOLDER: task-id]-status.json:
```json
{
  "status": "success",
  "files_modified": [],
  "verdict": "looks-good | needs-discussion | needs-rework",
  "strengths": [
    "what the code gets right (with file:line evidence)"
  ],
  "findings_by_severity": {
    "high": [
      {
        "issue": "specific correctness, security, or maintainability problem",
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
        "issue": "minor concern, cleanup, or convention mismatch",
        "evidence": "file:line or concrete observation",
        "impact": "why this matters"
      }
    ]
  },
  "test_gaps": [
    "missing or weak test coverage areas, with file references when possible"
  ],
  "suggestions": [
    {
      "what": "concrete improvement to the code or test suite",
      "why": "benefit and cost",
      "where": "affected files/sections"
    }
  ],
  "summary": "one-paragraph overall assessment"
}
```

## BEFORE YOU EXIT
Write the JSON above. If you cannot complete the review, set verdict to "needs-discussion" and explain why in summary.
