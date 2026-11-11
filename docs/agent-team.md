# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate a team of four custom agents that will build Mona's Project Pulse dashboard. All agent definitions live in `.github/agents/`.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Breaks down the request, delegates to the specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, verifies the integrated result, and reports the outcome. It does not implement anything itself. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repository and docs, then produces an implementation plan with ordered steps, file assignments, dependencies, parallel vs. sequential work, edge cases, and validation expectations. It writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code and logic within its assigned file scope, including runnable-app support such as `.vscode/launch.json` (cwd `${workspaceFolder}/app`, opening `index.html`), and validates changes before reporting. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Handles UI/UX, accessibility, and visual design, including polished dashboard styling with project cards, status badges, priority treatment, responsive layout, and CSS hooks like `.dashboard` and `.project-card`. | `.github/agents/designer.agent.md` |

All agents are told not to stage, commit, or push; I control git through Copilot CLI prompts.
