---
name: power-automate-error-handling
description: >
  Use this skill whenever the user wants to set up Power Automate error handling flows, create a
  Try-Catch-Finally pattern in Power Automate, deploy the ErrorHandling solution, add new flows
  to an existing Power Automate solution, create a new solution that references shared helper flows
  from another solution, or scaffold error handling helpers, or add error handling to EXISTING flows
  ("add error handling to existing flows", "retrofit error handling", "wrap my
  flows in try-catch", "add try-catch to flows in a solution"). Triggers on phrases like "set up error
  handling", "create the helper flows", "add a flow to the solution", "new solution referencing
  helpers", "cross-solution", "reference flows from another solution", "deploy the solution",
  "rebuild the zip", or anything involving the Power Automate error handling pattern.
---

# Power Automate Error Handling Setup

Scaffold and deploy the Try-Catch-Finally error handling solution for Power Automate.
The solution contains three flows: **Error Handling Template**, **Helper - Get Error Message**,
and **Helper - Send Notification** — plus an option to add custom flows.

---

## Step 1 — Gather requirements

### Pre-flight: Load environment config

Before asking any questions, load defaults from the project directory:

1. Check for env files in this priority order: `.env` → `.env.local` → `.env.example`. Use the first one found.
2. Parse any line matching `KEY=value` (ignore blank lines and `#` comments).
3. Store the following values as defaults (used to pre-fill prompts below):
   - `ENVIRONMENT_ID` → used in Q5 auth command
   - `PUBLISHER_UNIQUE_NAME` → default for Q2
   - `PUBLISHER_PREFIX` → default for Q2
   - `SOLUTION_UNIQUE_NAME` → default for Q3 (Mode A)
   - `SOLUTION_DISPLAY_NAME` → default for Q3 (Mode A)
4. **Tell the user** which file was loaded, e.g.: "Loaded settings from `.env.local`." If a value is still a placeholder (e.g. contains `YOUR-` or `-HERE`), treat it as missing and note that the user should supply it.

If no env file exists at all, tell the user and ask them to supply values manually.

---

Ask the user these questions **one at a time** (stop and wait for each answer before the next):

**Q1 — Mode:**
> "Do you want to:
> (A) Fresh setup — create the full solution with all helper flows from scratch
> (B) Add flow(s) — add a new Error Handling Template flow to an existing solution
> (C) New solution, shared helpers — create a brand-new solution whose flows call the helper flows that live in a different solution
> (D) Retrofit — add Try-Catch-Finally error handling to existing flows in an existing solution"

**Q2 — Publisher prefix** (all modes):
> "What is the publisher unique name and prefix? (default from env file: publisher=`<PUBLISHER_UNIQUE_NAME>`, prefix=`<PUBLISHER_PREFIX>`)"
>
> Show the values loaded from the env file as the default. If no env file was found, default to publisher=`Ruprect`, prefix=`rup`.
> The prefix is used to filter the solution list so only your solutions are shown as candidates.
> Unique name must be letters/digits/underscores only.

**Q3 — Solution selection:**

- **Mode A**: Ask for the new solution unique name and display name.
  - "What should the solution unique name be? (default from env file: `<SOLUTION_UNIQUE_NAME>`; fallback: `ErrorHandling`)"
  - "What should the display name be? (default from env file: `<SOLUTION_DISPLAY_NAME>`; fallback: `Error Handling`)"

- **Mode B**: Run the solution list command and show filtered results, then ask the user to pick:

  ```powershell
  pac solution list
  ```

  Parse the output and show only solutions whose unique name starts with the publisher prefix
  (case-insensitive, e.g. prefix `rup` → show lines matching `^rup` in the unique name column).
  If none match or the user wants a different filter, show the full list.

  Then ask: "Which solution should the new flow(s) be added to?"

- **Mode C** (two questions):
  - "What is the unique name for the **new** solution? (e.g. `MyBusinessSolution`)" — and display name
  - Run `pac solution list`, filter by publisher prefix, show results.
    Ask: "Which solution contains the helper flows? (default: `ErrorHandling`)"

- **Mode D**: Run `pac solution list` and show ALL solutions EXCEPT Microsoft/system
  ones — exclude unique names starting with `msdyn`, `msft`, `Microsoft`, `Default`,
  `Active`, or `Basic`, and canvas-app auto-solutions (`Cr` + 5 hex chars). Do NOT
  filter by publisher prefix: retrofit targets often belong to other publishers.
  Show the unfiltered list if the user asks.
  Then ask: "Which solution contains the flows to retrofit?"

**Q4 — Flow name(s)** (modes B and C):
> "What is the display name of the new flow? (e.g. `My Business Process`)"
>
> After each name ask: "Add another flow, or done?"

**Q4b — Helper solution** (Modes B and D; for Mode D the default is `ErrorHandling`):
> "Which solution contains the helper flows (`Helper - Get Error Message`,
> `Helper - Send Notification`) that the new flow should call? (default: the same solution
> you are adding the flow to; otherwise typically `ErrorHandling`)"

