# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Repository:** Session-Error-Handling-in-Power-Automate
**Presenter:** Michael Nielsen (mni@abakion.com)
**Conference:** EPPC 2026
**Topic:** Error handling patterns in Power Automate using the Try-Catch-Finally scope pattern

The repo accompanies a 94-slide EPPC 2026 presentation. The primary deliverable is a Power Automate solution (`ErrorHandling`) deployed to the Ruprect Development environment containing reusable flow templates.

---

## Repository Contents

| File / Folder | Purpose |
|---|---|
| `slides/EPPC 2026 Presentation.pdf` | 94-slide EPPC 2026 conference presentation (PDF export — .pptx is gitignored) |
| `slides/Old presentations/` | Previous slide decks (EPPC25, PPCC25) |
| `presentation-notes.md` | Structured Markdown extraction of all slides, expressions, and speaker notes *(gitignored — local only)* |
| `solutions/` | Importable solution zip files for the conference demo solution (see below) |
| `SOLUTION.md` | Detailed documentation of all flows in the demo solution |
| `README.md` | Public-facing repo overview and installation guide |
| `LICENSE` | Repo license |
| `.env.example` | Template for environment-specific deployment variables |
| `CLAUDE.md` | This file |
| `.claude/skills/power-automate-error-handling/` | Claude skill for scaffolding and deploying this solution |

---

## Power Platform Environment

| Property | Value |
|---|---|
| Environment Name | Development |
| Environment ID | *(see `.env.local` — not committed)* |
| Dataverse URL | *(see `.env.local` — not committed)* |
| Publisher | Ruprect (prefix: `rup`, optionValuePrefix: 91517) |

**Authenticate:** `pac auth create --deviceCode --environment <ENVIRONMENT_ID>`

---

## Solution: ErrorHandling

**Unique name:** `ErrorHandling`
**Display name:** Error Handling
**Current version:** 1.0.4.0
**Import command:** `pac solution import --path ErrorHandling_v2.zip --activate-plugins --force-overwrite`

The solution zip is **not committed to this repo** — it is rebuilt from XML/JSON sources in a temp directory and imported via pac CLI. Rebuild instructions are below.

### Flows in the Solution

#### 1. Error Handling Template
**GUID:** `3fc9a2b1-d4e5-4678-9012-3456789abcde`
**Purpose:** Instant/manual trigger template — drop your business logic in the Try scope.

**Pattern:**
- **Try** — placeholder for business logic (`Placeholder_-_Add_your_business_logic_here`)
- **Catch** — runs on Try failure; calls "Helper - Get Error Message" child flow with `tryResults` and `callerWorkflow`
- **Finally** — if Try succeeded: no-op; if failed: reads error from child flow, builds notification compose, terminates as Failed

The Catch scope is intentionally minimal — one action:
```json
"Run_child_flow_-_Get_Error_Message": {
  "type": "Workflow",
  "inputs": {
    "host": { "workflowReferenceName": "5ae8b3c2-f6d7-4891-b023-456789abcdef" },
    "body": {
      "tryResults": "@{string(result('Try'))}",
      "callerWorkflow": "@{string(workflow())}"
    }
  }
}
```

**Finally** accesses child flow result via `body('Run_child_flow_-_Get_Error_Message')?['ErrorMessage']` and `?['Status']`, with coalesce guards for when the child flow itself fails.

---

#### 2. Helper - Get Error Message
**GUID:** `5ae8b3c2-f6d7-4891-b023-456789abcdef`
**Purpose:** Reusable error extraction. Accepts the caller's Try results and full workflow context; returns a structured error object with FlowName and FlowLink.
**Source:** Translated from user-provided Azure Logic App Bicep template (`helperGetErrorMessage`).

**Trigger:** Manual (Button). All inputs require `"x-ms-dynamically-added": true` for the designer to display them.

| Trigger Input | Caller Expression | Purpose |
|---|---|---|
| `tryResults` (required) | `string(result('Try'))` | Serialized action results array |
| `callerWorkflow` (optional) | `string(workflow())` | Full workflow context — child derives `FlowName` and `FlowLink` from this |

**Why `callerWorkflow`?** `workflow()` inside a child flow refers to the child, not the caller. Passing `string(workflow())` from the parent means all URL-building logic lives in the helper rather than every caller. The child parses it with `json(triggerBody()?['callerWorkflow'])`.

