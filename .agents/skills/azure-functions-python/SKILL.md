---
name: azure-functions-python
description: >-
  Guidance for Azure Functions Python v2 programming model in this repository. USE FOR: writing or reviewing function_app.py entry points, timer triggers, Service Bus triggers, Blueprints, trigger decorators, binding configuration, and function app dependency or app-setting needs. DO NOT USE FOR: Azure resource provisioning, deployment, host.json or local.settings.json edits, or general Python code style (use clean-code).
---

# Azure Functions Python v2

Practical rules for the fetcher and sender Function Apps in this project. Both apps use the decorator-based v2 model.

## When to Use

- Adding or changing a `src/fetcher/function_app.py` or `src/sender/function_app.py` trigger.
- Choosing decorators, argument names, or binding settings for timer and Service Bus triggers.
- Splitting trigger wiring into Blueprints or deciding where business logic belongs.
- Naming an app setting the owner must configure.

## When Not to Use

- Provisioning Function Apps or other Azure resources; the owner does that.
- Deploying or running `func` commands.
- Editing `host.json` or `local.settings.json`; ask the owner for the required setting name instead.
- Writing clients, formatters, or domain models (use `python-api-client` or `clean-code`).

## Programming Model

- Import `azure.functions as func` and create one `app = func.FunctionApp()` per `function_app.py`.
- Declare each function as a module-level function with `@app.function_name(name="...")` plus exactly one trigger decorator.
- Keep entry points thin: parse the trigger payload, call one service, and return. No fetching, formatting, or sending logic in `function_app.py`.
- Use `func.Blueprint()` only when a single app grows several unrelated functions; register it with `app.register_functions(bp)` in `function_app.py`.
- Add `azure-functions` to `requirements.txt` when the dependency is not already present; keep the file in step with imports.

## Timer Trigger (fetcher)

```python
import logging

import azure.functions as func

from src.fetcher.services.daily_briefing_service import DailyBriefingService

app = func.FunctionApp()


@app.function_name(name="daily_fetch")
@app.schedule(schedule="0 0 7 * * *", arg_name="timer", run_on_startup=False)
def daily_fetch(timer: func.TimerRequest) -> None:
    if timer.past_due:
        logging.warning("Daily fetch timer is past due.")
    DailyBriefingService().run()
```

- Schedules are six-field NCRONTAB expressions: `second minute hour day month day-of-week`.
- Use UTC for the schedule and document the intended local time in the service or documentation, not in the decorator name.
- `run_on_startup=False` prevents a duplicate run after a restart; enable it only when explicitly required.

## Service Bus Trigger (sender)

```python
import json
import logging

import azure.functions as func

from src.shared.models.daily_briefing import DailyBriefing
from src.sender.services.dispatch_service import DispatchService

app = func.FunctionApp()


@app.function_name(name="briefing_dispatch")
@app.service_bus_queue_trigger(
    arg_name="message",
    queue_name="%BRIEFING_QUEUE_NAME%",
    connection="SERVICEBUS_CONNECTION",
)
def briefing_dispatch(message: func.ServiceBusMessage) -> None:
    payload = json.loads(message.get_body().decode("utf-8"))
    briefing = DailyBriefing.model_validate(payload)
    DispatchService().send(briefing)
```

- Use `func.ServiceBusMessage` and decode `get_body()` before validating against a model from `src/shared/models`.
- `queue_name`/`topic_name` may reference an app setting with `%SETTING_NAME%`; `connection` is always the name of an app setting, never a connection string.
- Use `subscription_name` with `@app.service_bus_topic_trigger` for topics.
- The runtime completes a message after a successful return and abandons it on an unhandled exception. Persist the payload to Blob Storage before returning so a later failure does not lose data.
- Design handlers to be idempotent: retries can deliver the same message again. Use `delivery_count` or a stable payload key to avoid duplicate sends.
- Set `cardinality=func.Cardinality.MANY` only when the dispatcher explicitly supports batches.

## Configuration and Security

- Read settings with `os.environ` or `os.getenv` in shared configuration code, not in trigger functions.
- Never log connection strings, queue names containing secrets, webhook URLs, or tokens.
- Ask the owner to create app settings by name; do not edit runtime or hosting configuration files.
- Keep fetcher and sender independent: they communicate only through Service Bus and shared models.

## Testing

- Call the decorated Python function directly with a mocked `func.TimerRequest` or `func.ServiceBusMessage`; no Core Tools needed.
- Patch the service the entry point calls and assert the trigger only wires inputs to that service.
- Follow `python-pytest` for fixtures and mocks.

## Validation Checklist

- [ ] `function_app.py` contains no business logic, only trigger wiring.
- [ ] Each trigger has one trigger decorator and a stable `function_name`.
- [ ] Connection and resource names come from app settings, never literals containing secrets.
- [ ] Service Bus handlers persist payloads before returning and tolerate redelivery.
- [ ] All new dependencies are reflected in `requirements.txt`.

## Common Pitfalls

| Pitfall | Solution |
|---|---|
| Six-field schedule written with five fields | Always include the leading seconds field. |
| Business logic inside the trigger function | Move it into a service and keep `function_app.py` thin. |
| Hardcoded queue name or connection string | Reference app settings via `%NAME%` and a `connection` setting name. |
| Assuming a message is processed exactly once | Make dispatch idempotent and persist before acknowledging. |
| Editing hosting files to fix a setting | Ask the owner for the app setting name instead. |