**Q5 — Confirm pac auth:**

Run `pac auth list` and check whether the active profile's Environment Url matches `ENVIRONMENT_URL`
from the env file (case-insensitive, trailing slash ignored). If it matches, tell the user they are
already connected (e.g. "Already connected to Development — no login needed.") and skip the prompt.

If not connected, show the ready-to-run command using `ENVIRONMENT_ID` from the env file:
> `pac auth create --deviceCode --environment <ENVIRONMENT_ID>`
>
> If no env file was found, show the placeholder and ask the user to supply the environment ID.
> Ask: "Are you authenticated to this environment? Run the command above if not, then confirm when ready."

**Q6 — Notification connectors** (Mode A only):
> "Which connectors should be wired up with real actions in Helper - Send Notification?
>
> (A) Email only — Office 365 Outlook 'Send an Email (V2)'
> (B) Microsoft Teams only — 'Post adaptive card in a chat or channel'
> (C) Both Email and Teams
> (D) Neither — I'll add connector actions manually after import"

**Q7 — Email recipient** (ask only if Q6 answer includes Email — A or C):
> "What email address should error notification emails be sent to?
> (e.g. `ops@example.com` — this is baked into the helper flow and can be changed later in the designer)"

**Q8 — Teams channel** (ask only if Q6 answer includes Teams — B or C):
> "What is the Teams **Group ID** and **Channel ID** for error notifications?
>
> To find these: in Teams, right-click the target channel → 'Get link to channel'. The URL contains:
> `groupId=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` and `channel=19%3Axxxxxxxxx%40thread.tacv2`
>
> URL-decode the channel value (replace `%3A` → `:`, `%40` → `@`) to get the Channel ID.
>
> Enter both values now."

Record all answers before proceeding.

---

## Step 2A — Fresh setup

Read these reference files before writing any files:
- `references/solution-xml.md` — templates for solution.xml, customizations.xml, [Content_Types].xml
- `references/flow-error-handling-template.json` — parent flow JSON
- `references/flow-get-error-message.json` — helper child flow JSON
- `references/flow-send-notification.json` — notification helper JSON

### Generate new GUIDs for this deployment

Always generate fresh GUIDs — **never reuse GUIDs from a previous deployment**. Reusing a GUID
after deleting a flow causes import errors because the environment may still hold orphaned references.

```powershell
$guidTemplate  = [System.Guid]::NewGuid().ToString()   # Error Handling Template
$guidGetError  = [System.Guid]::NewGuid().ToString()   # Helper - Get Error Message
$guidSendNotif = [System.Guid]::NewGuid().ToString()   # Helper - Send Notification

Write-Host "Generated GUIDs (save these if you plan cross-solution references):"
Write-Host "  Error Handling Template : $guidTemplate"
Write-Host "  Helper - Get Error Msg  : $guidGetError"
Write-Host "  Helper - Send Notif     : $guidSendNotif"
```

Show these GUIDs to the user before proceeding — they need them if they later create a Mode C
cross-solution that calls these helpers.

### Compute connection reference names (Mode A only)

```powershell
$publisherPrefix = "<from Q2>"
$crOffice365 = "${publisherPrefix}_office365_errorhandling"
$crTeams     = "${publisherPrefix}_teams_errorhandling"
```

