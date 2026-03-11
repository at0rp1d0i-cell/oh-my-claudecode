## AI Team

> Research / documentation-heavy project. Gemini for reading and synthesizing large documents, Codex for tooling and automation.

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | Build scripts, automation tools, document processing utilities |
| gemini | yes       | Literature synthesis, large document analysis, cross-referencing, report generation |

### Routing Rules
- Synthesize multiple long documents → gemini
- Build automation scripts, document converters → codex
- Cross-reference sources across many files → gemini
- Web research and synthesis (web_search + web_fetch) → gemini
- Writing decisions, argument structure, editorial choices → stay with Claude (coordinator)

### Codex Config
Model: gpt-5.3-codex
Sandbox: workspace-write

### Gemini Config
Model: gemini-3
Note: Use read_many_files to load all source documents at once before analysis
