# TODO 03 — Specify per-channel pruning for Q6 connector choices

**Priority:** HIGH — as written, three of the four Q6 answers produce a broken or failing import.
**File to change:** `.claude/skills/power-automate-error-handling/SKILL.md`
**Type of change:** Replace one section of SKILL.md with a more complete version (exact text provided below).

## Context (why the current instructions are broken)

`references/flow-send-notification.json` contains:
- A top-level `connectionReferences` object with TWO entries
  (`{{PUBLISHER_PREFIX}}_office365_errorhandling` and `{{PUBLISHER_PREFIX}}_teams_errorhandling`)
- A real Office 365 email action (`Send_an_email_V2`) containing token `{{NOTIFICATION_EMAIL_TO}}`
- A real Teams action (`Post_adaptive_card_in_a_chat_or_channel`) containing tokens
  `{{TEAMS_GROUP_ID}}` and `{{TEAMS_CHANNEL_ID}}`

SKILL.md Step 2A currently says "only substitute the ones relevant to Q6 choice" and has a short
"Handling Q6 'Neither'" section. This is insufficient:

- **Q6 = Email only**: the Teams action keeps literal `{{TEAMS_GROUP_ID}}` tokens, and the flow
  JSON still declares the Teams connection reference, which is NOT declared in customizations.xml
  (the skill says to include only the office365 `<connectionreference>` entry) → import fails or
  the flow is created with a broken connection reference.
- **Q6 = Teams only**: same problem mirrored — `{{NOTIFICATION_EMAIL_TO}}` token remains, and the
  office365 connection reference in the flow JSON is undeclared.
- **Q6 = Neither**: BOTH connection references in the flow JSON are undeclared, and all tokens remain.

Rule: whatever connection references the flow JSON declares must ALSO be declared in
customizations.xml, and no `{{...}}` tokens may remain in any written file.

## Exact change

In `SKILL.md`, find the section that begins with the heading:

```
### Handling Q6 "Neither" (no connector wiring)
```

Delete that ENTIRE section (the heading and its paragraph, up to but not including the next
`---` horizontal rule), and replace it with the following text exactly:

```markdown
### Pruning the Send Notification flow to match Q6

The reference file `flow-send-notification.json` ships with BOTH connectors wired. Before writing
the file, prune it so it matches the Q6 answer. The rule: every entry in the flow JSON's top-level
`connectionReferences` object MUST have a matching `<connectionreference>` element in
customizations.xml, and no `{{...}}` tokens may remain in any written file.

**Q6 = Both (C):** Use the reference file as-is. Substitute all tokens
(`{{PUBLISHER_PREFIX}}`, `{{NOTIFICATION_EMAIL_TO}}`, `{{TEAMS_GROUP_ID}}`, `{{TEAMS_CHANNEL_ID}}`).
Include both `<connectionreference>` entries in customizations.xml.

**Q6 = Email only (A):**
1. In the flow JSON's `connectionReferences` object, DELETE the
   `{{PUBLISHER_PREFIX}}_teams_errorhandling` entry (keep the office365 entry).
2. In the TEAMS switch case, REPLACE the entire `Post_adaptive_card_in_a_chat_or_channel` action with:
   ```json
   "Compose_-_PLACEHOLDER_Post_to_Teams": {
     "type": "Compose",
     "inputs": "ADD YOUR POST TO TEAMS ACTION HERE. Use: adaptiveCard=outputs('Compose_-_Teams_Adaptive_Card'). Recommended action: 'Post adaptive card in a chat or channel' (Teams connector).",
     "runAfter": { "Compose_-_Teams_Adaptive_Card": ["Succeeded"] },
     "metadata": { "operationMetadataId": "c1c1c1c1-0001-0001-0001-000000000032" }
   }
   ```
3. Substitute `{{PUBLISHER_PREFIX}}` and `{{NOTIFICATION_EMAIL_TO}}`.
4. In customizations.xml, include ONLY the office365 `<connectionreference>` entry.

**Q6 = Teams only (B):**
1. In the flow JSON's `connectionReferences` object, DELETE the
   `{{PUBLISHER_PREFIX}}_office365_errorhandling` entry (keep the teams entry).
2. In the EMAIL switch case, REPLACE the entire `Send_an_email_V2` action with:
   ```json
   "Compose_-_PLACEHOLDER_Send_Email": {
     "type": "Compose",
     "inputs": "ADD YOUR SEND EMAIL ACTION HERE. Use: subject=outputs('Compose_-_Resolved_Subject'), body=outputs('Compose_-_Email_HTML'), importance=outputs('Compose_-_Severity_Config')?['importance']",
     "runAfter": { "Compose_-_Email_HTML": ["Succeeded"] },
     "metadata": { "operationMetadataId": "c1c1c1c1-0001-0001-0001-000000000022" }
   }
   ```
3. Substitute `{{PUBLISHER_PREFIX}}`, `{{TEAMS_GROUP_ID}}`, `{{TEAMS_CHANNEL_ID}}`.
4. In customizations.xml, include ONLY the teams `<connectionreference>` entry.

**Q6 = Neither (D):**
1. Set the flow JSON's top-level `connectionReferences` to an empty object: `"connectionReferences": {},`
2. Apply BOTH action replacements from the Email-only and Teams-only cases above
   (both connector actions become placeholder Compose actions).
3. Substitute `{{PUBLISHER_PREFIX}}` if it still appears anywhere; no other tokens should remain.
4. In customizations.xml, OMIT the `<connectionreferences>` block entirely.

**Final check for every Q6 choice:** search all files about to be zipped for the substring `{{` —
there must be zero matches before zipping.
```

## Verification

1. SKILL.md must no longer contain the text `Handling Q6 "Neither"`.
2. SKILL.md must contain the new heading `### Pruning the Send Notification flow to match Q6`.
3. The four Q6 cases (Both / Email only / Teams only / Neither) must each be described with
   numbered steps.
