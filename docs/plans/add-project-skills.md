# Add Project Skills

## Implementation plan

1. [x] **Add project-planning skill** — Create `.agents/skills/project-planning/SKILL.md` with the custom planning workflow and approval-gated instructions.
2. [ ] **Add committing skill** — Create `.agents/skills/committing/SKILL.md` to enforce single-change commit cadence, imperative messaging, and explicit user approval before staging or committing.
3. [ ] **Add grill-me skill** — Create `.agents/skills/grill-me/SKILL.md` for project-level discovery, interview-based requirements elicitation, and design interrogation.
4. [ ] **Add grilling skill** — Create `.agents/skills/grilling/SKILL.md` for codebase-specific structural inspection, trace analysis, and architectural audit.
5. [ ] **Add clean-code skill** — Create `.agents/skills/clean-code/SKILL.md` adhering to Uncle Bob (Robert C. Martin) Clean Code and SOLID principles tailored for Python 3.11+ and Azure cloud integrations.
6. [ ] **Add python-pytest skill** — Create `.agents/skills/python-pytest/SKILL.md` with guidelines for pytest, fixtures, parameterized testing, and mocking external HTTP APIs (WeatherAPI, iRail) and Azure SDKs.
7. [ ] **Add azure-functions-python skill** — Create `.agents/skills/azure-functions-python/SKILL.md` for Azure Functions Python v2 programming model (Timer triggers, Service Bus triggers, Blueprints, and configuration).
8. [ ] **Add python-api-client skill** — Create `.agents/skills/python-api-client/SKILL.md` for resilient external API client implementation (Pydantic validation, timeout handling, retry policies, and error encapsulation).
9. [ ] **Create skills-lock.json** — Create `.agents/skills-lock.json` to track and pin workspace skills.
10. [ ] **Verify** — Validate directory structure, frontmatter schemas (`name`, `description`), and accessibility.

## Constraints and guidance

- _Best practices and guidelines from `AGENTS.md` must be followed, and you have to use skills and the MCP servers._
- Follow the best practices and guidelines in `AGENTS.md` and `ARCHITECTURE.md`.
- Skills must follow the Antigravity skill structure (`.agents/skills/<skill-name>/SKILL.md`) with valid YAML frontmatter (`name`, `description`).
- Every skill should provide concise, practical instructions with progressive disclosure where relevant.
- Explicit approval is required before implementing these changes or running git commands.
