# AGENTS

Project: IT Integration Eindopdracht

- Runtime: Python 3.11+
- Platform: Microsoft Azure
- Goal: an automated integration system that gathers external public data daily, stores the raw payloads, communicates asynchronously, and delivers a formatted daily briefing to a specific e-mail address each morning at 07:00.

## Current status

There is no application code yet.

## Hard rules

- Never read, search, print, or otherwise access the contents of `.env` files or `local.settings.json`. Never create or modify them. `.env.example` is the safe placeholder file.
- Always use the `grilling` skill for questions about the existing codebase or architecture; use `grill-me` for requirement discovery before the design is settled.
- Before starting a task, check the Skills section and use the applicable skill before consulting an MCP server. Use an MCP server only when it is relevant and needed for the task.
- Follow the [clean-code skill](.agents/skills/clean-code/SKILL.md) (Uncle Bob clean code) for clean code best practices whenever you create, modify, or review code or documentation.
- When writing Azure Functions entry points, API clients, or tests, follow [azure-functions-python](.agents/skills/azure-functions-python/SKILL.md), [python-api-client](.agents/skills/python-api-client/SKILL.md), and [python-pytest](.agents/skills/python-pytest/SKILL.md) respectively.
- Unless the user explicitly says not to, use the [project-planning skill](.agents/skills/project-planning/SKILL.md) to make a concise implementation plan and get the user's approval before making changes. Save implementation plans under `docs/plans/` and remove the task's plan after the work is complete, unless the user asks to keep it. Keep a plan-only request's plan as its requested deliverable.
- Follow the [committing skill](.agents/skills/committing/SKILL.md) for every added file, deleted file, or significant change to an existing file. Handle each one separately: briefly tell the user what changed and why, draft a short imperative commit message, and ask for an explicit yes/no commit approval using the available button prompt. Wait for the answer before proceeding. A yes authorizes only that change's commit; stage only that change. A no means do not commit and continue only if the user otherwise wants the task to continue. Ask permission before any other Git command or Git-related action, including staging outside an approved commit, branching, pushing, or interacting with GitHub. Ask for explicit permission before making any change to GitHub Actions files.
- Never commit before showing the exact proposed commit message to the user and receiving explicit approval.
- Stay within the scope defined in the Scope section: write only Python code for the fetcher, sender, and shared components. Never deploy, publish, or provision anything.
- Never hardcode credentials, connection strings, or API tokens. Load all secrets from environment variables, Azure Key Vault, or App Configuration.

## Scope

This is a school assignment. The project owner handles everything outside the Python code: Azure resource setup, deployment, publishing, and cloud configuration. The agent only writes the Python code that makes up the briefing system (fetching, aggregating, formatting, and sending the mail or Teams message).

Not allowed (the owner does these):

- Deploying or publishing: `func azure functionapp publish`, `az` CLI commands, Docker image pushes, or any other release step.
- Creating or changing Azure resources: Service Bus, Blob Storage, Function Apps, Key Vault, App Configuration, and similar. This includes infrastructure-as-code (Bicep, Terraform, ARM templates).
- Writing CI/CD pipelines, GitHub Actions workflows, or deployment scripts.
- Writing or changing `host.json`, `local.settings.json`, or other runtime and hosting configuration. Ask the owner if the code needs a new setting, and name the environment variable it expects.

If a task needs something outside the allowed list, stop and tell the owner what is needed instead of doing it.

## Target data flow (not yet implemented)

1. The fetcher's timer trigger fires each morning.
2. The fetcher queries the source APIs and aggregates the results.
3. The fetcher saves the aggregated payload to Azure Blob Storage under a timestamped key.
4. The fetcher publishes an event message to Azure Service Bus.
5. The sender's Service Bus trigger receives the message, formats the briefing, and dispatches it to the destination (Teams Adaptive Card and/or email).

## Source and destination systems

Candidate systems; none are integrated yet.

- Weather: [WeatherAPI](https://www.weatherapi.com/) (current weather, rain forecast, alerts).
- Schedule: [KU Leuven Timetable API](https://admin.kuleuven.be/icts/services/dataservices/api/uurrooster), or Nylas / a calendar API.
- Commute: [Belgian Mobility / iRail NMBS API](https://data.belgianmobility.io/en/index.html).
- Destination: Microsoft Teams (Adaptive Card webhook) and/or email (Outlook / Microsoft Graph / SMTP).

## Naming conventions

| Kind | Convention | Example |
|---|---|---|
| Module / package | `snake_case` | `weather_client.py` |
| Class | `PascalCase` | `WeatherClient` |
| API client | `<Name>Client` in `fetcher/clients` | `NmbsClient` |
| Formatter | `<Name>Formatter` in `sender/formatters` | `AdaptiveCardFormatter` |
| Channel | `<Name>Channel` in `sender/channels` | `TeamsChannel` |
| Payload model | plain noun in `shared/models` | `DailyBriefing` |
| Test file | `test_<module>.py` in `tests/` | `test_weather_client.py` |.

## Skills

Agent skills live in `.agents/skills` and are pinned by `.agents/skills-lock.json`.

| Skill | Use for |
|---|---|
| `grilling` | Inspecting the codebase, tracing data flow, auditing architecture and secret handling |
| `grill-me` | Requirement discovery and design decisions before implementation |
| `project-planning` | Concise implementation plans and approval-gated work |
| `committing` | Commit approval gate for every added, deleted, or significantly changed file |
| `clean-code` | Uncle Bob clean code and SOLID practices for Python code and documentation |
| `azure-functions-python` | Function App v2 triggers, `function_app.py`, bindings, and app-setting names |
| `python-api-client` | Resilient fetcher clients: timeouts, retries, status handling, Pydantic validation |
| `python-pytest` | Unit/integration tests, fixtures, and mocking external APIs and Azure SDKs |

## MCP Servers

MCP servers are configured in `.agents/plugins/project-mcp/mcp_config.json`. There is no root `.mcp.json` in this repository; treat the plugin config as the source of truth. Use them only when a skill does not cover the need.

- **Context7 (`context7`)**: real-time, version-specific documentation (Azure Functions Python v2, Azure Service Bus SDK, Azure Storage Blob, Pydantic, HTTP libraries). Query it to verify method signatures, parameter names, and configuration schemas whenever writing code for external libraries or cloud SDKs.
  - Command: `npx -y @upstash/context7-mcp`
  - Env: `CONTEXT7_API_KEY` (optional; empty placeholder in `.env.example`)
  - Remote alternative: `https://mcp.context7.com/mcp`
- **Fetch (`fetch`)**: converts external web pages to markdown (iRail docs, WeatherAPI schemas, timetable endpoints).
  - Command: `npx -y @modelcontextprotocol/server-fetch`
