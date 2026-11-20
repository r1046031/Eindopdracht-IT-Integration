---
name: grilling
description: >-
  Perform in-depth codebase inspection, trace execution paths, verify architecture boundaries, and audit implementation details. USE FOR: when asked a technical question about the codebase, inspecting existing code, verifying decoupled boundaries, auditing secret handling, or tracing data flow between Azure components. DO NOT USE FOR: high-level project requirement discovery (use grill-me) or writing code changes directly.
---

# Grilling (Codebase Inspection & Architecture Audit)

Perform systematic inspection and interrogation of the codebase to answer technical questions accurately, verify architecture rules, and audit code quality against repository standards.

## When to Use

- Answering questions about existing codebase structure, implementations, or dependencies.
- Tracing data flow across Azure Functions components (Fetcher -> Blob Storage -> Service Bus -> Sender).
- Auditing code against `AGENTS.md` rules (decoupling, secret management, persistence, error resilience).
- Reviewing pull requests or checking test coverage of specific modules.

## When Not to Use

- High-level project discovery and requirement elicitation before architecture is decided (use `grill-me`).
- Making code edits directly without following `project-planning` and `committing`.

## Workflow

1. **Locate source files.** Locate the relevant files in `src/`, `tests/`, or configuration files using file search tools.
2. **Inspect exact lines.** Read the actual implementation lines rather than assuming behavior. Trace imports, call chains, and exception handling.
3. **Verify architectural constraints:**
   - **Decoupled Architecture**: Ensure no direct imports or monolithic bundling between `src/fetcher/` and `src/sender/`.
   - **Persistence**: Ensure raw payloads are persisted to Azure Blob Storage before emitting events to Azure Service Bus.
   - **Zero Hardcoded Secrets**: Ensure no API keys, credentials, or connection strings exist in code.
   - **External Resilience**: Check that HTTP calls have explicit timeouts and retry policies.
4. **Deliver evidence-based findings.** Provide clear answers with file references and line numbers (e.g., `[src/fetcher/clients/weather.py:15-30](file:///...)`).

## Codebase Audit Matrix

| Check | Key Verification |
|---|---|
| **Fetcher Isolation** | Only depends on `src/shared/`, external clients, and Blob Storage/Service Bus publisher. Never imports `src/sender/`. |
| **Sender Isolation** | Triggered only via Service Bus message; consumes payloads and dispatches to channels. Never fetches raw external APIs. |
| **Pydantic Validation** | Ingestion payloads and briefing models are strictly typed and validated with Pydantic schemas in `src/shared/models/`. |
| **Storage Audit** | Blobs stored with ISO 8601 timestamped hierarchical keys (e.g. `raw/weather/YYYY/MM/DD/...`). |

## Validation Checklist

- [ ] Every assertion is backed by actual file inspection and clickable file links.
- [ ] Decoupling violations or leaked secrets are flagged immediately.
- [ ] Findings distinguish clearly between existing implementation and proposed improvements.
