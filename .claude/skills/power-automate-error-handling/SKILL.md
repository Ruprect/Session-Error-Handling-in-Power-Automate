---
name: power-automate-error-handling
description: >
  Use this skill whenever the user wants to set up Power Automate error handling flows, create a
  Try-Catch-Finally pattern in Power Automate, deploy the ErrorHandling solution, add new flows
  to an existing Power Automate solution, create a new solution that references shared helper flows
  from another solution, or scaffold error handling helpers. Triggers on phrases like "set up error
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
> (C) New solution, shared helpers — create a brand-new solution whose flows call the helper flows that live in a different solution"

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

**Q4 — Flow name(s)** (modes B and C):
> "What is the display name of the new flow? (e.g. `My Business Process`)"
>
> After each name ask: "Add another flow, or done?"

**Q5 — Confirm pac auth:**
> Show the ready-to-run command using `ENVIRONMENT_ID` loaded from the env file:
> `pac auth create --deviceCode --environment <ENVIRONMENT_ID>`
>
> If no env file was found, show the placeholder and ask the user to supply the environment ID.
> Ask: "Are you authenticated to this environment? Run the command above if not."

Record all answers before proceeding.

---

## Step 2A — Fresh setup

Read these reference files before writing any files:
- `references/solution-xml.md` — templates for solution.xml, customizations.xml, [Content_Types].xml
- `references/flow-error-handling-template.json` — parent flow JSON
- `references/flow-get-error-message.json` — helper child flow JSON
- `references/flow-send-notification.json` — notification helper JSON

### Directory structure to create

```
<temp>\ErrorHandling\
├── [Content_Types].xml
├── solution.xml
├── customizations.xml
└── Workflows\
    ├── ErrorHandlingTemplate-3FC9A2B1-D4E5-4678-9012-3456789ABCDE.json
    ├── GetErrorMessage-5AE8B3C2-F6D7-4891-B023-456789ABCDEF.json
    └── HelperSendNotification-7CD4E5F6-A8B9-4C2D-B1E3-567890ABCDEF.json
```

Use `$env:TEMP\PA_ErrorHandling` as the temp directory.

### Fixed GUIDs (never change these — they ensure re-import is idempotent)

| Flow | GUID |
|---|---|
| Error Handling Template | `3fc9a2b1-d4e5-4678-9012-3456789abcde` |
| Helper - Get Error Message | `5ae8b3c2-f6d7-4891-b023-456789abcdef` |
| Helper - Send Notification | `7cd4e5f6-a8b9-4c2d-b1e3-567890abcdef` |

### PowerShell script pattern

```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
$dir = "$env:TEMP\PA_ErrorHandling\ErrorHandling"
$wfDir = "$dir\Workflows"
New-Item -ItemType Directory -Force -Path $wfDir | Out-Null

# Write each file — MUST use $utf8NoBom to avoid BOM that breaks pac importer
[System.IO.File]::WriteAllText("$dir\[Content_Types].xml", $contentTypesXml, $utf8NoBom)
[System.IO.File]::WriteAllText("$dir\solution.xml", $solutionXml, $utf8NoBom)
[System.IO.File]::WriteAllText("$dir\customizations.xml", $customizationsXml, $utf8NoBom)
[System.IO.File]::WriteAllText("$wfDir\ErrorHandlingTemplate-3FC9A2B1-D4E5-4678-9012-3456789ABCDE.json", $errorTemplateJson, $utf8NoBom)
[System.IO.File]::WriteAllText("$wfDir\GetErrorMessage-5AE8B3C2-F6D7-4891-B023-456789ABCDEF.json", $getErrorMsgJson, $utf8NoBom)
[System.IO.File]::WriteAllText("$wfDir\HelperSendNotification-7CD4E5F6-A8B9-4C2D-B1E3-567890ABCDEF.json", $sendNotifJson, $utf8NoBom)

# Zip
Add-Type -AssemblyName System.IO.Compression.FileSystem
$zipPath = "$env:TEMP\PA_ErrorHandling\ErrorHandling_v2.zip"
if (Test-Path $zipPath) { [System.IO.File]::Delete($zipPath) }
[System.IO.Compression.ZipFile]::CreateFromDirectory($dir, $zipPath)

# Import
pac solution import --path $zipPath --activate-plugins --force-overwrite
```

