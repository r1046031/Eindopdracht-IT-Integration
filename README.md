# IT Integration Final Project

Repository for the IT Integration final assignment.

> **Current status:** no application code exists yet. The repository contains project guidance, pinned agent skills, and this README.

## Goal

Build an automated integration system that gathers public external data daily, stores the raw payloads, communicates asynchronously, and delivers a formatted daily briefing to a specific e-mail address each morning at 07:00.

## Intended data flow

1. The fetcher's timer trigger fires each morning.
2. The fetcher queries the source APIs and aggregates the results.
3. The fetcher saves the aggregated payload to Azure Blob Storage under a timestamped key.
4. The fetcher publishes an event message to Azure Service Bus.
5. The sender's Service Bus trigger receives the message, formats the briefing, and dispatches it to Teams and/or e-mail.

## Candidate systems

None of these integrations exist yet.

| Role | Candidates |
|---|---|
| Weather | [WeatherAPI](https://www.weatherapi.com/) — current weather, rain forecast, alerts |
| Schedule | KU Leuven timetable API, Nylas, or another calendar API |
| Commute | Belgian Mobility / [iRail NMBS API](https://data.belgianmobility.io/en/index.html) |
| Destination | Microsoft Teams Adaptive Card webhook and/or e-mail (Microsoft Graph or SMTP) |

## Scope

This is a school assignment. Work in this repository is limited to Python code for the fetcher, sender, and shared components: fetching, aggregating, formatting, and delivering the briefing.

The project owner handles Azure resource setup, deployment, publishing, and cloud configuration. This repository does not contain infrastructure-as-code, hosting configuration, CI/CD pipelines, or deployment scripts.

No credentials, connection strings, or API tokens are hardcoded. Secrets come from environment variables, Azure Key Vault, or App Configuration.

## Configuration

- Runtime: Python 3.11+
- Platform: Microsoft Azure
- `.env.example` is the safe placeholder file; it only contains a placeholder for `CONTEXT7_API_KEY`, used by documentation tooling.
- Never commit `.env` or `local.settings.json`. If the code needs a new setting, name the expected environment variable instead of adding hosting configuration.

## Repository layout

| Path | Purpose |
|---|---|
| `AGENTS.md` | Hard rules, scope, and naming conventions |
| `.agents/skills/` | Repository skills; versions are pinned in `.agents/skills-lock.json` |
| `.agents/plugins/project-mcp/` | MCP server configuration |
| `docs/plans/` | Implementation plans |
| `src/fetcher/`, `src/sender/`, `src/shared/`, `tests/` | Planned application code; not created yet |

See `AGENTS.md` for the full rules and conventions.
