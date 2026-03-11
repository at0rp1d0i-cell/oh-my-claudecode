## AI Team

> CLI tool / library project. Codex-primary with Gemini for API design review and docs analysis.

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | Core implementation, test-fix loops, CLI command scaffolding, refactoring |
| gemini | yes       | API design review, large dependency analysis, compatibility audits |

### Routing Rules
- Implement new commands, flags, core logic → codex
- Review entire public API for consistency → gemini
- Analyze all dependencies for vulnerabilities → gemini
- Test generation for specific functions → codex
- API design decisions, breaking changes → stay with Claude (coordinator)

### Codex Config
Model: gpt-5.3-codex
Sandbox: workspace-write
Done criteria: always include `npm test` or equivalent

### Gemini Config
Model: gemini-3
Preferred tasks: loading entire src/ + test/ for API consistency review