**Substitute** the user's answers from Q2 and Q3 into solution.xml:
- `<UniqueName>` and `<LocalizedNames>` ← solution name / display name from Q3
- `<Publisher><UniqueName>` ← publisher unique name from Q2
- `<CustomizationPrefix>` ← publisher prefix from Q2
- Update `<CustomizationOptionValuePrefix>` only if the user provides a custom value; otherwise leave the default `91517`

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
$newGuid = [System.Guid]::NewGuid().ToString().ToUpper()
Write-Host $newGuid
```

### 2B-4: Create the new flow JSON file(s)

Based on `references/flow-error-handling-template.json`:
- Generate fresh `operationMetadataId` GUIDs for every action (avoids collisions)
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
<RootComponent type="29" id="{<NEW-GUID-UPPERCASE>}" behavior="0" />
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
referenced only by their stable GUIDs.

**Prerequisite:** The helper solution must already be imported in the target environment before
importing the new solution, otherwise the child flow references will be broken.

Read these reference files:
- `references/solution-xml.md` — for the cross-solution solution.xml pattern
- `references/flow-error-handling-template.json` — base template for the new business flows

### How cross-solution child flow references work

The calling flow's JSON references helpers by GUID inside the Catch scope:
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

Power Automate resolves this GUID at runtime regardless of which solution owns the target flow.
The new solution does **not** need to redeclare the helpers as its own RootComponents — it only
registers the business flows it owns.

### 2C-1: Generate a GUID for each new business flow

```powershell
$newGuid = [System.Guid]::NewGuid().ToString()
Write-Host $newGuid
```

### 2C-2: Create the new solution directory

```
$env:TEMP\PA_ErrorHandling\<NewSolutionName>\
├── [Content_Types].xml
├── solution.xml               ← only lists the new business flows
├── customizations.xml         ← only defines the new business flows
└── Workflows\
    └── <FlowName>-<GUID-UPPERCASE>.json   ← one per business flow
```

### 2C-3: solution.xml for the new solution

Same structure as the standard template in `references/solution-xml.md`, but:
- Use the **new** solution unique name and display name
- `<RootComponents>` lists **only the new business flow GUIDs** — no helper GUIDs
- Do **not** include the helpers as RootComponents (they belong to the other solution)

```xml
<RootComponents>
  <RootComponent type="29" id="{<NEW-FLOW-GUID-UPPERCASE>}" behavior="0" />
  <!-- one entry per new business flow; no helper entries here -->
</RootComponents>
```

### 2C-4: customizations.xml for the new solution

Only include `<Workflow>` entries for the new business flows (`<Subprocess>0</Subprocess>`).
No entries for the helpers.

### 2C-5: Flow JSON files

Use `references/flow-error-handling-template.json` as the base. The Catch scope's
`workflowReferenceName` stays exactly as-is (`5ae8b3c2-f6d7-4891-b023-456789abcdef`) — this
points to the helper in the other solution and works by GUID lookup at runtime.

Generate fresh `operationMetadataId` values for all actions in each new flow to avoid collisions:
```powershell
$ids = 1..10 | ForEach-Object { [System.Guid]::NewGuid().ToString() }
```

### 2C-6: Build zip and import

```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
$solutionName = "<NewSolutionName>"
$dir = "$env:TEMP\PA_ErrorHandling\$solutionName"
$wfDir = "$dir\Workflows"
New-Item -ItemType Directory -Force -Path $wfDir | Out-Null

[System.IO.File]::WriteAllText("$dir\[Content_Types].xml", $contentTypesXml, $utf8NoBom)
[System.IO.File]::WriteAllText("$dir\solution.xml", $solutionXml, $utf8NoBom)
[System.IO.File]::WriteAllText("$dir\customizations.xml", $customizationsXml, $utf8NoBom)
# Write one JSON file per new business flow
[System.IO.File]::WriteAllText("$wfDir\<FileName>.json", $flowJson, $utf8NoBom)

Add-Type -AssemblyName System.IO.Compression.FileSystem
$zipPath = "$env:TEMP\PA_ErrorHandling\${solutionName}.zip"
if (Test-Path $zipPath) { [System.IO.File]::Delete($zipPath) }
[System.IO.Compression.ZipFile]::CreateFromDirectory($dir, $zipPath)

pac solution import --path $zipPath --activate-plugins --force-overwrite
```

### 2C-7: Post-import wiring note

After import, open each new business flow in the designer → Catch scope → "Run a child flow"
action. If the reference shows as broken, re-select **Helper - Get Error Message** from the
dropdown — Power Automate will re-link it to the installed helper from the other solution.

---

## Step 3 — Post-import

After a successful import, remind the user:

1. **Wire up child flow reference**: Open "Error Handling Template" (or the new flow) in the designer → Catch scope → "Run a child flow" action → re-select **Helper - Get Error Message** from the dropdown if it shows as broken.

2. **Replace placeholder send actions**: In **Helper - Send Notification**, the EMAIL and TEAMS cases each have a placeholder Compose action. Replace it with the real connector:
   - EMAIL → Office 365 Outlook: Send an Email (V2), use `outputs('Compose_-_Resolved_Subject')` and `outputs('Compose_-_Email_HTML')`
   - TEAMS → Post adaptive card in a chat or channel, use `outputs('Compose_-_Teams_Adaptive_Card')`

3. **Add business logic**: In the flow's Try scope, replace `Placeholder_-_Add_your_business_logic_here` with real actions.

---

## Critical rules

- **Always use UTF-8 without BOM**: `New-Object System.Text.UTF8Encoding $false`. Using `[System.Text.Encoding]::UTF8` adds a BOM that silently breaks `pac solution import`.
- **GUIDs are stable**: Never change the helper flow GUIDs — re-importing with `--force-overwrite` updates rather than duplicates.
- **`x-ms-dynamically-added: true`** is required on every trigger input property or the designer won't show inputs.
- **Dropdown inputs**: Use `"x-ms-content-hint": "DROP_DOWN"` + `"enum": [...]` for option sets.

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
