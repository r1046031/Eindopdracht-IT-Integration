---
name: project-planning
description: >-
  Create concise, actionable implementation plans for this repository and guide approval-gated work. USE FOR: when asked to make a plan, or before implementing a requested change under AGENTS.md. DO NOT USE FOR: domain-specific planning owned by another skill, such as complex Blazor component decomposition (use plan-ui-change).
---

# Project Planning

Create a short, repository-aware plan that makes the intended work, constraints, approval gate, and verification clear. Use this skill for explicit planning requests and for the plan required before implementing a change in this repository.

## When to Use

- The user asks for an implementation plan.
- A requested change requires a plan and approval under `AGENTS.md`.

## When Not to Use

- A domain-specific skill owns the planning workflow, such as complex Blazor UI decomposition; follow that skill for its specialized plan.
- The user asks a straightforward project question that does not request planning or changes.

## Workflow

1. **Check applicable guidance.** Read relevant repository files and check available skills first. Use an applicable skill before consulting an MCP server; consult a configured MCP server only when relevant and needed. Keep research proportional to the task.
2. **Write a concise plan.** Unless the user explicitly opts out, use Markdown with a title, numbered actionable steps, and `[ ]` status markers. Include constraints, dependencies, and verification. For implementation work, keep steps small enough to report progress accurately.
3. **Include project guidance.** State that the best practices in `AGENTS.md` and `ARCHITECTURE.md` apply. Keep MCP lookups targeted and do not expose credentials or unrelated source.
4. **Save and request approval.** Always save the plan under `docs/plans/` with a descriptive filename, including plan-only requests. Present the saved plan and wait for approval before changing implementation files. If the user asked only for a plan, stop after presenting it and keep the saved plan as the deliverable. If the user opted out of planning, do not create a plan.
5. **Execute approved work.** Follow the approved steps, update checkboxes as each step completes, and report changes and verification truthfully. After any completed step that adds or removes a file, or makes substantial edits to a file, briefly explain what changed and why. Propose a short, imperative commit message and use the available button prompt to ask for explicit yes/no approval before committing; wait for the answer. A yes authorizes only that step's commit, and only that step's changes may be staged. A no means do not commit; continue only if the user otherwise wants the task to continue. Ask permission before any other Git command or Git-related action, including staging outside an approved commit. Once implementation work is complete, remove its plan file unless the user asked to keep it. Do not mark a step complete if it failed or remains unverified.

## Plan Format

```markdown
# <Short task title>

## Implementation plan

1. [ ] **<Action>** — <brief description and outcome>
2. [ ] **Verify** — <specific validation>

## Constraints and guidance

- Follow the best practices and guidelines in `AGENTS.md` and `ARCHITECTURE.md`.
- Use relevant skills and the configured project MCP server as applicable.
- <Task-specific constraints, tests, and dependencies>
```

## Validation

- [ ] The plan is always saved under `docs/plans/` and contains clear, ordered actions.
- [ ] Project constraints, relevant skills/MCP use, and specific verification are stated.
- [ ] Approval is obtained before implementation changes.
- [ ] Progress markers and verification results reflect what actually completed.

## Common Pitfalls

| Pitfall | Solution |
|---|---|
| Starting implementation after presenting a plan but before approval | Stop and wait for explicit user approval. |
| Using generic steps that do not identify project constraints | Inspect the relevant code and state applicable architecture, testing, and dependency requirements. |
| Treating a plan-only request as permission to implement | Present and save the plan, then stop. |
| Marking validation complete after a failed or unavailable check | Report the failure or limitation accurately and leave the step incomplete. |
| Calling an unrelated MCP server to satisfy a checklist | Use the configured project MCP server only when relevant; state when no MCP lookup is needed. |
