<!-- WHEN TO USE: Use this to understand high-level structure, identify technical debt, or verify adherence to architectural patterns. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before dispatching -->

## Task
Analyze the codebase architecture, identify patterns and anti-patterns, specifically looking at [PLACEHOLDER: specific architectural concerns]. Focus on structural integrity, maintainability, and scalability.

## Analytical Dimensions
Evaluate the loaded files across these architectural dimensions:
1. **Coupling and Cohesion:** Assess if modules are overly dependent on each other or if they represent single, well-defined responsibilities.
2. **Circular Dependencies:** Detect imports that loop back, creating fragile and hard-to-test code.
3. **Layer Violations:** Check for violations of the established hierarchy (e.g., UI calling database directly, domain models leaking to API layer).
4. **God Objects:** Identify files or classes that have grown too large and handle too many responsibilities.
5. **Abstractions:** Evaluate if common logic is abstracted appropriately or if there's excessive duplication across modules.
6. **Error Handling:** Check for consistent and robust error propagation and handling strategies.
7. **Performance Hotspots:** Identify patterns likely to cause bottlenecks (e.g., N+1 queries, deep recursion, redundant computations).

## Files to Load
Load ALL of these files at once using read_many_files before starting:
- [PLACEHOLDER: absolute path to relevant file]
- [PLACEHOLDER: add more paths]

## Search Patterns
Use ripgrep to identify structural issues:
- **Deep Nesting (Complexity):** `if\s*\(.*\)\s*\{\s*if\s*\(.*\)\s*\{\s*if\s*\(.*\)\s*\{\s*if\s*\(.*\)`
- **Layer Violations (Cross-Layer Imports):** `import .* from ['"]\.\./\.\./(db|api|services|models)['"]` (Search from UI/View layers)
- **Circular Dependency Indicators:** `import .* from ['"]\.\/.*['"]` (Look for mutual file-to-file imports)
- **Large Logic Blocks (Switch Case):** `(switch|if|else if){5,}` (Looking for complex dispatching)
- **Duplication (Manual Data Copy):** `\.map\(.*\{.*return\s*\{.*` (Identify manual object mapping that should be centralized)

## Constraints
- Read and analyze files only — do not modify source code
- Write output only to the path specified in the ## Output section
- Do not ask for confirmation before writing the output file

## Output
Write result to: [PLACEHOLDER: <project-root>/.ai-team/outputs/gemini-<task-id>-output.md]
Format: markdown report with the following sections:
1. **Component Overview:** Briefly describe the high-level components and their interactions.
2. **Architecture Diagram:** A Mermaid-compatible text representation of the core module relationships.
3. **Issues by Severity:** Categorized list of anti-patterns found (High/Medium/Low priority).
4. **Positive Patterns:** Note what is working well and should be maintained.
5. **Candidate Actions:** A list of technical options with trade-offs, leaving the final choice to the Coordinator. Do not prescribe a single course of action.

## Done When
Output file exists at the specified path

## BEFORE YOU EXIT
Your final action MUST be writing the output file to the .ai-team/outputs/ path above. Do not ask for confirmation.
