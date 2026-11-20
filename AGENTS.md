# AGENTS

Project: IT Integration Eindopdracht

- Runtime: Python 3.11+
- Platform: Microsoft Azure
- Goal: an automated integration system that gathers external public data daily, stores the raw payloads, communicates asynchronously, and delivers a formatted daily briefing to a user destination.

## Hard rules

Note: not all skills have been added yet. see `docs/plans/add-project-skills.md`

- Never read, search, print, or otherwise access the contents of `.env` files or `local.settings.json`. Never create or modify them.
- Always use the `grilling` skill when asked a question about this project.
- Before starting a task, check for relevant skills and use the applicable skill before consulting an MCP server. Use an MCP server only when it is relevant and needed for the task.
- Follow the [ub-quality skill](.agents/skills/ub-quality/SKILL.md) (Uncle Bob clean code) for clean code best practices whenever you create, modify, or review code or documentation.
- Unless the user explicitly says not to, use the [project-planning skill](.agents/skills/project-planning/SKILL.md) to make a concise implementation plan and get the user's approval before making changes. Save implementation plans under `docs/plans/` and remove the task's plan after the work is complete, unless the user asks to keep it. Keep a plan-only request's plan as its requested deliverable.
- Follow the [committing skill](.agents/skills/committing/SKILL.md) for every added file, deleted file, or significant change to an existing file. Handle each one separately: briefly tell the user what changed and why, draft a short imperative commit message, and ask for an explicit yes/no commit approval using the available button prompt. Wait for the answer before proceeding. A yes authorizes only that change's commit; stage only that change. A no means do not commit and continue only if the user otherwise wants the task to continue. Ask permission before any other Git command or Git-related action, including staging outside an approved commit, branching, pushing, or interacting with GitHub. Ask for explicit permission before making any change to GitHub Actions files.
- Never commit before showing the exact proposed commit message to the user and receiving explicit approval.
- Stay within the scope defined in the Scope section: write only Python code for the fetcher, sender, and shared components. Never deploy, publish, or provision anything.
- Never hardcode credentials, connection strings, or API tokens. Load all secrets from environment variables, Azure Key Vault, or App Configuration.

## Scope

This is a school assignment. The project owner handles everything outside the Python code: Azure resource setup, deployment, publishing, and cloud configuration. The agent only writes the Python code that makes up the briefing system (fetching, aggregating, formatting, and sending the mail or Teams message).

Allowed:

- Python source files under `src/fetcher`, `src/sender`, and `src/shared`.
- Unit and integration tests under `tests/`.
- Keeping `requirements.txt` in step with the code.
- Implementation plans under `docs/plans/`, as required by the project-planning skill.

Not allowed (the owner does these):

- Deploying or publishing: `func azure functionapp publish`, `az` CLI commands, Docker image pushes, or any other release step.
- Creating or changing Azure resources: Service Bus, Blob Storage, Function Apps, Key Vault, App Configuration, and similar. This includes infrastructure-as-code (Bicep, Terraform, ARM templates).
- Writing CI/CD pipelines, GitHub Actions workflows, or deployment scripts.
- Writing or changing `host.json`, `local.settings.json`, or other runtime and hosting configuration. Ask the owner if the code needs a new setting, and name the environment variable it expects.

If a task needs something outside the allowed list, stop and tell the owner what is needed instead of doing it.

## System layout

| Component | Contains | Depends on |
|---|---|---|
| `src/shared` | Domain models and payload schemas (`models/`), Blob Storage helpers (`storage/`), configuration | nothing |
| `src/fetcher` | Timer-triggered Azure Function: API clients (`clients/`), orchestration (`services/`), entry point (`function_app.py`) | shared |
| `src/sender` | Service Bus-triggered Azure Function: formatters (`formatters/`), channel dispatchers (`channels/`), entry point (`function_app.py`) | shared |

Other folders: `tests/` (unit and integration tests), `docs/plans/` (implementation plans), `.agents/` (skills and MCP plugin config), `requirements.txt` (shared dependencies).

## Desired Eventual Data flow

