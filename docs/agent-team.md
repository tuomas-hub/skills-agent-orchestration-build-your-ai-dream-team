# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate the custom agents that build Mona's Project Pulse dashboard.

:: AGENTS ::

Agent: Orchestrator  
Target Model: Claude Opus 5.5 (copilot)  
Responsibility: Breaks down the request, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, verifies the integrated result, and reports the outcome. Does not implement.  
Definition: .github/agents/orchestrator.agent.md

Agent: Planner  
Target Model: Claude Opus 5.5 (copilot)  
Responsibility: Researches the repo and docs, and returns an implementation plan with ordered steps, file assignments, dependencies, edge cases, and validation expectations. Writes no code.  
Definition: .github/agents/planner.agent.md

Agent: Coder  
Target Model: GPT-5.5 (copilot)  
Responsibility: Implements logic and bug fixes within assigned files, and creates runnable-app support such as .vscode/launch.json (cwd ${workspaceFolder}/app, opening index.html).  
Definition: .github/agents/coder.agent.md

Agent: Designer  
Target Model: Gemini 3.7 Flash (copilot)  
Responsibility: Handles UI/UX, accessibility, and visual design: polished dashboard with project cards, status badges, priority treatment, responsive layout, and .dashboard / .project-card CSS hooks.  
Definition: .github/agents/designer.agent.md

All agents are forbidden from staging, committing, or pushing; I control git through Copilot CLI prompts.
