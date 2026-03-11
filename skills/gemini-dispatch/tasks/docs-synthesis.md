<!-- WHEN TO USE: Use this when Gemini should read code plus scattered documentation fragments and synthesize a coherent documentation artifact for a specific audience. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before dispatching -->

## Task
Synthesize documentation for [PLACEHOLDER: feature/system/topic] by reading the provided code and documentation fragments, then produce a single coherent markdown document for [PLACEHOLDER: target audience].

## Documentation Goal
- Target audience: [PLACEHOLDER: developers | operators | contributors | end users | mixed]
- Primary objective: [PLACEHOLDER: what this document should help the reader understand or do]
- Scope to cover:
  - [PLACEHOLDER: capability, subsystem, or workflow in scope]
  - [PLACEHOLDER: second in-scope area]
- Explicitly exclude:
  - [PLACEHOLDER: adjacent topic or internals to omit]
  - [PLACEHOLDER: second exclusion]
- Include:
  - [PLACEHOLDER: required concepts, workflows, APIs, config, examples, or caveats]
  - [PLACEHOLDER: second required inclusion]
- Format requirements:
  - [PLACEHOLDER: required heading structure or section names]
  - [PLACEHOLDER: tone/style constraints]
  - [PLACEHOLDER: length, examples, tables, or reference requirements]

## Files to Load
Load ALL of these files at once using read_many_files before starting:
- [PLACEHOLDER: absolute path to source file]
- [PLACEHOLDER: absolute path to another source file]
- [PLACEHOLDER: absolute path to existing README, ADR, design note, or doc fragment]
- [PLACEHOLDER: add more absolute paths as needed]

## Search Patterns
Use ripgrep only to resolve gaps, confirm identifiers, or find supporting references:
- **Public Interfaces / Entry Points:** `export\s+(async\s+)?(function|class|const)|module\.exports|app\.(get|post|put|delete)`
- **Configuration / Flags:** `process\.env\.[A-Z0-9_]+|--[a-z0-9-]+`
- **Error / Warning Paths:** `throw new|logger\.(error|warn)|console\.(error|warn)`
- **Domain Terms:** `[PLACEHOLDER: project-specific identifier or concept to trace]`

## Constraints
- Read and analyze files only — do not modify source code or existing docs
- Reconcile conflicting fragments by favoring what the code currently does, and note meaningful discrepancies in the output
- Write output only to the path specified in the ## Output section
- Do not ask for confirmation before writing the output file

## Output
Write result to: [PLACEHOLDER: <project-root>/.ai-team/outputs/gemini-<task-id>-output.md]
Format: markdown document with the following sections:
1. **Audience and Scope:** State who this document is for and what it covers.
2. **Overview:** Summarize the system/feature and why it exists.
3. **Key Concepts or Components:** Explain the main moving parts using names from the codebase.
4. **How It Works:** Describe the relevant workflows, control flow, or lifecycle in a way the target audience can follow.
5. **Usage or Operational Guidance:** Include commands, configuration, extension points, or examples that the target audience needs.
6. **Caveats and Non-Goals:** Call out important limitations, assumptions, or excluded areas.
7. **Source Notes:** List the primary files consulted and any conflicts or gaps between code and existing docs.

## Done When
Output file exists at the specified path and satisfies the declared audience, scope, inclusion/exclusion, and format requirements

## BEFORE YOU EXIT
Your final action MUST be writing the output file to the .ai-team/outputs/ path above. Do not ask for confirmation.
