---
name: committing
description: >-
  Enforce the repository commit cadence and approval gate. USE FOR: immediately after adding any file, deleting any file, or making a significant edit such as adding or removing a method, adding a UI element like a small button, or changing behavior, API, or configuration; also before any git commit. DO NOT USE FOR: read-only work, whitespace or comment-only cleanup, generated build output, or changes that are already committed.
---

# Committing

This skill makes every qualifying change pass through one gate: describe the change, propose the exact commit message, get explicit approval, then commit only the approved change. It prevents bundled commits, silent commits, and commits made before the user has seen the message.

## When to Use

- Immediately after a new file is added.
- Immediately after a file is deleted.
- Immediately after a significant change to an existing file: a new method, a removed method, an added UI element even as small as a button, or a behavior, API, configuration, or architecture change.
- Before any commit operation.

## When Not to Use

- Read-only inspection, searching, or answering questions.
- Whitespace-only or comment-only cleanup that changes no behavior.
- Generated build output such as `bin/`, `obj/`, or test result files.
- Work that has already been committed.

## Inputs

| Input | Required | Description |
|---|---|---|
| qualifying change | Yes | The single added file, deleted file, or significant edit just made. |
| approved commit message | No | Only present after the user answers yes; otherwise draft it in step 2. |

## Workflow

1. **Identify one qualifying change.** Use `git status --short` and `git diff`, plus the edits just made. Treat each added file and each deleted file as its own change. Treat a significant edit as its own change. Do not combine multiple files into one change.
2. **Draft one message.** Write a short, imperative commit message that describes only this change, matching the repository's style, for example `add in-memory auth tests`.
3. **Ask for approval.** Use the available button prompt. Show the exact proposed message and ask a clear yes/no question. Do not stage or commit before the answer.
4. **On yes:** stage only the files for this change and commit with the approved message. Verify with `git status --short` and `git log --oneline -1`.
5. **On no, unanswered, or a different answer:** do not stage and do not commit. Report that nothing was committed, and continue only if the user asks.
6. **Stop before further edits.** Wait for the approval decision before making another change.

## Validation

- [ ] The exact commit message was shown to the user before any staging or committing.
- [ ] An explicit yes was received; no commit happened on no, unanswered, or an unrelated answer.
- [ ] Only the approved change was staged; no unrelated files were included.
- [ ] After yes, `git log --oneline -1` shows the approved message.

## Common Pitfalls

| Pitfall | Solution |
|---|---|
| Bundling several new files into one commit | Ask and commit once per added or deleted file. |
| Staging before asking | Show the message and wait for yes first. |
| Vague message such as `updates` | Name the specific change in a short imperative sentence. |
| Treating a small UI addition as trivial | A small button is still a significant change; run the skill. |
| Committing generated output | Exclude `bin/`, `obj/`, and test result artifacts. |
| Continuing to edit while waiting | Stop until the user answers. |
