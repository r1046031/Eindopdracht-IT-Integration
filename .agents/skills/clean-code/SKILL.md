---
name: clean-code
description: >-
  Apply Uncle Bob (Robert C. Martin) Clean Code, SOLID principles, and the Boy Scout Rule tailored for Python 3.11+ and Azure integrations. USE FOR: writing clean Python code, refactoring functions and classes, enforcing Single Responsibility, designing clean boundary adapters for external APIs, and eliminating code smells. DO NOT USE FOR: commit gating (use committing) or planning workflows (use project-planning).
---

# Clean Code (Uncle Bob Principles for Python)

Guidelines and rules based on Uncle Bob (Robert C. Martin) Clean Code and SOLID software craftsmanship, customized for Python 3.11+ and cloud integrations.

## When to Use

- Authoring, reviewing, or refactoring Python code in `src/`.
- Designing functions, classes, data models, and API clients.
- Defining module boundaries between external services (WeatherAPI, iRail) and core business logic.
- Eliminating code smells and technical debt.

## When Not to Use

- High-level project planning (use `project-planning`).
- Git commit approvals and gating (use `committing`).

## Core Principles

### 1. Meaningful Names
- **Intention-revealing**: Variable and function names must declare their intent (e.g. `is_rain_expected` instead of `chk_rn`, `daily_briefing_payload` instead of `data`).
- **Pronounceable & searchable**: Avoid cryptic abbreviations (`temp_celsius` over `tc`, `departure_timestamp` over `deptm`).
- **Class names**: Nouns or noun phrases (`WeatherClient`, `BriefingFormatter`, `TrainScheduleService`).
- **Function names**: Verbs or verb phrases (`fetch_current_weather`, `format_adaptive_card`, `publish_event`).

### 2. Small Functions (Do One Thing)
- **Single Responsibility (SRP)**: A function should do one thing and do it well.
- **Single Level of Abstraction (SLAP)**: Keep all statements within a function at the same abstraction level.
- **Function arguments**: Ideal is 0 to 2 arguments. If a function needs 3+ arguments, bundle them into a Pydantic model or dataclass.
- **No side effects**: Functions should not secretly mutate global state or modify input arguments unexpectedly.

### 3. Clean Boundaries (Third-Party APIs & SDKs)
- **Isolate third-party libraries**: Wrap external APIs (WeatherAPI, NMBS/iRail) and Azure SDKs (`BlobServiceClient`, `ServiceBusClient`) in dedicated client adapter classes in `src/fetcher/clients/` and `src/shared/storage/`.
- **Domain independence**: Domain logic should depend on clean abstractions, not the raw external JSON structures.
- **Dependency Inversion (DIP)**: Inject clients and configurations into orchestrators rather than instantiating them deep within business logic.

### 4. Robust Error Handling
- **Exceptions over return codes**: Raise domain-specific exceptions (e.g. `ExternalServiceError`, `PayloadValidationError`) rather than returning `-1`, `None`, or error strings.
- **Provide context with exceptions**: Include the failing operation, relevant identifiers, and status codes in exception messages.
- **Never swallow exceptions**: Avoid bare `except:` or `except Exception: pass`. Always log or handle expected exceptions explicitly.
- **Don't return or pass None**: Prefer empty collections or raise exceptions instead of returning `None` to avoid nested `if x is not None:` null checks.

### 5. SOLID Principles in Python
- **S** — Single Responsibility: Each class or module has one reason to change.
- **O** — Open/Closed: Extend functionality by adding new classes/modules, not by modifying existing tested code.
- **L** — Liskov Substitution: Derived classes/implementations must be substitutable for their base types.
- **I** — Interface Segregation: Keep protocols (`typing.Protocol`) small and focused.
- **D** — Dependency Inversion: Depend upon abstractions (`Protocol` / base classes), not concrete details.

### 6. The Boy Scout Rule
- **Leave the code cleaner than you found it**: Whenever touching existing files, clean up dead code, rename confusing variables, and improve type hints.

## Clean Code Checklist

- [ ] Functions are short, focused, and have clear verbs as names.
- [ ] Strict type annotations (`typing`, `Pydantic`) are used across public interfaces.
- [ ] External API calls and SDKs are encapsulated behind client adapters.
- [ ] No hardcoded configuration, magic strings, or magic numbers.
- [ ] Error handling uses structured exceptions with contextual messages.
- [ ] Commented-out dead code is removed, not preserved.