### Directory structure to create

```
$env:TEMP\PA_ErrorHandling\<SolutionName>\
├── [Content_Types].xml
├── solution.xml
├── customizations.xml
└── Workflows\
    ├── ErrorHandlingTemplate-<GUID_TEMPLATE_UPPER>.json
    ├── GetErrorMessage-<GUID_GET_ERROR_UPPER>.json
    └── HelperSendNotification-<GUID_SEND_NOTIF_UPPER>.json
```

Use `$env:TEMP\PA_ErrorHandling` as the base temp directory.

### PowerShell script pattern

```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false

$solutionName = "<from Q3>"
$dir   = "$env:TEMP\PA_ErrorHandling\$solutionName"
$wfDir = "$dir\Workflows"
New-Item -ItemType Directory -Force -Path $wfDir | Out-Null

# --- Read reference JSON files ---
# (paste the contents of each reference file into these variables)
$errorTemplateJson = '<contents of flow-error-handling-template.json>'
$getErrorMsgJson   = '<contents of flow-get-error-message.json>'
$sendNotifJson     = '<contents of flow-send-notification.json>'

# --- Token substitution: GUIDs referenced inside flow JSON ---
# Each flow's own GUID is not embedded in the JSON itself — only cross-references are.
# The Error Handling Template references Get Error Message and Send Notification by GUID
# (in its Catch and Finally scopes). Update those references to the new GUIDs:
$errorTemplateJson = $errorTemplateJson.Replace(
    '"workflowReferenceName": "HELPER_GET_ERROR_GUID"',
    "`"workflowReferenceName`": `"$guidGetError`""
)
$errorTemplateJson = $errorTemplateJson.Replace(
    '"workflowReferenceName": "HELPER_SEND_NOTIF_GUID"',
    "`"workflowReferenceName`": `"$guidSendNotif`""
)

# --- Token substitution: publisher prefix in connection reference names ---
$sendNotifJson = $sendNotifJson.Replace("{{PUBLISHER_PREFIX}}", $publisherPrefix)

# --- Prune to Q6 BEFORE substituting or writing ---
# Apply the per-channel pruning from "Pruning the Send Notification flow to match Q6" (below),
# then substitute only the tokens that remain in the pruned JSON.
$sendNotifJson = $sendNotifJson.Replace("{{NOTIFICATION_EMAIL_TO}}", $notificationEmailTo)   # Q7
$sendNotifJson = $sendNotifJson.Replace("{{TEAMS_GROUP_ID}}",        $teamsGroupId)           # Q8
$sendNotifJson = $sendNotifJson.Replace("{{TEAMS_CHANNEL_ID}}",      $teamsChannelId)         # Q8

# --- Compute file names (GUIDs must be UPPERCASE in file names) ---
$templateFile  = "ErrorHandlingTemplate-$($guidTemplate.ToUpper()).json"
$getErrorFile  = "GetErrorMessage-$($guidGetError.ToUpper()).json"
$sendNotifFile = "HelperSendNotification-$($guidSendNotif.ToUpper()).json"

# --- Write flow JSON files ---
[System.IO.File]::WriteAllText("$wfDir\$templateFile",  $errorTemplateJson, $utf8NoBom)
[System.IO.File]::WriteAllText("$wfDir\$getErrorFile",  $getErrorMsgJson,   $utf8NoBom)
[System.IO.File]::WriteAllText("$wfDir\$sendNotifFile", $sendNotifJson,     $utf8NoBom)

# --- Build XML files (substitute template tokens) ---
# See references/solution-xml.md for full templates.
# Key substitutions:
#   {{GUID_TEMPLATE}}        → $guidTemplate   (lowercase)
#   {{GUID_GET_ERROR}}       → $guidGetError   (lowercase)
#   {{GUID_SEND_NOTIF}}      → $guidSendNotif  (lowercase)
#   {{GUID_TEMPLATE_UPPER}}  → $guidTemplate.ToUpper()   (UPPERCASE for JsonFileName)
#   {{GUID_GET_ERROR_UPPER}} → $guidGetError.ToUpper()
#   {{GUID_SEND_NOTIF_UPPER}}→ $guidSendNotif.ToUpper()
#   {{SOLUTION_NAME}}        → $solutionName
#   {{SOLUTION_DISPLAY_NAME}}→ $solutionDisplayName
#   {{PUBLISHER_NAME}}       → $publisherName
#   {{PUBLISHER_PREFIX}}     → $publisherPrefix
#   {{VERSION}}              → "1.0.0.0"

