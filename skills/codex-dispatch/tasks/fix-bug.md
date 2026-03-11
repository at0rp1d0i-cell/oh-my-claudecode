<!-- WHEN TO USE: Dispatch this template when fixing a specific bug with known reproduction steps and expected behavior. -->
<!-- FILL IN: all [PLACEHOLDER] values before dispatching -->

## Task
Fix [PLACEHOLDER: bug description] so [PLACEHOLDER: corrected behavior after fix].

## Reproduction
Steps to reproduce:
1. [PLACEHOLDER: step 1]
2. [PLACEHOLDER: step 2]
3. [PLACEHOLDER: step 3]

Expected behavior:
[PLACEHOLDER: what should happen]

Actual behavior:
[PLACEHOLDER: what currently happens]

Impact and frequency:
[PLACEHOLDER: who/what is affected and how often]

## Bug Boundary
- Must fix: [PLACEHOLDER: specific failing behavior]
- Must not change: [PLACEHOLDER: nearby behavior that should remain unchanged]

## Context
Starting files:
- [PLACEHOLDER: path to relevant source file]
- [PLACEHOLDER: path to relevant test file]

## Verification
- Regression test(s) to add or update: [PLACEHOLDER: test files or cases]
- Test command: `[PLACEHOLDER: test command]`

## Constraints
- Do not modify: [PLACEHOLDER: list of protected files/dirs]
- Do not install new dependencies
- Leave changes staged, do not commit

## Done When
- Reproduction steps no longer produce the bug
- Expected behavior is observed for the documented reproduction path
- `[PLACEHOLDER: test command]` passes

## BEFORE YOU EXIT
Write to /tmp/codex-[PLACEHOLDER: task-id]-status.json:
{
  "status": "success" | "failed" | "partial",
  "files_modified": ["relative paths"],
  "summary": "one sentence",
  "blockers": "if failed: what blocked you"
}
