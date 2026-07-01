# TODO 05 — Add TimedOut to Catch/Finally runAfter in template flow

**Priority:** LOW — consistency fix. Without it, a Try scope that times out skips the Catch scope
in flows built from the template, while both helper flows already handle TimedOut.
**File to change:** `.claude/skills/power-automate-error-handling/references/flow-error-handling-template.json`
**Type of change:** Two exact string replacements. No judgment required.

## Context

Both helper flows (`flow-get-error-message.json`, `flow-send-notification.json`) use:
- Catch: `"runAfter": { "Try": ["Failed", "TimedOut"] }`
- Finally: `"runAfter": { "Catch": ["Succeeded", "Failed", "Skipped", "TimedOut"] }`

The parent template (`flow-error-handling-template.json`) only has `["Failed"]` and
`["Succeeded", "Failed", "Skipped"]`. If an action in Try times out, Catch is skipped and the
error is never extracted or notified.

## Exact changes

In `references/flow-error-handling-template.json`:

1. Replace (this is the `runAfter` of the `Catch` scope):
   - old: `"runAfter": { "Try": ["Failed"] },`
   - new: `"runAfter": { "Try": ["Failed", "TimedOut"] },`

2. Replace (this is the `runAfter` of the `Finally` scope — note it is formatted across
   multiple lines in the file):
   - old:
     ```
          "runAfter": {
            "Catch": ["Succeeded", "Failed", "Skipped"]
          },
     ```
   - new:
     ```
          "runAfter": {
            "Catch": ["Succeeded", "Failed", "Skipped", "TimedOut"]
          },
     ```

## Verification

1. The file must contain exactly one occurrence of `"Try": ["Failed", "TimedOut"]`.
2. The file must contain exactly one occurrence of `"Catch": ["Succeeded", "Failed", "Skipped", "TimedOut"]`.
3. The file must remain valid JSON.
