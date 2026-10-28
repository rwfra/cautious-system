# Agent team

I will use GitHub Copilot CLI in a Codespace to orchestrate the work of these
custom agents for Mona's Project Pulse dashboard:

| Agent | Target model | Responsibility | Definition |
|---|---|---|---|
| Orchestrator | Claude Opus 5.5 (copilot) | Coordinates the specialist agents, breaks work into phases with explicit file scopes, and verifies the integrated result. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repository and requirements, then produces an implementation plan with steps, dependencies, edge cases, and validation expectations. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements the assigned application logic, follows repository patterns, and validates code changes; can add assigned runnable-app support such as a launch configuration. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Defines and implements the assigned dashboard UX, visual design, accessibility, information hierarchy, and responsive behavior. | `.github/agents/designer.agent.md` |
