<!-- WHEN TO USE: Dispatch this template when refactoring code to a defined pattern or structure without changing intended behavior. -->
<!-- FILL IN: all [PLACEHOLDER] values before dispatching -->

## Task
Refactor [PLACEHOLDER: scope/component] from [PLACEHOLDER: current pattern] to [PLACEHOLDER: target pattern].

## Refactor Intent
- Scope: [PLACEHOLDER: modules/files included in this migration]
- Pattern to migrate FROM: [PLACEHOLDER: old pattern/structure]
- Pattern to migrate TO: [PLACEHOLDER: target pattern/structure]
- Behavior that must remain unchanged: [PLACEHOLDER: invariants]
- Explicit non-goals: [PLACEHOLDER: what should not be refactored]

## Context
Starting files:
- [PLACEHOLDER: path to relevant source file]
- [PLACEHOLDER: path to relevant test file]

## Migration Boundaries
- Do not modify: [PLACEHOLDER: list of protected files/dirs]
- Public interfaces that must stay compatible: [PLACEHOLDER: APIs/contracts]

## Constraints
- Do not install new dependencies
- Leave changes staged, do not commit

## Verification
- Existing tests that protect current behavior: [PLACEHOLDER: key test suites/files]
- Test command: `[PLACEHOLDER: test command for full affected suite]`

## Done When
- Code in scope follows the target pattern and no longer uses the old pattern where migration is required
- Existing behavior is preserved for declared invariants
- All existing tests still pass, including `[PLACEHOLDER: test command for full affected suite]`

## BEFORE YOU EXIT
Write to /tmp/codex-[PLACEHOLDER: task-id]-status.json:
{
  "status": "success" | "failed" | "partial",
  "files_modified": ["relative paths"],
  "summary": "one sentence",
  "blockers": "if failed: what blocked you"
}
