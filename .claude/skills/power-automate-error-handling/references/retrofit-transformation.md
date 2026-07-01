# Retrofit Transformation (Mode D)

How to add Try-Catch-Finally to an EXISTING flow's JSON, in place. The flow's GUID,
trigger, `connectionReferences` object, and every existing action's name and
`operationMetadataId` are preserved — only the action tree is restructured.

Read `flow-error-handling-template.json` first: its `Catch` and `Finally` scopes are
copied into the retrofitted flow verbatim (they are the single source of truth).

---

## Skip rules — evaluate BEFORE transforming

Load the flow JSON and inspect `properties.definition.actions`. Flag the flow and
exclude it from the default selection if ANY of these hold:

1. **Already has error handling:** a top-level action named `Try` of type `Scope`.
   (Overridable — the user may still select it, e.g. to inspect manually. Do NOT
   transform it; "override" means the user takes it for manual treatment.)
2. **Name collision:** ANY top-level action named `Try`, `Catch`, or `Finally`,
   regardless of type. Appending the new scopes would collide on action names.
3. **Entangled variables:** any `InitializeVariable` action whose `runAfter`
   references a non-`InitializeVariable` action, OR whose variable `value`
   expression contains `outputs(` or `body(` (it reads another action's result).
   Re-chaining such variables before Try would change semantics or fail at runtime.
4. **Empty flow:** `actions` has no entries. Nothing to wrap — skip outright.

Detection snippet (per flow file):

```powershell
$flow    = Get-Content $file -Raw | ConvertFrom-Json
$actions = $flow.properties.definition.actions
$names   = @($actions.PSObject.Properties.Name)

$flags = @()
if ($names -contains 'Try' -and $actions.Try.type -eq 'Scope') { $flags += 'already-has-try' }
if (@('Try','Catch','Finally') | Where-Object { $names -contains $_ }) { $flags += 'name-collision' }
foreach ($n in $names) {
  $a = $actions.$n
  if ($a.type -eq 'InitializeVariable') {
    $deps = @($a.runAfter.PSObject.Properties.Name)
    if ($deps | Where-Object { $actions.$_.type -ne 'InitializeVariable' }) { $flags += 'entangled-initvar' }
    $valJson = ($a.inputs.variables | ConvertTo-Json -Depth 10)
    if ($valJson -match 'outputs\(' -or $valJson -match 'body\(') { $flags += 'entangled-initvar' }
  }
}
if ($names.Count -eq 0) { $flags += 'empty' }
$flags = $flags | Select-Object -Unique
```

---

## Transformation algorithm

Work on `properties.definition.actions`. Everything else in the file stays untouched.

### 1. Partition

- `InitializeVariable` actions stay at top level (platform restriction: they cannot
  live inside a Scope).
- Every other top-level action moves into a new `Try` scope.

### 2. Rewire runAfter

- Re-chain the top-level `InitializeVariable` actions sequentially in their original
  file order: the first gets `"runAfter": {}`, each subsequent one gets
  `"runAfter": { "<previous InitVar name>": ["Succeeded"] }`.
- The `Try` scope gets `"runAfter": { "<last InitVar name>": ["Succeeded"] }`, or
  `"runAfter": {}` if the flow has no variables.
- Actions moved into Try keep their runAfter entries that point at other moved
  actions. Remove any runAfter entry that points at an `InitializeVariable` (now
  outside the scope). If that leaves an action's runAfter empty (`{}`), it becomes a
  Try entry point — that is correct.

### 3. Append Catch and Finally

Copy the entire `Catch` and `Finally` actions from
`flow-error-handling-template.json` (`properties.definition.actions.Catch` /
`.Finally`) into the flow, then make exactly these substitutions in the copied JSON:

- `HELPER_GET_ERROR_GUID`  → the discovered GUID of "Helper - Get Error Message"
- `HELPER_SEND_NOTIF_GUID` → the discovered GUID of "Helper - Send Notification"
- Every `operationMetadataId` in the COPIED actions → a fresh
  `[System.Guid]::NewGuid()` value. Never touch metadata of pre-existing actions.

Do not edit anything else inside the copied scopes. Resulting runAfter wiring
(already present in the template — verify, don't re-create):
`Catch.runAfter = { "Try": ["Failed", "TimedOut"] }`,
`Finally.runAfter = { "Catch": ["Succeeded", "Failed", "Skipped", "TimedOut"] }`.

### 4. Write back

Serialize and write with UTF-8 no BOM. Keep the original file name (the flow GUID in
the name must not change).

---

## Validation — run per transformed file, before rezipping

```powershell
$t = Get-Content $file -Raw | ConvertFrom-Json   # throws if invalid JSON
$a = $t.properties.definition.actions
if ($a.Try.type -ne 'Scope')  { throw "Try scope missing" }
if (-not $a.Catch)            { throw "Catch missing" }
if (-not $a.Finally)          { throw "Finally missing" }
if (($a.Catch.runAfter.Try -join ',') -ne 'Failed,TimedOut') { throw "Catch runAfter wrong" }
if (($a.Finally.runAfter.Catch -join ',') -ne 'Succeeded,Failed,Skipped,TimedOut') { throw "Finally runAfter wrong" }
$tryNames = @($a.Try.actions.PSObject.Properties.Name)
foreach ($n in $tryNames) {
  if ($a.Try.actions.$n.type -eq 'InitializeVariable') { throw "InitializeVariable inside Try: $n" }
}
$raw = Get-Content $file -Raw
if ($raw.Contains('HELPER_GET_ERROR_GUID') -or $raw.Contains('HELPER_SEND_NOTIF_GUID')) { throw "Unsubstituted helper GUID token" }
Write-Host "OK: $file"
```

Also confirm (by diff or inspection) that every pre-existing action kept its name and
its original `operationMetadataId`.

---

## Worked example

Before (`properties.definition.actions`):

```json
{
  "Initialize_variable_-_Result": {
    "type": "InitializeVariable",
    "inputs": { "variables": [{ "name": "Result", "type": "string" }] },
    "runAfter": {},
    "metadata": { "operationMetadataId": "aaaaaaaa-0000-0000-0000-000000000001" }
  },
  "Get_items": {
    "type": "OpenApiConnection",
    "inputs": { "host": { "...": "unchanged" } },
    "runAfter": { "Initialize_variable_-_Result": ["Succeeded"] },
    "metadata": { "operationMetadataId": "aaaaaaaa-0000-0000-0000-000000000002" }
  },
  "Set_variable_-_Result": {
    "type": "SetVariable",
    "inputs": { "...": "unchanged" },
    "runAfter": { "Get_items": ["Succeeded"] },
    "metadata": { "operationMetadataId": "aaaaaaaa-0000-0000-0000-000000000003" }
  }
}
```

After:

```json
{
  "Initialize_variable_-_Result": {
    "type": "InitializeVariable",
    "inputs": { "variables": [{ "name": "Result", "type": "string" }] },
    "runAfter": {},
    "metadata": { "operationMetadataId": "aaaaaaaa-0000-0000-0000-000000000001" }
  },
  "Try": {
    "type": "Scope",
    "actions": {
      "Get_items": {
        "type": "OpenApiConnection",
        "inputs": { "host": { "...": "unchanged" } },
        "runAfter": {},
        "metadata": { "operationMetadataId": "aaaaaaaa-0000-0000-0000-000000000002" }
      },
      "Set_variable_-_Result": {
        "type": "SetVariable",
        "inputs": { "...": "unchanged" },
        "runAfter": { "Get_items": ["Succeeded"] },
        "metadata": { "operationMetadataId": "aaaaaaaa-0000-0000-0000-000000000003" }
      }
    },
    "runAfter": { "Initialize_variable_-_Result": ["Succeeded"] },
    "metadata": { "operationMetadataId": "<fresh-guid>" }
  },
  "Catch":   { "— copied from flow-error-handling-template.json with helper GUID + fresh metadata ids —": "" },
  "Finally": { "— copied from flow-error-handling-template.json with helper GUID + fresh metadata ids —": "" }
}
```

Note how `Get_items` lost its runAfter on the InitializeVariable (now outside the
scope) and became the Try entry point, while `Set_variable_-_Result` kept its
dependency on `Get_items`. The two original actions keep their exact
`operationMetadataId` values.
