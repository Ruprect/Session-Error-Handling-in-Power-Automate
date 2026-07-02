# TODO 07 — Live test: import and round-trip verification

**Kind:** ENVIRONMENT REQUIRED. Prerequisite: TODO 06 is DONE (transformed flow JSON
validated in `$env:TEMP\PA_ErrorHandling\Flowswithouterrorhandling\`).

**This TODO modifies the Development environment.** Before importing, show the user a
compact summary of the transformed flow (top-level action names, types, and runAfter)
and wait for explicit go-ahead.

## Steps

### 1. Bump version, rezip, import

In `$env:TEMP\PA_ErrorHandling\Flowswithouterrorhandling\solution.xml`, bump
`<Version>` by one patch (e.g. `1.0.0.0` → `1.0.1.0`). Write back with UTF-8 no BOM:

```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
# ... edit version, then WriteAllText with $utf8NoBom
```

Then:

```powershell
$dir = "$env:TEMP\PA_ErrorHandling\Flowswithouterrorhandling"
$importZip = "$env:TEMP\PA_ErrorHandling\Flowswithouterrorhandling_import.zip"
if (Test-Path $importZip) { [System.IO.File]::Delete($importZip) }
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::CreateFromDirectory($dir, $importZip)
pac solution import --path $importZip --activate-plugins --force-overwrite
```

Expected: `Solution Imported successfully.`
If the import FAILS, the rollback is: re-import the untouched
`Flowswithouterrorhandling_export.zip` from TODO 06 step 2.

### 2. Export back and re-validate

```powershell
$verifyZip = "$env:TEMP\PA_ErrorHandling\fwoeh_verify.zip"
$verifyDir = "$env:TEMP\PA_ErrorHandling\fwoeh_verify"
pac solution export --name Flowswithouterrorhandling --path $verifyZip --overwrite
if (Test-Path $verifyDir) { Remove-Item $verifyDir -Recurse -Force }
[System.IO.Compression.ZipFile]::ExtractToDirectory($verifyZip, $verifyDir)
```

Run the validation script from `retrofit-transformation.md` against the flow JSON in
`$verifyDir\Workflows\`. Expected: `OK` — Try/Catch/Finally survived the import
round-trip, helper GUIDs still wired, original action metadata intact.

## On success

Rename this file `DONE-` and note the imported solution version at the bottom.

---

## Execution notes (2026-07-02)

- Solution version bumped 1.0.0.0 -> 1.0.1.0; imported with --force-overwrite: success.
- Round-trip export re-validated: Try/Catch/Finally intact with correct runAfter arrays,
  original Select/Respond actions preserved (names, operationMetadataIds, Response body),
  helper GUIDs wired, flow GUID unchanged.
- Rollback zip retained: %TEMP%\PA_ErrorHandling\Flowswithouterrorhandling_export.zip
