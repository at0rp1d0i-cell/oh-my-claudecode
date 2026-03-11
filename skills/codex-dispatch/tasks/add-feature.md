<!-- WHEN TO USE: Dispatch this template when adding a new feature that extends existing product behavior. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before dispatching -->

## Task
Implement [PLACEHOLDER: feature name] so [PLACEHOLDER: user-visible outcome].

## Feature Intent
- User story: [PLACEHOLDER: who needs this and why]
- In scope: [PLACEHOLDER: concrete capabilities this feature must add]
- Out of scope: [PLACEHOLDER: adjacent behavior that must not be added]

## Existing Patterns To Reuse
- Similar existing code lives in:
  - [PLACEHOLDER: path to first similar implementation]
  - [PLACEHOLDER: path to second similar implementation]
- Follow these project conventions:
  - [PLACEHOLDER: key pattern, helper, or architecture rule to mirror]

## Code Placement
- New logic should live in: [PLACEHOLDER: target directories/modules]
- New tests should live in: [PLACEHOLDER: target test directories/files]
- Keep these files/directories off-limits: [PLACEHOLDER: protected files/dirs]

## Feature Boundary Tests
- Add or update tests that define the boundary for:
  - [PLACEHOLDER: positive scenario]
  - [PLACEHOLDER: negative or edge scenario]
- Verification command: `[PLACEHOLDER: test command]`

## Context
Starting files:
- [PLACEHOLDER: path to relevant source file]
- [PLACEHOLDER: path to relevant test file]

## Constraints
- Do not install new dependencies
- Leave changes staged, do not commit

## Done When
- Feature behavior matches the Feature Intent and stays within the declared scope
- Code follows existing patterns in the referenced locations
- `[PLACEHOLDER: test command]` passes

## BEFORE YOU EXIT
Write to <project-root>/.ai-team/outputs/codex-[PLACEHOLDER: task-id]-status.json:
{
  "status": "success" | "failed" | "partial",
  "files_modified": ["relative paths"],
  "summary": "one sentence",
  "blockers": "if failed: what blocked you"
}
