---
name: grill-me
description: >-
  Conduct an interactive interview to challenge assumptions, resolve design ambiguities, and extract detailed requirements before planning or implementation. USE FOR: when asked a general question about the project, or when aligning on requirements, architecture choices, or domain constraints through interactive grilling. DO NOT USE FOR: deep inspection of existing codebase implementation details (use grilling) or routine file exploration.
---

# Grill-Me (Project Requirements & Design Alignment)

Conduct an interactive, structured interview to probe assumptions, uncover edge cases, and align on system design before committing to code or implementation plans.

## When to Use

- Asked general or open-ended questions about the project, architecture, or features.
- Preparing to plan or implement a feature with ambiguous constraints or trade-offs.
- Aligning on external API integration strategies, message formats, or delivery schedules.

## When Not to Use

- Answering questions about specific existing code, functions, or file paths (use `grilling`).
- Straightforward factual lookups that require no architectural decisions.

## Workflow

1. **Understand the premise.** Identify the feature, integration, or architectural decision in question.
2. **Formulate focused probes.** Ask 1–3 high-value questions at a time. Do not overwhelm the user with exhaustive questionnaires.
   - Clarify edge cases (e.g., API downtime, rate limits, empty responses, invalid tokens).
   - Clarify non-functional requirements (e.g., delivery timing, idempotency, retry thresholds).
   - Propose clear options with a recommended default.
3. **Challenge unverified assumptions.** Identify implicit dependencies or potential points of failure (e.g., Service Bus message size limits, API quotas, authentication rotation).
4. **Iterate and synthesize.** Acknowledge answers, refine the architectural direction, and conclude with a concise summary of agreed decisions before transitioning to `project-planning`.

## Interview Questions Guide

| Area | Focus Questions |
|---|---|
| **Data Sources** | Which endpoints are required? How are rate limits, pagination, and failures handled? |
| **Azure Services** | What tier of Service Bus (Basic vs Standard for topics)? What blob tier and retention policy? |
| **Delivery Channel** | What format (Adaptive Card schema vs HTML email)? Who is the recipient/channel target? |
| **Resilience** | What is the expected retry backoff? How are dead-letter queue (DLQ) messages monitored? |

## Validation Checklist

- [ ] Questions are concise, actionable, and focus on architectural clarity.
- [ ] Options include a clear recommendation prefixed with `(Recommended)`.
- [ ] Requirements and constraints are agreed upon before generating an implementation plan.