**Try scope — error extraction logic (in order):**
1. `Filter_array_-_Remove_null_values` — filter nulls from parsed `tryResults`
2. `Filter_Array_-_Get_failed_step` — filter for `status == 'Failed'`
3. `Compose_-_Add_root_to_first_result_in_filter_array` — wrap first failed step: `{Root: first(...)}`
4. `Compose_-_Convert_to_XML` — `xml(...)` conversion
5. `Compose_-_Get_error_message_using_XPath` — coalesced XPath: `//error/message[not(preceding-sibling::message)]/text()` falling back to `//message[not(preceding-sibling::message)]/text()`
6. `Select_-_Get_Error_Messages` — union dedup of XPath results
7. `Compose_-_Get_validation_errors` — `first(failed)?['outputs']?['errors']` with `json('[]')` fallback
8. `Select_-_Format_validation_errors` — `concat('Path: ', path, ', SchemaId: ', schemaId)`
9. `Condition_-_Is_there_any_content` → true branch: `Compose_-_Result` with `{Status, ActionName, ErrorMessage, Contents, FlowName, FlowLink}` — FlowName and FlowLink derived from parsed `callerWorkflow`; ErrorMessage joins standard + validation errors with `\r\n`

**Catch scope** (runs if child's own Try fails — meta-error handling):
- Filters `result('Try')` of the child's Try scope
- Basic `//message/text()` XPath as fallback
- Responds with fallback error object

**Finally scope:**
- If child Try succeeded → `Respond_to_a_PowerApp_or_flow` with `outputs('Compose_-_Result')`
- If child Try failed → `Compose_-_Fallback_result` → `Respond_to_a_PowerApp_or_flow_-_Fallback`

**Return schema (both paths):** `{Status, ActionName, ErrorMessage, Contents, FlowName, FlowLink}`

---

#### 3. Helper - Send Notification
**GUID:** `7cd4e5f6-a8b9-4c2d-b1e3-567890abcdef`
**Purpose:** Reusable notification dispatcher. Accepts caller context, channel, severity, and optional error/subject/message; builds channel-specific templates and responds. The actual send action is a placeholder — callers wire up their own Email/Teams connector.
**Source:** Translated from user-provided Azure Logic App Bicep template (`helperSendNotification`).

**Trigger:** Manual (Button). All inputs require `"x-ms-dynamically-added": true`.

| Trigger Input | Type | Required | Description |
|---|---|---|---|
| `callerWorkflow` | string | yes | `string(workflow())` from caller — used to build FlowName and FlowLink |
| `messageService` | string | yes | `EMAIL` or `TEAMS` (case-insensitive, normalized with `toUpper()`) |
| `severity` | string | yes | `INFO`, `WARN`, or `ERROR` (case-insensitive) |
| `errorObject` | string | no | JSON string of error object from "Helper - Get Error Message" output |
| `subject` | string | no | Override for the notification subject line |
| `message` | string | no | Override for the message body text |

**Try scope — actions (in order):**

1. `Compose_-_Severity_Config` — outputs a single config object with all visual properties:
   ```json
   {
     "icon":        "🚨 / ⚠️ / ℹ️",
     "label":       "Error / Warning / Information",
     "headerColor": "#dc3545 / #e67e22 / #0d6efd",
     "accentBg":    "#f8d7da / #fff3cd / #cfe2ff",
     "accentColor": "#721c24 / #6d4c00 / #084298",
     "importance":  "High / Normal / Low",
     "teamsColor":  "attention / warning / accent"
   }
   ```
   Both email and Teams templates reference this via `outputs('Compose_-_Severity_Config')?['headerColor']` etc.

2. `Compose_-_Flow_Name` — `json(triggerBody()?['callerWorkflow'])?['tags']?['flowDisplayName']`
3. `Compose_-_Flow_Link` — same concat pattern as Helper - Get Error Message
4. `Compose_-_Resolved_Subject` — `coalesce(triggerBody()?['subject'], concat(icon, ' ', label, ' — ', flowName))`

5. `Switch_-_Message_Service` on `toUpper(messageService)`:
   - **Case EMAIL:**
     - `Compose_-_Email_HTML` — full HTML email with severity-colored header, error details table, Power Automate link button; error message sourced from `coalesce(json(errorObject)?['ErrorMessage'], triggerBody()?['message'], '(no details)')`
     - `Compose_-_PLACEHOLDER_Send_Email` — instructions string: replace with Office 365 Send Email action using `subject=outputs('Compose_-_Resolved_Subject')`, `body=outputs('Compose_-_Email_HTML')`, `importance=outputs('Compose_-_Severity_Config')?['importance']`
   - **Case TEAMS:**
     - `Compose_-_Teams_Adaptive_Card` — Adaptive Card v1.5 JSON object (not a string) with `teamsColor` Container, severity icon + label header, FactSet with flow/severity/action/status, conditional TextBlock for error message, Action.OpenUrl to FlowLink
     - `Compose_-_PLACEHOLDER_Post_to_Teams` — instructions string: replace with Teams Post Adaptive Card action using `outputs('Compose_-_Teams_Adaptive_Card')` as the card payload

**Catch scope:** `Compose_-_Catch_Error` stores `result('Try')`.

**Finally scope:** Condition on Try status:
- Succeeded → `Respond_to_a_PowerApp_or_flow_-_Success` (200)
- Failed → `Respond_to_a_PowerApp_or_flow_-_Failed` (200)

---

## Demo Solution: ErrorHandlinginPowerAutomate

This is the **conference demo solution** used at PPCC25 and EPPC25. It is separate from the `ErrorHandling` solution built in sessions 1–4. Importable zips are committed to `solutions/`.

**Unique name:** `ErrorHandlinginPowerAutomate`
**Publisher:** Michael Nielsen (prefix: `mni`)
**Versions available:**
- `solutions/ErrorHandlinginPowerAutomate_1_0_0_3.zip` — unmanaged v1.0.0.3
- `solutions/ErrorHandlinginPowerAutomate_1_0_0_3_managed.zip` — managed v1.0.0.3
- `solutions/ErrorHandlinginPowerAutomate_1_0_0_4.zip` — unmanaged v1.0.0.4 (latest)

**Import command:** `pac solution import --path solutions/ErrorHandlinginPowerAutomate_1_0_0_4.zip --activate-plugins --force-overwrite`

### Environment Variables (configure after import)

| Schema Name | Purpose |
|---|---|
| `mni_ErrorHandlerEmail` | Email for fallback notification when error handler itself fails |
| `mni_TestingAPIEndpoint` | Valid API URL for successful demo runs |
| `mni_TestingAPIEndpointInvalid` | Invalid API URL to intentionally trigger errors |

### Connection References

| Connector | Reference Name | Used For |
|---|---|---|
| Dataverse | `mni_sharedcommondataserviceforapps_ad497` | CRUD on `mni_session_errorhandling_cars` table |
| Microsoft Teams | `mni_sharedteams_752c3` | Error notifications |

### Flows in the Demo Solution

| Flow | Type | Description |
|---|---|---|
| `001 No error handling` | Demo | Shows a flow failing with no safety net |
| `002 Basic error handling` | Demo | "Configure run after" on individual actions |
| `003 Advanced error handling` | Demo | Full Try-Catch-Finally with inline Catch |
| `004 Advanced error handling - With child flows` | Demo | Catch calls centralized error handler (030) |
| `005 Flow with ForEach and Switch - Without additional error` | Demo | Error handling in loops/switches |
| `006 Flow with ForEach and Switch and additional error extra` | Demo | Nested Try-Catch inside ForEach |
| `010 Get car details` | Child flow | Calls external API; returns car data |
| `030 Error Handler` | Child flow | Centralized error logger + Teams notifier; has its own Catch → email fallback |
| `040 Send notification` | Child flow | Teams notification dispatcher |
| `999 Template Try Catch Finally scopes` | Template | Ready-to-copy skeleton for new flows |

See `SOLUTION.md` for full per-flow documentation.

---

## Key Design Patterns

### callerWorkflow pattern
Pass the full workflow context from caller to helper — do not construct URLs in the caller:
```
// In the caller (parent flow):
"callerWorkflow": "@{string(workflow())}"

// In the helper (child flow) — derive FlowName:
@json(triggerBody()?['callerWorkflow'])?['tags']?['flowDisplayName']

// In the helper — derive FlowLink:
@concat('https://make.powerautomate.com/environments/',
  json(triggerBody()?['callerWorkflow'])?['tags']?['environmentName'],
  '/flows/',
  json(triggerBody()?['callerWorkflow'])?['name'],
  '/runs/',
  json(triggerBody()?['callerWorkflow'])?['run']?['name'])
```

### x-ms-dynamically-added: true
Every trigger input property in a Manual (Button) flow **must** include this flag or the designer shows "Add an input" instead of the defined inputs:
```json
"myParam": {
  "title": "myParam",
  "type": "string",
  "x-ms-dynamically-added": true,
  "description": "...",
  "x-ms-summary": "myParam"
}
```

### Severity Config as single Compose
Rather than separate templates per severity × channel (6 total), one `Compose - Severity Config` outputs all visual properties. Both email HTML and Teams Adaptive Card reference this compose. Changing a severity color requires editing exactly one action.

### Manual Trigger vs HTTP Trigger for child flows
- **Manual (Button) + Respond to PowerApp or flow** — simpler, no connection required, appears in "Run a child flow" dropdown. Inputs must be flat types (serialize arrays/objects with `string()`). Choose this for internal helpers.
- **HTTP Request + HTTP Response** — supports complex schemas, works across environments, testable with Postman. Choose this for cross-tenant reuse or when the caller is not Power Automate.

---

## Key Expressions Reference

```
// Get all results from Try scope (pass to child flow)
string(result('Try'))

// Pass caller workflow context to child flow
string(workflow())

// Filter for failed steps
@equals(item()?['status'], 'Failed')

// Coalesced XPath — tries error/message first, falls back to message
@coalesce(
  xpath(xml, '//error/message[not(preceding-sibling::message)]/text()'),
  xpath(xml, '//message[not(preceding-sibling::message)]/text()')
)

// Join messages with CRLF
@join(body('Select_-_Get_Error_Messages'), '\r\n')

// Parse callerWorkflow in child flow
@json(triggerBody()?['callerWorkflow'])

// Build flow run link in child flow (using parsed callerWorkflow)
@concat('https://make.powerautomate.com/environments/',
  json(triggerBody()?['callerWorkflow'])?['tags']?['environmentName'],
  '/flows/',
  json(triggerBody()?['callerWorkflow'])?['name'],
  '/runs/',
  json(triggerBody()?['callerWorkflow'])?['run']?['name'])

// Access child flow result in parent
@body('Run_child_flow_-_Get_Error_Message')?['ErrorMessage']
@body('Run_child_flow_-_Get_Error_Message')?['FlowLink']

// Severity-driven config lookup (in Send Notification)
@outputs('Compose_-_Severity_Config')?['headerColor']
@outputs('Compose_-_Severity_Config')?['teamsColor']
```

---

## Deployment: How to Rebuild and Re-import

The solution XML/JSON sources live in a temp build directory during each session. To rebuild from scratch:

1. Create directory structure:
   ```
   ErrorHandling/
   +-- [Content_Types].xml
   +-- solution.xml
   +-- customizations.xml
   +-- Workflows/
       +-- ErrorHandlingTemplate-3FC9A2B1-D4E5-4678-9012-3456789ABCDE.json
       +-- GetErrorMessage-5AE8B3C2-F6D7-4891-B023-456789ABCDEF.json
       +-- HelperSendNotification-7CD4E5F6-A8B9-4C2D-B1E3-567890ABCDEF.json
   ```
2. Use **UTF-8 without BOM** for all files (`New-Object System.Text.UTF8Encoding $false`)
3. Zip using `System.IO.Compression.ZipFile`
4. Import: `pac solution import --path ErrorHandling_v2.zip --activate-plugins --force-overwrite`

**Idempotency:** The GUIDs are fixed. Re-importing the same zip with `--force-overwrite` updates existing flows rather than creating duplicates. To create a new flow (e.g., a renamed variant), assign a new GUID.

**UTF-8 BOM pitfall:** `[System.Text.Encoding]::UTF8` silently adds a BOM that breaks the pac importer. Always use:
```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
[System.IO.File]::WriteAllText($path, $content, $utf8NoBom)
```

---

## pac CLI

Installed as dotnet global tool: `Microsoft.PowerApps.CLI.Tool` v2.8.1+
Update: `dotnet tool update --global Microsoft.PowerApps.CLI.Tool`

---

## What Was Built (Session History)

### Session 1
- Read all 94 EPPC 2026 slides + speaker notes → `presentation-notes.md`
- Authenticated to Development environment via device code
- Created `ErrorHandling` solution (v1.0.0.0) with "Error Handling Template" flow
  - Full Try-Catch-Finally with inline XPath error extraction in Catch scope
  - Finally: condition on Try status → notification compose → Terminate Failed
- Created `.env.example` with all deployment variables
- Fixed UTF-8 BOM bug: `[System.Text.Encoding]::UTF8` adds BOM; always use `New-Object System.Text.UTF8Encoding $false`

### Session 2
- Parsed user-provided Azure Logic App Bicep template for `helperGetErrorMessage`
- Translated Logic App → Power Automate child flow "Helper - Get Error Message" (GUID `5ae8b3c2-f6d7-4891-b023-456789abcdef`)
  - Improvements over v1 Catch: null filtering, coalesced XPath, union dedup, validation error extraction and formatting, meta-Catch scope for child flow self-errors
- Simplified parent "Error Handling Template" Catch scope to a single child flow call
- Discovered: trigger inputs missing in designer — fixed by adding `x-ms-dynamically-added: true` to all trigger input properties
- Refactored from `flowName`/`flowLink` parameters to `callerWorkflow = string(workflow())` — URL logic consolidated in the helper
- Updated solution to v1.0.1.0 with both flows registered; re-imported with `--force-overwrite` → success

### Session 3
- Parsed user-provided Azure Logic App Bicep template for `helperSendNotification`
- Translated to Power Automate helper flow "Helper - Send Notification" (GUID `7cd4e5f6-a8b9-4c2d-b1e3-567890abcdef`)
  - Supports Email and Teams channels via Switch action on `messageService`
  - Supports Info/Warn/Error severity via `Compose_-_Severity_Config` (single config object — all visual properties in one place)
  - Email: full HTML template with severity colors; Teams: Adaptive Card v1.5 with `teamsColor` Container, FactSet, conditional error message, link button
  - Send action is a placeholder compose — audience adds their own connector action
- Updated solution to v1.0.2.0 with all three flows registered

### Session 4
- Changed `messageService` and `severity` trigger inputs in "Helper - Send Notification" from free-text strings to dropdown option sets (`x-ms-content-hint: DROP_DOWN` + `enum`)
  - `messageService`: `["EMAIL", "TEAMS"]`
  - `severity`: `["INFO", "WARN", "ERROR"]`
- Updated solution to v1.0.3.0; re-imported with `--force-overwrite` → success
- Renamed solution: unique name `NewErrorHandling` → `ErrorHandling`, display name `New Error Handling` → `Error Handling`; imported as v1.0.4.0 (creates new solution — old `NewErrorHandling` can be deleted from environment)
- Created `superpowers:power-automate-error-handling` skill with 3 modes (fresh setup, add flow, cross-solution) and full flow JSON reference files

### Session 5
- Migrated repo from `Try-Catch-Finally` to `Session-Error-Handling-in-Power-Automate` (new GitHub repo); files manually copied since the old repo had no GitHub remote
- Added conference demo solution zips to `solutions/` (`ErrorHandlinginPowerAutomate` v1.0.0.3 unmanaged + managed, v1.0.0.4 unmanaged)
- Added `README.md` (public-facing overview), `LICENSE`, and `SOLUTION.md` (full per-flow documentation for the demo solution)
- Reorganized slides into `slides/` subfolder; old decks (EPPC25, PPCC25) moved to `slides/Old presentations/`
- Copied `presentation-notes.md` from old repo (was missing from this repo)
- Updated CLAUDE.md: corrected repo name, updated file paths, added Demo Solution section

### Session 6
- Wired "Helper - Send Notification" into "Error Handling Template" Finally scope (false branch)
  - Added `Run_child_flow_-_Send_Notification` after `Compose:_Notification_message`, before `Terminate:_Flow_failed`
  - Passes `messageService=EMAIL`, `severity=ERROR`, `errorObject=string(outputs('Compose:_Notification_message'))`
  - `Terminate:_Flow_failed` now runs after Send Notification with `["Succeeded", "Failed"]` — flow always terminates red even if notification errors
  - Re-imported solution; updated skill reference JSON and SKILL.md post-import notes
  - Verified both child flow references correct in designer (Get Error Message in Catch, Send Notification in Finally)

---

## Known Limitations / Next Steps

- **Wire up send actions:** In "Helper - Send Notification", replace the two placeholder Compose actions with real connector calls:
  - Email: Office 365 Outlook → Send an Email (V2). Use `subject=outputs('Compose_-_Resolved_Subject')`, `body=outputs('Compose_-_Email_HTML')`, `importance=outputs('Compose_-_Severity_Config')?['importance']`
  - Teams: Post adaptive card in a chat or channel. Use `outputs('Compose_-_Teams_Adaptive_Card')` as the card payload.
- **Child flow wiring in designer:** The `workflowReferenceName` in the parent references children by GUID. If the designer shows a reference as broken, open the "Run a child flow" action and re-select the helper from the dropdown (applies to both Get Error Message in Catch and Send Notification in Finally). *(Verified correct after Session 6 import.)*
- **Add real business logic:** Replace `Placeholder_-_Add_your_business_logic_here` in the Try scope with actual actions.
- **Child flow shown as standalone:** Power Automate may list helper flows as regular flows rather than subprocesses. This is cosmetic — child flow calls still work correctly.
