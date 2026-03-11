<!-- WHEN TO USE: Use this when planning to refactor legacy code, switch frameworks, or modernize a specific subsystem. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before dispatching -->

## Task
Analyze the legacy code and produce a structured migration plan for [PLACEHOLDER: source component/framework] to [PLACEHOLDER: target system]. Identify technical risks, breaking changes, and a phased implementation strategy.

## Analytical Dimensions
Evaluate the loaded files across these migration-focused dimensions:
1. **Dependency Analysis:** Map out what other modules depend on the code being migrated and how tightly they are coupled.
2. **Breaking Changes:** Identify API methods, data structures, or behaviors in the source that have no direct equivalent in the target.
3. **Data Migration Risks:** Check for schema changes, state persistence, or side-effects that could lead to data loss during transition.
4. **API Surface Changes:** Determine how public interfaces need to change to accommodate the new system.
5. **Testing Gap Analysis:** Evaluate the quality and coverage of existing tests to ensure they can verify the migration's success.
6. **Performance Impact:** Estimate if the target system introduces new overhead or improves latency.

## Files to Load
Load ALL of these files at once using read_many_files before starting:
- [PLACEHOLDER: absolute path to relevant file]
- [PLACEHOLDER: add more paths]

## Search Patterns
Use ripgrep to identify migration targets and risks:
- **Deprecated/Legacy Identifiers:** `\b(deprecated|obsolete|legacy|old|TODO_MIGRATION)\b`
- **Version Strings/Imports:** `['"](v\d+|\d+\.\d+\.\d+)['"]` in configuration or headers.
- **Direct Usage of Target Library:** `import .* from ['"][PLACEHOLDER: target_library]['"]` (Check for existing partial migrations)
- **Complex Side-Effects:** `\b(useEffect|hook|listener|onEvent|emitter\.on)\b` (Find reactive logic that might break)
- **Environment Variables/Config:** `process\.env\.[A-Z0-9_]+` (Identify dependencies on specific runtime configs)

## Constraints
- Read and analyze files only — do not modify source code
- Write output only to the path specified in the ## Output section
- Do not ask for confirmation before writing the output file

## Output
Write result to: [PLACEHOLDER: <project-root>/.ai-team/outputs/gemini-<task-id>-output.md]
Format: markdown report with the following sections:
1. **Current State Analysis:** Summary of the existing architecture and its limitations.
2. **Risk Matrix:** A table evaluating risks by Impact and Probability.
3. **Candidate Implementation Strategy:** A phased roadmap showing technical dependencies and options, leaving final sequencing to the Coordinator.
   - **Phase 1 (Preparation):** Refactoring to isolate the legacy component.
   - **Phase 2 (Parallel Run/Beta):** Running both systems side-by-side if applicable.
   - **Phase 3 (Full Cutover):** Decommissioning the old system.
4. **Testing Strategy:** How to verify correctness at each phase.
5. **Rollback Considerations:** Steps to take if the migration fails mid-way.

## Done When
Output file exists at the specified path

## BEFORE YOU EXIT
Your final action MUST be writing the output file to the .ai-team/outputs/ path above. Do not ask for confirmation.
