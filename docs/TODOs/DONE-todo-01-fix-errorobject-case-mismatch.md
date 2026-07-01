# TODO 01 — Fix PascalCase/lowercase mismatch in errorObject property access

**Priority:** HIGH — this is a real runtime bug. Error details silently disappear from notifications.
**File to change:** `.claude/skills/power-automate-error-handling/references/flow-send-notification.json`
**Type of change:** Simple find-and-replace-all. No judgment required.

## Context (why this is a bug)

The "Helper - Get Error Message" flow (`references/flow-get-error-message.json`) returns its
response body with **lowercase** property keys: `status`, `actionname`, `errormessage`, `contents`,
`flowname`, `flowlink`. (The PascalCase names like `ErrorMessage` only appear as `title` display
labels in the schema — they are NOT the actual JSON keys.)

The parent flow (`references/flow-error-handling-template.json`) correctly uses lowercase — its
`Compose:_Notification_message` fallback JSON uses `errormessage`/`actionname`/`status`, and its
Terminate action reads `?['status']` and `?['errormessage']`.

But `flow-send-notification.json` reads the `errorObject` trigger input with **PascalCase** keys:
`json(triggerBody()?['errorObject'])?['ErrorMessage']` etc. At runtime these lookups return `null`,
so every notification shows `(no details)` / `—` instead of the actual error.

## Exact changes

In `references/flow-send-notification.json`, perform these three replace-ALL operations
(each pattern occurs multiple times — replace every occurrence):

1. Replace all:
   - old: `json(triggerBody()?['errorObject'])?['ErrorMessage']`
   - new: `json(triggerBody()?['errorObject'])?['errormessage']`

2. Replace all:
   - old: `json(triggerBody()?['errorObject'])?['ActionName']`
   - new: `json(triggerBody()?['errorObject'])?['actionname']`

3. Replace all:
   - old: `json(triggerBody()?['errorObject'])?['Status']`
   - new: `json(triggerBody()?['errorObject'])?['status']`

Expected occurrence counts (verify after replacing):
- `?['errormessage']` — 4 occurrences (2 in the email HTML, 2 in the Teams adaptive card: the
  TextBlock `text` and its `isVisible` expression)
- `?['actionname']` — 3 occurrences (2 in email HTML, 1 in Teams FactSet)
- `?['status']` — 2 occurrences (1 in email HTML, 1 in Teams FactSet)

## Verification

1. Search the file for `errorObject'])?['E` , `errorObject'])?['A` , `errorObject'])?['S` —
   there must be ZERO matches (no remaining PascalCase access on errorObject).
2. Confirm the file is still valid JSON (parse it with any JSON parser).
3. Do NOT change the trigger input name `errorObject` itself — only the property lookups
   inside `json(...)?[...]` expressions.
4. Do NOT change `title` values in any schema — those stay PascalCase.
