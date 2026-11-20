---
name: python-api-client
description: >-
  Guidance for resilient external API clients in the fetcher. USE FOR: implementing or reviewing src/fetcher/clients clients for WeatherAPI, KU Leuven timetable, or iRail/NMBS, including timeouts, retries, non-200 responses, Pydantic validation, logging, and secret handling. DO NOT USE FOR: Azure SDK/Blob/Service Bus code (use azure-functions-python), test authoring (use python-pytest), or domain model design alone.
---

# Python API Clients

Patterns for boundary adapters that call external public APIs in `src/fetcher/clients/`.

## When to Use

- Adding a `<Name>Client` for a new data source.
- Reviewing timeout, retry, status, or validation behaviour of an existing client.
- Deciding where raw API JSON becomes a shared domain model.

## When Not to Use

- Writing Azure Functions triggers or SDK bindings (use `azure-functions-python`).
- Writing tests (use `python-pytest`).
- Defining the shared payload schema itself; extend `src/shared/models` instead.

## Core Rules

- One API per client module named `<name>_client.py`, with one `<Name>Client` class.
- A client returns validated domain models from `src/shared/models`; raw API dictionaries never escape the client.
- Inject the HTTP client, base URL, API key provider, and timeout/retry settings into the constructor. No hidden global state.
- Keep methods verb-named and focused, e.g. `fetch_current_weather`, `fetch_departures`.
- Never hardcode keys or tokens. Read them from environment variables through shared configuration and pass them as parameters or headers.
- If an HTTP library such as `httpx` is used, add it to `requirements.txt`; the repository does not ship with it yet.

## Timeouts and Retries

- Always set both connect and read timeouts. Never rely on library defaults.
- Retry only transient failures: connection errors, timeouts, HTTP 429, and 5xx responses.
- Use a small bounded policy (for example, three attempts with exponential backoff and jitter). Do not retry 4xx responses other than 429.
- Respect `Retry-After` on 429 when present.
- Give up with a domain-specific exception such as `ExternalServiceError` that includes the operation, status code, and attempt count.

```python
from dataclasses import dataclass

import httpx

from src.shared.models.weather import CurrentWeather


@dataclass(frozen=True)
class ClientSettings:
    base_url: str
    api_key: str
    timeout_seconds: float = 10.0
    max_attempts: int = 3


class WeatherClient:
    def __init__(self, http_client: httpx.Client, settings: ClientSettings) -> None:
        self._http_client = http_client
        self._settings = settings

    def fetch_current_weather(self, city: str) -> CurrentWeather:
        response = self._http_client.get(
            f"{self._settings.base_url}/current.json",
            params={"key": self._settings.api_key, "q": city},
            timeout=self._settings.timeout_seconds,
        )
        if response.status_code >= 500 or response.status_code == 429:
            raise ExternalServiceError(
                f"WeatherAPI current weather failed with {response.status_code} for {city}"
            )
        if response.status_code != 200:
            raise ExternalServiceError(
                f"WeatherAPI current weather rejected {city}: {response.status_code}"
            )
        return CurrentWeather.model_validate(response.json()["current"])
```

The snippet shows status handling and validation only; wrap the request in the shared retry helper rather than duplicating retry loops in every client.

## Status Handling and Logging

- Check the status code explicitly before parsing JSON. A 200 with unexpected JSON is still a validation failure.
- Log at `warning` for retries and `error` for final failures, using the root `logging` module.
- Include operation, endpoint name, status code, and attempt number. Never log API keys, full URLs with query secrets, or raw payloads that may contain personal data.
- Never swallow exceptions or return `None` to signal failure; raise a context-rich domain exception.

## Validation

- Define response shapes in `src/shared/models` with Pydantic so fetcher and sender agree on one schema.
- Validate at the boundary and return typed models. Raise `PayloadValidationError` with the endpoint name when validation fails.
- Convert timestamps to timezone-aware UTC values in the model, not in formatters.
- Keep fields the project does not use out of the shared model unless they are needed downstream.

## Testing Seams

- Accept an HTTP client or a small transport protocol, so tests can inject a mock without network access.
- Make timeout and retry settings constructor arguments so tests can use zero-delay retries.
- Cover success, 4xx, 5xx, timeout, malformed JSON, and schema validation failures.

## Validation Checklist

- [ ] Timeout is explicit on every request.
- [ ] Non-200 responses are handled before JSON parsing.
- [ ] Retries are bounded and limited to transient failures.
- [ ] Returned values are validated shared models, not raw dictionaries.
- [ ] Secrets come from configuration and are never logged.
- [ ] Any new dependency is reflected in `requirements.txt`.

## Common Pitfalls

| Pitfall | Solution |
|---|---|
| Retrying every error | Retry only timeouts, connection errors, 429, and 5xx. |
| Logging the full request URL | Log the endpoint name and status; API keys live in query strings. |
| Parsing JSON before checking status | Validate status first, then parse and validate the schema. |
| Duplicating retry loops in each client | Put the retry policy in one shared helper and inject it. |
| Returning raw API JSON to the orchestrator | Validate into a `src/shared/models` model inside the client. |
