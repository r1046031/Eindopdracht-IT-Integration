# AGENTS

Project: IT Integration Eindopdracht

- **Language / Runtime**: Python 3.11+
- **Platform**: Microsoft Azure (Cloud environment)
- **Goal**: Automated integration system that gathers external public data daily, stores incoming payloads, communicates asynchronously, and delivers a formatted daily briefing message to a user destination.
- **Architecture & Design**: Detailed in `ARCHITECTURE.md`.

---

## System Overview & Requirements

### 1. Source Systems (Public APIs)

The system integrates at least two external public services to collect daily information:

- **Weather Service**: [WeatherAPI](https://www.weatherapi.com/) (Current weather, rain forecast, weather alerts/warnings).
- **Schedule / Calendar Service**: [KU Leuven Timetable API](https://admin.kuleuven.be/icts/services/dataservices/api/uurrooster) or Nylas / Calendar API for class and schedule updates.
- **Commute / Public Transport Service**: [Belgian Mobility / iRail NMBS API](https://data.belgianmobility.io/en/index.html) for commute and train connection calculations.

### 2. Destination System

Delivers a consolidated, user-friendly daily briefing:

- **Primary Destination**: Microsoft Teams (Adaptive Card webhook) and/or Email (Outlook / Microsoft Graph / SMTP).

### 3. Azure Cloud Components

- **Decoupled Architecture**: Strict separation between the data fetcher/aggregator and the message dispatcher/sender. Logic must never be bundled into a single monolithic component.
- **Compute Resources**: Azure Functions (or Azure Container Apps) in Python:
  - `Fetcher / Ingestion Component`: Triggered periodically (e.g., Timer trigger at morning start), queries external APIs, persists raw payload, and publishes an event message.
  - `Sender / Delivery Component`: Triggered by incoming queue/topic messages, formats the final briefing (e.g., Adaptive Cards or HTML), and dispatches it to the destination.
- **Message Broker**: **Azure Service Bus** (Queue or Topic) for asynchronous decoupled communication between the fetcher and sender.
- **Persistence**: **Azure Blob Storage** to persist incoming payloads (raw API snapshots / aggregated data) for auditability and history.

---

## Architecture & Code Organization

```
Eindopdracht-IT-Integration/
├── .agents/
│   ├── plugins/
│   │   └── project-mcp/
│   │       ├── mcp_config.json   # MCP server config (Context7, Fetch)
│   │       └── plugin.json       # MCP plugin manifest
│   └── skills/                   # Agent skills (to be added)
├── .mcp.json                     # Workspace MCP configuration
├── src/
│   ├── fetcher/                  # Azure Function / Worker for data fetching & aggregation
│   │   ├── clients/              # API clients (WeatherAPI, NMBS/iRail, Calendar)
│   │   ├── services/             # Orchestration & business logic
│   │   └── function_app.py       # Timer Trigger entry point
│   ├── sender/                   # Azure Function / Worker for formatting & delivery
│   │   ├── formatters/           # Message formatters (Adaptive Cards, Email HTML)
│   │   ├── channels/             # Dispatchers (Teams webhook, Email client)
│   │   └── function_app.py       # Service Bus Trigger entry point
│   └── shared/                   # Shared models, utilities, and configuration
│       ├── models/               # Domain models & payload schemas
│       └── storage/              # Azure Blob Storage helper clients
├── docs/
│   └── plans/                    # Agent implementation plans
├── tests/                        # Unit and integration tests
├── requirements.txt              # Shared Python dependencies
├── ARCHITECTURE.md               # Architectural decision record & diagram
└── AGENTS.md                     # Agent operating rules & instructions
```

---

## Rules

- **Separation of Concerns**: Fetching data and sending messages are distinct components communicating strictly via Azure Service Bus.
- **Dependencies & Secrets**: Never hardcode credentials, connection strings, or API tokens. All secrets must be loaded via environment variables / Azure Key Vault / App Configuration.
- **Persistence**: Every incoming aggregated payload must be saved to Azure Blob Storage with a timestamped key before being acknowledged or dispatched.
- **Resilience & Error Handling**: External API calls must handle timeouts and non-200 responses gracefully with proper logging and retry policies.

---

## Skills

- Agent skills live in `.agents/skills` (pinned by `skills-lock.json`).
- Always use the skill `grill-me` or `grilling` when you are asked a question about the project. The `grill-me` skill is a general-purpose skill for asking questions about the project, and `grilling` is a more specific skill for asking questions about the codebase.

---

## MCP Servers

MCP servers are configured in `.agents/plugins/project-mcp/mcp_config.json` and mirrored in `.mcp.json`:

- **Context7 (`context7`)**:
  - **Purpose**: Fetches real-time, version-specific documentation and code snippets directly from official documentation (e.g. Azure Functions Python v2 programming model, Azure Service Bus SDK, Azure Storage Blob, Pydantic, HTTP libraries). Eliminates hallucinations and outdated API usage.
  - **Command**: `npx -y @upstash/context7-mcp`
  - **Environment Variables**: `CONTEXT7_API_KEY` (optional; reads user API key from environment when configured).
  - **Remote Endpoint**: `https://mcp.context7.com/mcp` (can be used as an alternative Streamable HTTP endpoint).
  - **Usage Guideline**: Whenever writing code for external libraries or cloud SDKs, query Context7 to verify exact method signatures, parameter names, and configuration schemas.

- **Fetch (`fetch`)**:
  - **Purpose**: Fetches external documentation and web pages, converting them to clean markdown for fast context inspection (e.g., NMBS/iRail API docs, WeatherAPI schemas, timetable endpoints).
  - **Command**: `npx -y @modelcontextprotocol/server-fetch`

---

## Agent Interaction Policy

- **Approval before changes**: Before executing any change, produce a short implementation plan and obtain approval.
- **Permission before commits**: After completing each plan step, present the changes and ask for permission before committing.
- **Commit style**: Commits should be concise, imperative, and you must ask permission before running any git commands.

### Making a Plan

When you are asked to make a plan, produce a short implementation plan in markdown:

- The plan should be concise, clear, and actionable.
- It should include a numbered list of steps, each with a brief description of the intended change.
- The plan should also include any relevant constraints or guidance, such as coding standards, testing requirements, or dependencies on other work.
- Each step should have an indicator to show if it has been completed or not. For example: `[ ]` for not started, `[x]` for completed.
- Each plan should contain a message stating:
  > _Best practices and guidelines from `AGENTS.md` must be followed, and you have to use skills and the MCP servers._