1. The fetcher's timer trigger fires each morning.
2. The fetcher queries the source APIs and aggregates the results.
3. The fetcher saves the aggregated payload to Azure Blob Storage under a timestamped key.
4. The fetcher publishes an event message to Azure Service Bus.
5. The sender's Service Bus trigger receives the message, formats the briefing, and dispatches it to the destination (Teams Adaptive Card and/or email).

## Source and destination systems

- Weather: [WeatherAPI](https://www.weatherapi.com/) (current weather, rain forecast, alerts).
- Schedule: [KU Leuven Timetable API](https://admin.kuleuven.be/icts/services/dataservices/api/uurrooster), or Nylas / a calendar API.
- Commute: [Belgian Mobility / iRail NMBS API](https://data.belgianmobility.io/en/index.html).
- Destination: Microsoft Teams (Adaptive Card webhook) and/or email (Outlook / Microsoft Graph / SMTP).

## Rules

- Fetcher and sender never import from each other. Both may import from `shared`. They communicate only through Azure Service Bus.
- Never bundle fetching and sending into one component.
- Every aggregated payload is saved to Blob Storage with a timestamped key before it is dispatched or acknowledged.
- External API calls must set timeouts, handle non-200 responses, log failures, and follow a retry policy.
- Entry points (`function_app.py`) stay thin: they wire triggers to services and contain no business logic.
- Payload schemas live in `shared/models`. Fetcher and sender both use them; neither defines its own copy.
- Keep `requirements.txt` in step when adding or removing a dependency.

## Naming conventions

| Kind | Convention | Example |
|---|---|---|
| Module / package | `snake_case` | `weather_client.py` |
| Class | `PascalCase` | `WeatherClient` |
| API client | `<Name>Client` in `fetcher/clients` | `NmbsClient` |
| Formatter | `<Name>Formatter` in `sender/formatters` | `AdaptiveCardFormatter` |
| Channel | `<Name>Channel` in `sender/channels` | `TeamsChannel` |
| Payload model | plain noun in `shared/models` | `DailyBriefing` |
| Test file | `test_<module>.py` in `tests/` | `test_weather_client.py` |

## Adding a new source

1. Add or extend the payload model in `shared/models`.
2. Add `<Name>Client` in `fetcher/clients` (timeout, status handling, logging, retry).
3. Call it from the orchestration in `fetcher/services`.
4. Extend the formatters in `sender/formatters` so the data appears in the briefing.
5. Add unit tests for the client and the formatter change.

## Adding a new destination

1. Add `<Name>Channel` in `sender/channels`.
2. Add a matching formatter in `sender/formatters` if the output format differs.
3. Register the channel in the sender's dispatch logic.
4. Add unit tests for the channel and formatter.

## Configuration and secrets

- Local settings live in `local.settings.json` or environment variables. Neither is committed and neither may be read by agents.
- Deployed secrets come from Azure Key Vault / App Configuration, set up by the owner. The code only reads them through environment variables.
- Do not copy credentials, keys, or webhook URLs into documentation, plans, or commit messages.

## Build and run

The agent may run the tests from the repository root:

```bash
pip install -r requirements.txt
pytest tests/
```

The owner runs and deploys the components. Locally this is done with Azure Functions Core Tools (`func start` in `src/fetcher` or `src/sender`). The agent does not run `func` commands or any deployment command.

## Skills

Agent skills live in `.agents/skills` (pinned by `skills-lock.json`).

## MCP Servers

MCP servers are configured in `.agents/plugins/project-mcp/mcp_config.json` and mirrored in `.mcp.json`. Use them only when a skill does not cover the need.

- **Context7 (`context7`)**: real-time, version-specific documentation (Azure Functions Python v2, Azure Service Bus SDK, Azure Storage Blob, Pydantic, HTTP libraries). Query it to verify method signatures, parameter names, and configuration schemas whenever writing code for external libraries or cloud SDKs.
  - Command: `npx -y @upstash/context7-mcp`
  - Env: `CONTEXT7_API_KEY` (optional)
  - Remote alternative: `https://mcp.context7.com/mcp`
- **Fetch (`fetch`)**: converts external web pages to markdown (iRail docs, WeatherAPI schemas, timetable endpoints).
  - Command: `npx -y @modelcontextprotocol/server-fetch`