# Include <connectionreferences> block in customizations.xml when Q6 != "Neither".
# For Email-only: include only the office365 entry.
# For Teams-only: include only the teams entry.
# For Both: include both entries.
# For Neither: omit the entire <connectionreferences> block.

[System.IO.File]::WriteAllText("$dir\[Content_Types].xml", $contentTypesXml, $utf8NoBom)
[System.IO.File]::WriteAllText("$dir\solution.xml",         $solutionXml,    $utf8NoBom)
[System.IO.File]::WriteAllText("$dir\customizations.xml",   $customXml,      $utf8NoBom)

# --- Zip ---
Add-Type -AssemblyName System.IO.Compression.FileSystem
$zipPath = "$env:TEMP\PA_ErrorHandling\${solutionName}.zip"
if (Test-Path $zipPath) { [System.IO.File]::Delete($zipPath) }
[System.IO.Compression.ZipFile]::CreateFromDirectory($dir, $zipPath)

# --- Import ---
pac solution import --path $zipPath --activate-plugins --force-overwrite
```

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

---

## Step 2B — Add flow(s) to an existing solution

This mode works for **any** target solution — not just `ErrorHandling`. The safest approach is to
export the current solution, add the new flow(s) into it, then re-import. This preserves all
existing flows and metadata rather than reconstructing them from scratch.

Read `references/flow-error-handling-template.json` before writing any files.

### 2B-1: Export the current solution

```powershell
$solutionName = "<TargetSolutionUniqueName>"   # from Q3
$exportDir    = "$env:TEMP\PA_ErrorHandling\$solutionName"
$exportZip    = "$env:TEMP\PA_ErrorHandling\${solutionName}_export.zip"

New-Item -ItemType Directory -Force -Path $exportDir | Out-Null
pac solution export --name $solutionName --path $exportZip --overwrite
```

### 2B-2: Unzip the export

```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem
# Remove any previous unpack
if (Test-Path $exportDir\solution.xml) { Remove-Item "$exportDir\*" -Recurse -Force }
[System.IO.Compression.ZipFile]::ExtractToDirectory($exportZip, $exportDir)
```

Inspect the unpacked files to confirm the Workflows folder and existing flow count.

### 2B-3: Generate a GUID for each new flow

```powershell
# Lowercase for solution.xml/customizations.xml; uppercase ONLY in the JSON file name
$newGuid      = [System.Guid]::NewGuid().ToString()
$newGuidUpper = $newGuid.ToUpper()
Write-Host $newGuid
```

### 2B-4: Create the new flow JSON file(s)

Based on `references/flow-error-handling-template.json`:
- Generate fresh `operationMetadataId` GUIDs for every action (avoids collisions)
- Update the `workflowReferenceName` values inside the Catch and Finally scopes to the current
  GUIDs of the installed helper flows. Discover them from the helper solution named in Q4b:
  - If Q4b = the target solution itself: parse the ALREADY-EXPORTED customizations.xml in
    `$exportDir` — find the `<Workflow>` elements whose `Name` contains "Get Error" and
    "Send Notification" and take their `WorkflowId` values (strip the braces).
  - If Q4b = a different solution: export THAT solution separately and parse its
    customizations.xml the same way — use the exact export-and-parse script from Step 2C-1.
  Confirm the discovered GUIDs with the user before writing any files.
- File name: `<PascalCaseFlowName>-<GUID-UPPERCASE>.json`

```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
[System.IO.File]::WriteAllText(
  "$exportDir\Workflows\<FileName>.json",
  $newFlowJson,
  $utf8NoBom
)
```

### 2B-5: Update solution.xml

Read the unpacked solution.xml, then add one `<RootComponent>` inside `<RootComponents>`:
```xml
<RootComponent type="29" id="{<NEW-GUID-LOWERCASE>}" behavior="0" />
```
Bump `<Version>` by one patch (e.g. `1.0.4.0` → `1.0.5.0`).
Write back with `$utf8NoBom`.

### 2B-6: Update customizations.xml

Read the unpacked customizations.xml, then add inside `<Workflows>`:
```xml
<Workflow WorkflowId="{<new-guid-lowercase>}" Name="<Display Name>">
  <JsonFileName>/Workflows/<FileName>.json</JsonFileName>
  <Type>1</Type>
  <Subprocess>0</Subprocess>
  <Category>5</Category>
  <Mode>0</Mode>
  <Scope>4</Scope>
  <OnDemand>1</OnDemand>
  <TriggerOnCreate>0</TriggerOnCreate>
  <TriggerOnDelete>0</TriggerOnDelete>
  <AsyncAutodelete>0</AsyncAutodelete>
  <SyncWorkflowLogOnFailure>0</SyncWorkflowLogOnFailure>
  <StateCode>1</StateCode>
  <StatusCode>2</StatusCode>
  <RunAs>1</RunAs>
  <IsTransacted>1</IsTransacted>
  <IntroducedVersion>1.0.0.0</IntroducedVersion>
  <IsCustomizable>1</IsCustomizable>
  <BusinessProcessType>0</BusinessProcessType>
  <IsCustomProcessingStepAllowedForOtherPublishers>1</IsCustomProcessingStepAllowedForOtherPublishers>
  <ModernFlowType>0</ModernFlowType>
  <PrimaryEntity>none</PrimaryEntity>
  <LocalizedNames>
    <LocalizedName languagecode="1033" description="<Display Name>" />
  </LocalizedNames>
