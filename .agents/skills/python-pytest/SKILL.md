---
name: python-pytest
description: >-
  Guidelines and patterns for authoring, structuring, and running pytest tests in this repository. USE FOR: writing unit and integration tests, creating test fixtures in conftest.py, mocking external APIs (WeatherAPI, NMBS/iRail) and Azure services (Blob Storage, Service Bus), and verifying test coverage. DO NOT USE FOR: authoring production application code or deploying services.
---

# Python Pytest Guidelines

Comprehensive guide for writing reliable, isolated, and maintainable automated tests using pytest for the IT Integration project.

## When to Use

- Writing unit tests for data transformation, clients, formatters, and models.
- Writing integration tests for Azure Functions blueprints and triggers with simulated payloads.
- Implementing reusable test fixtures and mocks in test suites.
- Running test suites and validating code coverage.

## When Not to Use

- Authoring production business logic or Azure Functions code.
- Managing Azure infrastructure deployments.

## Core Testing Principles

Follow Uncle Bob's Clean Code FIRST principles:
- **Fast**: Tests must execute quickly without real network overhead.
- **Independent**: Tests must not depend on execution order or shared state.
- **Repeatable**: Tests must yield identical outcomes in any environment.
- **Self-Validating**: Tests produce a binary pass/fail result with no manual inspection needed.
- **Timely**: Tests should be written alongside or immediately prior to code implementation.

Always structure tests using the AAA (Arrange, Act, Assert) pattern.

## Directory Structure

```text
tests/
├── conftest.py                # Shared fixtures (mock clients, sample payloads)
├── unit/
│   ├── test_models.py         # Payload schemas and Pydantic validation
│   ├── test_clients.py        # WeatherAPI, NMBS/iRail clients with mocked responses
│   └── test_formatters.py     # Adaptive Cards and briefing formatters
└── integration/
    ├── test_fetcher_app.py    # Ingestion timer trigger with mocked external services
    └── test_sender_app.py     # Service Bus trigger with mocked dispatchers
```

## Mocking External Dependencies

Never make live network calls during unit tests. Mock external HTTP endpoints and Azure SDK clients.

### Mocking HTTP Calls (WeatherAPI / iRail)

```python
from unittest.mock import AsyncMock, patch
import pytest

@pytest.mark.asyncio
async def test_weather_client_returns_data():
    mock_response = {
        "current": {"temp_c": 18.5, "condition": {"text": "Partly cloudy"}}
    }
    with patch("httpx.AsyncClient.get", new_callable=AsyncMock) as mock_get:
        mock_get.return_value.status_code = 200
        mock_get.return_value.json.return_value = mock_response
        
        # Verify client handling with mock_get
```

### Mocking Azure Services (Blob Storage & Service Bus)

```python
from unittest.mock import MagicMock

def test_blob_storage_upload():
    mock_blob_service = MagicMock()
    mock_container_client = MagicMock()
    mock_blob_client = MagicMock()

    mock_blob_service.get_container_client.return_value = mock_container_client
    mock_container_client.get_blob_client.return_value = mock_blob_client

    # Verify upload logic with mock_blob_client.upload_blob
```

## Running Tests

Execute tests from the repository root:

```bash
# Run all unit tests
pytest tests/unit -v

# Run with coverage report
pytest --cov=src --cov-report=term-missing tests/
```

## Validation Checklist

- [ ] All external API calls and Azure SDK clients are mocked.
- [ ] Tests follow the AAA structure with clear assertions.
- [ ] No hardcoded secrets, connection strings, or production API tokens are used in tests.
- [ ] Tests run deterministically and pass offline without network access.

## Common Pitfalls

| Pitfall | Solution |
|---|---|
| Live HTTP requests during test runs | Always mock HTTP clients (e.g. `httpx`, `requests`) using `unittest.mock` or `respx`. |
| Inter-dependent test state | Use pytest fixtures with appropriate scopes (`function`) to ensure complete isolation. |
| Missing failure assertions | Assert both happy paths and edge cases (timeouts, 4xx/5xx responses, malformed payloads). |
