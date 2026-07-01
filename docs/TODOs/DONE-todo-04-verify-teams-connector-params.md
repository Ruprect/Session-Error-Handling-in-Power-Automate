# TODO 04 — Verify Teams connector operationId/parameters and runtimeSource

**Priority:** MEDIUM — **verify-first**. Do NOT apply the suspected fixes without confirming them
against a real source. If you cannot verify, write your findings at the bottom of this file and stop.
**File to change:** `.claude/skills/power-automate-error-handling/references/flow-send-notification.json`
(and the matching caveat bullet in `SKILL.md` "Critical rules" once resolved)

## Context

The Teams action in `flow-send-notification.json` was written from memory, not from a real
exported flow. Three things are suspect:

1. **operationId** — the file uses `PostAdaptiveCardToConversation`. Real exports of the Teams
   action "Post adaptive card in a chat or channel" are believed to use operationId
   `PostCardToConversation`.

2. **Parameter names** — the file uses:
   - `body/recipient/groupId`
   - `body/recipient/channelId`
   - `body/messageDetails/adaptiveCardContent`

   Real exports are believed to use:
   - `poster` = `Flow bot`
   - `location` = `Channel`
   - `body/recipient/groupId`
   - `body/recipient/channelId`
   - `body/messageBody` = the adaptive card JSON **as a string**

   (i.e. the card content parameter is likely `body/messageBody`, not
   `body/messageDetails/adaptiveCardContent`.)

3. **runtimeSource** — both entries in the flow JSON's top-level `connectionReferences` object use
   `"runtimeSource": "invoker"`. Solution flows that use connection references normally export with
   `"runtimeSource": "embedded"`. `invoker` may cause the connection to not bind on import.

The same `runtimeSource` question applies to the Office 365 entry. The Office 365 email action
itself (`SendEmailV2`, `emailMessage/To|Subject|Body|Importance`) matches real exports and does
NOT need verification.

## How to verify (pick whichever source is available)

**Option A (best):** Export a real flow from the environment that already posts an adaptive card
via the Teams connector (e.g. the demo solution `ErrorHandlinginPowerAutomate` contains
`040 Send notification` which posts to Teams — its zips are in `solutions/` in this repo).
Unzip `solutions/ErrorHandlinginPowerAutomate_1_0_0_4.zip`, open the workflow JSON for
`040 Send notification`, and copy the EXACT `host` block, parameter names, and
`connectionReferences` entry format from it.

**Option B:** Search Microsoft Learn documentation for the Teams connector action
"Post adaptive card in a chat or channel" (`shared_teams`, `PostCardToConversation`) and confirm
the OpenApiConnection parameter paths.

## What to change once verified

1. Update the `Post_adaptive_card_in_a_chat_or_channel` action's `operationId` and `parameters`
   keys in `flow-send-notification.json` to the verified names. Keep the existing expression
   `@{string(outputs('Compose_-_Teams_Adaptive_Card'))}` as the card-content value (the card must
   be passed as a string).
2. Update `"runtimeSource"` on BOTH `connectionReferences` entries to the verified value
   (expected: `"embedded"`).
3. In `SKILL.md`, find the Critical rules bullet that begins `**Teams connector parameter paths**:`
   and rewrite it to state the now-verified parameter names as fact (remove the "may differ /
   compare with a working flow" hedging, but keep one sentence noting values were verified against
   a real export).

## Verification

1. The flow JSON must remain valid JSON.
2. Every parameter path in the Teams action must exactly match the verified export/docs.
3. Note at the bottom of this file which source you used for verification (file path or URL).

## Verification Notes

**Sources used:**
- `solutions/ErrorHandlinginPowerAutomate_1_0_0_4.zip` → extracted `040Sendnotification-2B78A82D-754A-F011-8779-6045BDE007D5.json` — confirmed `runtimeSource: "embedded"`.
  (Note: this flow uses `PostMessageToConversation` for plain-text chat messages, not the adaptive card action; it does not directly confirm the adaptive card operationId.)
- `https://learn.microsoft.com/en-us/connectors/teams` — confirmed:
  - operationId for "Post card in a chat or channel" = **`PostCardToConversation`**
  - Parameters: `poster`, `location`, `body` (dynamic) — sub-keys `body/recipient/groupId`, `body/recipient/channelId`, `body/messageBody` (card JSON string)
  - The old `PostChannelAdaptiveCard` (uses `PostAdaptiveCardRequest` parameter key) is marked **DEPRECATED**

**Changes applied to `flow-send-notification.json`:**
1. `operationId`: `PostAdaptiveCardToConversation` → `PostCardToConversation` ✓
2. `body/messageDetails/adaptiveCardContent` → `body/messageBody` ✓
3. `runtimeSource`: `"invoker"` → `"embedded"` on both connectionReferences entries ✓

**SKILL.md Critical rules bullet update:** Initially left as-is, but a follow-up verification pass
caught that this left the bullet asserting the OLD (wrong) parameter names
(`body/messageDetails/adaptiveCardContent`) as fact, contradicting the corrected reference JSON.
The bullet has now been rewritten per this TODO's instruction: it states the verified
operationId (`PostCardToConversation`), parameters (`poster`, `location`, `body/recipient/groupId`,
`body/recipient/channelId`, `body/messageBody` as string), and `runtimeSource: "embedded"` as fact,
with a note on the verification sources. ✓
