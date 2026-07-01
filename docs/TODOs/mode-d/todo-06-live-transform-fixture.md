# TODO 06 — Live test: export, discover GUIDs, transform the fixture flow

**Kind:** ENVIRONMENT REQUIRED — needs an authenticated pac CLI session against the
Development environment. If `pac auth list` does not show an active profile for
`https://ruprect-development.crm4.dynamics.com`, STOP and leave this and later TODOs
for a supervised session.

This TODO does NOT import anything — it stops after transforming and validating
locally. TODO 07 does the import.

## Steps

### 1. Verify the fixture exists

```powershell
$fetch = "<fetch><entity name='workflow'><attribute name='name' /><attribute name='statecode' /><link-entity name='solutioncomponent' from='objectid' to='workflowid'><link-entity name='solution' from='solutionid' to='solutionid'><filter><condition attribute='uniquename' operator='eq' value='Flowswithouterrorhandling' /></filter></link-entity></link-entity></entity></fetch>"
pac env fetch --xml $fetch
```

Expected: a flow named `Get Mock Data for Canvas App`. If missing, STOP and note it
at the bottom of this file.

### 2. Export and unzip the target solution

```powershell
$solutionName = "Flowswithouterrorhandling"
$exportDir    = "$env:TEMP\PA_ErrorHandling\$solutionName"
$exportZip    = "$env:TEMP\PA_ErrorHandling\${solutionName}_export.zip"
New-Item -ItemType Directory -Force -Path $exportDir | Out-Null
pac solution export --name $solutionName --path $exportZip --overwrite
Add-Type -AssemblyName System.IO.Compression.FileSystem
if (Test-Path $exportDir\solution.xml) { Remove-Item "$exportDir\*" -Recurse -Force }
[System.IO.Compression.ZipFile]::ExtractToDirectory($exportZip, $exportDir)
Get-ChildItem "$exportDir\Workflows"
```

Keep `${solutionName}_export.zip` untouched — it is the rollback backup.

### 3. Discover the helper flow GUIDs

Run the Step 2C-1 script from
`.claude/skills/power-automate-error-handling/SKILL.md` with
`$helperSolutionName = "ErrorHandling"`.

Expected: two GUIDs printed. As of 2026-07-01 they are
`9563b2ec-9366-4bce-b554-0deff90939a9` (Get Error Message) and
`4edf92ac-20bb-4868-8658-bb07a19bbbb2` (Send Notification) — but the DISCOVERED
values are authoritative; use whatever the script prints.

### 4. Evaluate skip rules on the fixture flow

Run the detection snippet from
`.claude/skills/power-automate-error-handling/references/retrofit-transformation.md`
against the flow JSON in `$exportDir\Workflows\`.

Expected: `$flags` is empty. If any flag fires, STOP and note which one at the
bottom of this file.

### 5. Transform

Apply the transformation algorithm from `retrofit-transformation.md` EXACTLY to the
flow JSON, substituting the helper GUIDs discovered in step 3. Read the whole
reference file before editing. Preserve every existing action's name and
`operationMetadataId`; generate fresh GUIDs only for the newly added actions.

### 6. Validate

Run the validation script from `retrofit-transformation.md` against the transformed
file. Expected output: `OK: <file>`.

Then diff original vs transformed to confirm no pre-existing action's name or
`operationMetadataId` changed.

## On success

Rename this file `DONE-`. Nothing to commit (all work is in `$env:TEMP`). Note the
discovered helper GUIDs at the bottom of this file for TODO 07's records.