</Workflow>
```
Write back with `$utf8NoBom`.

### 2B-7: Rezip and import

```powershell
$importZip = "$env:TEMP\PA_ErrorHandling\${solutionName}_import.zip"
if (Test-Path $importZip) { [System.IO.File]::Delete($importZip) }
[System.IO.Compression.ZipFile]::CreateFromDirectory($exportDir, $importZip)
pac solution import --path $importZip --activate-plugins --force-overwrite
```

---

## Step 2C — New solution, shared helpers (cross-solution references)

This mode creates a **new, independent solution** whose business flows call the helper flows
(`Helper - Get Error Message`, `Helper - Send Notification`) that are owned by a different solution
(typically `ErrorHandling`). The helper flows are **not** included in the new solution — they are
referenced only by their current GUIDs.

**Prerequisite:** The helper solution must already be imported in the target environment before
importing the new solution, otherwise the child flow references will be broken.

Read these reference files:
- `references/solution-xml.md` — for the cross-solution solution.xml pattern
- `references/flow-error-handling-template.json` — base template for the new business flows

### 2C-1: Discover the helper flow GUIDs from the installed solution

**Never assume or hardcode the helper flow GUIDs.** GUIDs change each time the helper solution is
deployed from scratch (as of the current SKILL design). Always discover the current GUIDs by
exporting the installed helper solution:

```powershell
$helperSolutionName = "<from Q3, e.g. ErrorHandling>"
$helperExportZip = "$env:TEMP\PA_ErrorHandling\helper_export.zip"
$helperExportDir = "$env:TEMP\PA_ErrorHandling\helper_inspect"

pac solution export --name $helperSolutionName --path $helperExportZip --overwrite

Add-Type -AssemblyName System.IO.Compression.FileSystem
if (Test-Path $helperExportDir) { Remove-Item "$helperExportDir\*" -Recurse -Force }
else { New-Item -ItemType Directory -Force -Path $helperExportDir | Out-Null }
[System.IO.Compression.ZipFile]::ExtractToDirectory($helperExportZip, $helperExportDir)

# Parse customizations.xml to find helper flow GUIDs by name
[xml]$helperCustomXml = Get-Content "$helperExportDir\customizations.xml"
$flows = $helperCustomXml.ImportExportXml.Workflows.Workflow

