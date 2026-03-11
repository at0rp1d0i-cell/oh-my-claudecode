<!-- WHEN TO USE: Use this reference when the Coordinator wants to convert a Gemini analysis report into a Codex implementation task without losing scope, evidence, or verification detail. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before using this handoff -->
<!-- ROLE: Coordinator reference (bridge Gemini analysis into Codex execution input) -->

## Purpose
Standardize the handoff from Gemini analysis output to Codex implementation input so the Coordinator preserves evidence, narrows scope, and gives Codex a task it can execute without redoing the analysis pass.

## Gemini Analysis Output Must Contain

### Required Context
- **Problem statement:** [PLACEHOLDER: one sentence describing the issue/opportunity]
- **Scope:** [PLACEHOLDER: exact modules, files, or workflows analyzed]
- **Evidence:** [PLACEHOLDER: file paths, line references, or concrete observations]
- **Priority:** [PLACEHOLDER: high | medium | low for each finding]
- **Recommended action options:** [PLACEHOLDER: candidate remediations with trade-offs]
- **Constraints / non-goals:** [PLACEHOLDER: what must not change]
- **Verification clues:** [PLACEHOLDER: tests, commands, or observable behaviors that would prove success]

### Preferred Finding Shape
Gemini reports are easier to translate when each actionable finding includes:
1. **Issue:** what is wrong or missing
2. **Impact:** why it matters
3. **Evidence:** exact files/lines or repository observations
4. **Candidate action:** what Codex could change
5. **Risk / caveat:** what the Coordinator should protect against
6. **Validation hint:** how to verify the fix

## Coordinator Transformation Rules

### 1. Choose The Correct Codex Template
- Use `skills/codex-dispatch/tasks/fix-bug.md` when Gemini identifies a broken behavior with clear expected vs actual behavior.
- Use `skills/codex-dispatch/tasks/refactor.md` when Gemini identifies structural or pattern problems and behavior should stay unchanged.
- Use `skills/codex-dispatch/tasks/add-feature.md` when Gemini identifies missing functionality and the goal is to add new behavior.
- Use `skills/codex-dispatch/tasks/code-review.md` when Gemini's output should be validated by an independent Codex coworker before implementation.

### 2. Convert Findings Into Task Parameters
- **`## Task`** comes from Gemini's highest-priority actionable finding, rewritten as one concrete outcome.
- **Scope files** come from the evidence section, trimmed to the minimum set Codex needs to start.
- **Reference files** come from "good examples" or neighboring tests identified in the analysis.
- **Constraints** come from Gemini's non-goals, compatibility notes, and risk caveats.
- **Done When** comes from Gemini's validation hints, converted into observable acceptance criteria and test commands.
- **Output path** always follows `.ai-team/outputs/codex-[PLACEHOLDER: task-id]-status.json`.

### 3. Remove Analysis Noise Before Dispatch
- Do not pass every finding into a single Codex task if they are unrelated; split them into separate tasks.
- Do not forward speculative ideas without evidence.
- Do not ask Codex to "analyze and implement" in one prompt when Gemini already did the analysis; preserve the division of labor.
- Do keep the evidence that justifies the task, especially file references and test expectations.

## Minimum Handoff Packet For Codex
Before dispatching, the Coordinator should be able to fill all of these from the Gemini report:
- **Goal:** [PLACEHOLDER: one sentence]
- **Starting files:** [PLACEHOLDER: 1-5 concrete file paths]
- **Boundary:** [PLACEHOLDER: protected files, invariants, or non-goals]
- **Verification:** [PLACEHOLDER: test command or observable success condition]
- **Risk note:** [PLACEHOLDER: the most important caveat Codex must avoid]

## Concrete Example

### Gemini Analysis Output (excerpt)
- **Problem statement:** authorization checks are duplicated across three HTTP handlers and one handler is missing the admin guard.
- **Evidence:**
  - `src/server/routes/admin-users.ts:18-44` performs inline role checking
  - `src/server/routes/admin-teams.ts:15-39` performs similar inline role checking
  - `src/server/routes/admin-billing.ts:22-41` performs similar inline role checking
  - `src/server/routes/admin-settings.ts:12-36` has no admin guard
- **Candidate action:** extract a shared `requireAdmin` middleware and apply it to all admin routes
- **Constraints / non-goals:** preserve existing HTTP responses and do not change public route paths
- **Validation hint:** existing route tests under `tests/server/admin-routes.test.ts` should be extended to cover unauthorized access to `admin-settings`

### Coordinator Transformation
- **Chosen Codex template:** `skills/codex-dispatch/tasks/refactor.md`
- **Why:** behavior should stay the same except for the missing guard defect, and the main goal is to move duplicated authorization logic to a shared pattern

### Codex Task Parameters
- **Task:** Refactor admin route authorization from duplicated inline role checks to shared `requireAdmin` middleware.
- **Refactor Intent / Scope:** `src/server/routes/admin-users.ts`, `src/server/routes/admin-teams.ts`, `src/server/routes/admin-billing.ts`, `src/server/routes/admin-settings.ts`
- **Behavior that must remain unchanged:** existing success responses and route paths
- **Starting files:**
  - `src/server/routes/admin-users.ts`
  - `src/server/middleware/require-admin.ts`
  - `tests/server/admin-routes.test.ts`
- **Verification:**
  - Add/update tests for unauthorized access to `admin-settings`
  - Run: `npm test -- tests/server/admin-routes.test.ts`

## Coordinator Checklist
- Is the Gemini finding evidence-backed and specific enough to implement?
- Does the chosen Codex template match the kind of work required?
- Are scope, non-goals, and verification concrete enough that Codex will not need to re-analyze the whole codebase?
- If multiple findings remain, should they become separate Codex tasks instead of one overloaded prompt?
