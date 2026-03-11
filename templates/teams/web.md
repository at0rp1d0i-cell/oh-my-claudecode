## AI Team

> Web project (frontend + backend). Gemini for architecture/review, Codex for feature implementation.

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | React components, API endpoints, test-fix loops |
| gemini | yes       | Full-stack architecture review, dependency audits, large codebase analysis |

### Routing Rules
- Feature implementation (components, endpoints, hooks) → codex
- Architecture review, security audit, large refactoring analysis → gemini
- Test generation for specific functions → codex
- Whole-repo analysis ("find all places where X is used") → gemini
- Design decisions, API design, brainstorming → stay with Claude (coordinator)

### Codex Config
Model: gpt-5.3-codex
Sandbox: workspace-write
Done criteria: always include a specific test command

### Gemini Config
Model: gemini-3
Preferred tasks: one-shot analysis, producing structured reports