$guidGetError  = ($flows | Where-Object { $_.Name -like "*Get Error*" }).WorkflowId -replace '[{}]', ''
$guidSendNotif = ($flows | Where-Object { $_.Name -like "*Send Notification*" }).WorkflowId -replace '[{}]', ''

Write-Host "Helper - Get Error Message  GUID: $guidGetError"
Write-Host "Helper - Send Notification  GUID: $guidSendNotif"
```

Confirm with the user that the discovered GUIDs match expected flows before continuing.

### 2C-2: Generate a GUID for each new business flow

```powershell
$newGuid = [System.Guid]::NewGuid().ToString()
Write-Host $newGuid
```

### 2C-3: Create the new solution directory

```
$env:TEMP\PA_ErrorHandling\<NewSolutionName>\
├── [Content_Types].xml
├── solution.xml               ← only lists the new business flows
├── customizations.xml         ← only defines the new business flows
└── Workflows\
    └── <FlowName>-<GUID-UPPERCASE>.json   ← one per business flow
```

### 2C-4: solution.xml for the new solution

Same structure as the standard template in `references/solution-xml.md`, but:
- Use the **new** solution unique name and display name
- `<RootComponents>` lists **only the new business flow GUIDs** — no helper GUIDs
- Do **not** include the helpers as RootComponents (they belong to the other solution)

```xml
<RootComponents>
  <RootComponent type="29" id="{<NEW-FLOW-GUID-LOWERCASE>}" behavior="0" />
  <!-- one entry per new business flow; no helper entries here -->
</RootComponents>
```

### 2C-5: customizations.xml for the new solution

Only include `<Workflow>` entries for the new business flows (`<Subprocess>0</Subprocess>`).
No entries for the helpers.

### 2C-6: Flow JSON files

Use `references/flow-error-handling-template.json` as the base. Substitute the discovered
helper GUIDs into the `workflowReferenceName` fields in the Catch and Finally scopes:

```powershell
$flowJson = $flowJson.Replace(
    '"workflowReferenceName": "HELPER_GET_ERROR_GUID"',
    "`"workflowReferenceName`": `"$guidGetError`""
)
$flowJson = $flowJson.Replace(
    '"workflowReferenceName": "HELPER_SEND_NOTIF_GUID"',
    "`"workflowReferenceName`": `"$guidSendNotif`""
)
```

Generate fresh `operationMetadataId` values for all actions in each new flow to avoid collisions:
```powershell
$ids = 1..10 | ForEach-Object { [System.Guid]::NewGuid().ToString() }
```

### 2C-7: Build zip and import

```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
$solutionName = "<NewSolutionName>"
$dir = "$env:TEMP\PA_ErrorHandling\$solutionName"
$wfDir = "$dir\Workflows"
New-Item -ItemType Directory -Force -Path $wfDir | Out-Null

[System.IO.File]::WriteAllText("$dir\[Content_Types].xml", $contentTypesXml, $utf8NoBom)
[System.IO.File]::WriteAllText("$dir\solution.xml", $solutionXml, $utf8NoBom)
[System.IO.File]::WriteAllText("$dir\customizations.xml", $customizationsXml, $utf8NoBom)
[System.IO.File]::WriteAllText("$wfDir\<FileName>.json", $flowJson, $utf8NoBom)

Add-Type -AssemblyName System.IO.Compression.FileSystem
$zipPath = "$env:TEMP\PA_ErrorHandling\${solutionName}.zip"
if (Test-Path $zipPath) { [System.IO.File]::Delete($zipPath) }
[System.IO.Compression.ZipFile]::CreateFromDirectory($dir, $zipPath)

