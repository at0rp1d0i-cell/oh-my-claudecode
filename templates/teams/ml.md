## AI Team

> Machine learning / data science project. Gemini for research and large codebase analysis, Codex for training scripts and utilities.

### Workers
| Worker | Available | Strength |
|--------|-----------|---------|
| codex  | yes       | Training scripts, data pipelines, utility functions, experiment configs |
| gemini | yes       | Research synthesis, paper analysis, large dataset schema review, model architecture comparison |

### Routing Rules
- Implement training loop, data loader, evaluation metrics → codex
- Research synthesis, paper cross-referencing, literature review → gemini
- Analyze experiment results across many log files → gemini
- Implement specific model architecture from paper → codex
- Design decisions, experiment strategy → stay with Claude (coordinator)

### Codex Config
Model: gpt-5.4
Sandbox: workspace-write
Note: ML tasks often need complex reasoning; use gpt-5.4

### Gemini Config
Model: gemini-3
Preferred tasks: loading entire src/ for architecture overview, research document analysis