pac solution import --path $zipPath --activate-plugins --force-overwrite
```

### 2C-8: Post-import wiring note

After import, open each new business flow in the designer. If any child flow action shows as broken,
re-select the helper from the dropdown — Power Automate will re-link it to the installed helper:
- Catch scope → "Run a child flow - Get Error Message" → re-select **Helper - Get Error Message**
- Finally scope (false branch) → "Run a child flow - Send Notification" → re-select **Helper - Send Notification**

---

## Step 2D — Retrofit error handling onto existing flows

Adds the Try-Catch-Finally pattern to flows that ALREADY EXIST in a target solution,
in place. Flow GUIDs, triggers, and connection references are preserved — nothing
needs re-linking after import.

Read these files before transforming anything:
- `references/retrofit-transformation.md` — skip rules, algorithm, validation script
- `references/flow-error-handling-template.json` — source of the Catch/Finally scopes

### 2D-1: Export and unzip the target solution

Use Step 2B-1 and 2B-2 verbatim with the Mode D target solution name from Q3.
Keep the export zip untouched — it is the rollback backup (restore = re-import it).

### 2D-2: Discover helper GUIDs

Use Step 2C-1 verbatim with the helper solution from Q4b (default `ErrorHandling`).
Confirm the discovered GUIDs with the user before transforming.

### 2D-3: List flows and let the user pick

Parse the `<Workflow>` elements in `$exportDir\customizations.xml`. For each flow,
load its JSON file and evaluate the skip rules from
`references/retrofit-transformation.md`. Present the list with flags
(`already-has-try`, `name-collision`, `entangled-initvar`, `empty`) and ask which
flows to retrofit — default: all unflagged flows. Never transform a flagged flow;
flagged flows need manual treatment.

### 2D-4: Transform each selected flow

Apply the algorithm in `references/retrofit-transformation.md` exactly, then run its
validation script on every transformed file. Write back with `$utf8NoBom`, keeping
the original file names.

### 2D-5: Bump version, rezip, import

Bump the solution version by one patch in `$exportDir\solution.xml` (as in Step
2B-5) but do NOT add RootComponents — the flows already belong to the solution.
Then rezip and import per Step 2B-7.

### 2D-6: Post-import

Step 3 items 1 (child flow references) and 4 (business logic already exists — skip)
apply. Verify by exporting the solution again and re-running the validation script
on each retrofitted flow, then test with a forced failure: temporarily add an HTTP
action inside Try with an invalid hostname (see Critical rules — 4xx responses are
NOT extractable; use a nonexistent hostname) and confirm the notification arrives.

---

## Step 3 — Post-import

After a successful import, remind the user:

1. **Wire up child flow references** (if any show as broken): Open the flow(s) in the designer.
   Re-select from the dropdown for any broken child flow actions:
   - Catch scope → "Run a child flow - Get Error Message" → re-select **Helper - Get Error Message**
   - Finally scope (false branch) → "Run a child flow - Send Notification" → re-select **Helper - Send Notification**

2. **Link connection references** (Mode A, when connectors were wired in Q6):
   - Power Automate's import UI should prompt for connection references automatically.
   - If it didn't, go to the solution → **Connection References** → link each reference to a real
     connection (the user may need to create a new Office 365 Outlook or Teams connection first).
   - Connection references to link: `{{PUBLISHER_PREFIX}}_office365_errorhandling` (Office 365 Outlook)
     and/or `{{PUBLISHER_PREFIX}}_teams_errorhandling` (Microsoft Teams), depending on Q6 choice.

3. **Verify connector actions** (if connectors were wired):
   - Open **Helper - Send Notification** → Try → Switch → EMAIL case.
     Confirm `Send_an_email_V2` shows the correct `To` address, subject, body, and importance.
   - Open the TEAMS case. Confirm `Post_adaptive_card_in_a_chat_or_channel` shows the correct
     Group ID and Channel ID. The Teams connector parameters (`body/recipient/groupId` etc.) may
     need adjustment if the connector version in your environment differs — verify by exporting an
     existing working Teams flow and comparing the parameter names.

4. **Add business logic**: In the template flow's Try scope, replace
   `Placeholder_-_Add_your_business_logic_here` with real actions.

5. **Save generated GUIDs**: If the user will create cross-solution (Mode C) flows later,
   remind them to save the GUIDs printed during Step 2A. They can always re-discover them
   by re-running Mode C Step 2C-1 (exporting and parsing the helper solution).

---

## Critical rules

- **Always generate fresh GUIDs** with `[System.Guid]::NewGuid()` for each new deployment.
  Never reuse GUIDs from a previous deployment — if a flow was deleted, its GUID may still be
  orphaned in the environment and cause import conflicts.
- **Always use UTF-8 without BOM**: `New-Object System.Text.UTF8Encoding $false`. Using `[System.Text.Encoding]::UTF8` adds a BOM that silently breaks `pac solution import`.
- **`<RootComponent id="...">` GUIDs must be lowercase** in solution.xml. Uppercase GUIDs cause a false "component is not declared as a root component" import error even though the GUID appears correct visually.
- **`x-ms-dynamically-added: true`** is required on every trigger input property or the designer won't show inputs.
- **Dropdown inputs**: Use `"x-ms-content-hint": "DROP_DOWN"` + `"enum": [...]` for option sets.
- **HTTP 4xx responses are not extractable by the XPath helper.** When an HTTP action receives a 4xx/5xx HTTP response, Power Automate stores the failure as `{"code":"NotFound","outputs":{"statusCode":404,"body":{...}}}` — there is no `error.message` field, so the helper's XPath (`//error/message/text()` and `//message/text()`) finds nothing. The helper only works on **network-level failures** (DNS errors, connection refused, timeouts) which produce `{"error":{"message":"No such host is known..."}}`. If you need to demonstrate the error handler with an HTTP failure, use an **invalid hostname** (e.g. `https://api.hostname-that-does-not-exist.com/...`) rather than a bad path or invalid query parameter.
- **Teams connector parameters**: The `Post adaptive card in a chat or channel` action uses
  operationId `PostCardToConversation` with parameters `poster` (`Flow bot`), `location` (`Channel`),
  `body/recipient/groupId`, `body/recipient/channelId`, and `body/messageBody` — the adaptive card
  JSON passed **as a string** (`string(outputs('Compose_-_Teams_Adaptive_Card'))`). Connection
  reference entries in the flow JSON use `"runtimeSource": "embedded"`. Verified against the
  Microsoft Teams connector reference (learn.microsoft.com/connectors/teams) and a real solution
  export; do not use the deprecated `PostChannelAdaptiveCard`/`PostAdaptiveCardRequest` shape.
- **Connection reference logical names must be unique per publisher** — the pattern
  `{prefix}_office365_errorhandling` and `{prefix}_teams_errorhandling` is intentional. If the
  environment already has connection references with these names (from a previous deployment), the
  import will reuse them, which is correct behaviour.

### Respond to a PowerApp or flow — exact required format

Power Automate silently strips the response body on import unless all of the following are true:

1. **`"kind": "powerapp"`** — must be lowercase. `"PowerApp"` causes the entire body to be reset to `{}`.
2. **Schema properties require `"x-ms-dynamically-added": true` and `"title"`** — without these, the importer does not recognise the outputs and resets the body.
3. **Property names are lowercase** in both `body` and `schema` (e.g. `"errormessage"`, not `"ErrorMessage"`). Use `"title"` for the PascalCase display name.
4. **Inline all expressions directly in the body** — do not reference a Compose action that holds the whole result object (`"body": "@outputs('Compose_-_Result')"` is stripped). Map every property individually with `@{expression}`.
5. **Avoid cross-scope/cross-condition references** — expressions that reach into a nested condition inside another scope (e.g. Finally → Try → Condition → true branch) are stripped. Reference the nearest available action output directly instead (e.g. `body('Filter_Array_-_Get_failed_step')` from Finally is fine; `outputs('Compose_-_Result')` from inside a condition is not).
6. **No-data helpers**: if the flow only needs to signal success/failure with no return values, use `"inputs": { "statusCode": 200 }` with no `body` or `schema` — do not add empty properties.

**Correct format template:**
```json
{
  "type": "Response",
  "kind": "powerapp",
  "inputs": {
    "statusCode": 200,
    "body": {
      "myproperty": "@{<expression>}"
    },
    "schema": {
      "type": "object",
      "properties": {
        "myproperty": {
          "title": "MyProperty",
          "x-ms-dynamically-added": true,
          "type": "string"
        }
      }
    }
  }
}
```
