# Mode D (Retrofit Error Handling) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Mode D to the `power-automate-error-handling` skill that retrofits the Try-Catch-Finally pattern onto existing flows in an existing solution, in place.

**Architecture:** One new reference file documents the JSON transformation algorithm; SKILL.md gains option D in Q1, a Mode D solution-pick variant, and a short Step 2D that reuses existing steps 2B-1/2B-2/2C-1/2B-5/2B-7. Catch/Finally scopes are copied from the existing `flow-error-handling-template.json` (single source of truth). Live verification runs against the `Flowswithouterrorhandling` solution in the Development environment.

**Tech Stack:** Markdown skill files, PowerShell + pac CLI (v2.8.1), Dataverse FetchXML via `pac env fetch`.

**Spec:** `docs/superpowers/specs/2026-07-01-retrofit-error-handling-design.md`

**Conventions that apply to every task:** All file writes use UTF-8 without BOM when writing solution files (`New-Object System.Text.UTF8Encoding $false`). Repo commits go directly to `main` (repo convention). Commit messages end with `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`.

---

## File structure

| File | Responsibility |
|---|---|
| `.claude/skills/power-automate-error-handling/references/retrofit-transformation.md` | **Create.** The complete Mode D transformation algorithm: skip rules with detection snippets, partition/rewire/append steps, substitution rules, validation script, worked before/after example. |
| `.claude/skills/power-automate-error-handling/SKILL.md` | **Modify.** Frontmatter description (new trigger phrases), Q1 option D, Q3 Mode D solution listing, Q4b scope widened to Modes B and D, new Step 2D section. |
| `CLAUDE.md` | **Modify (final task).** Session history entry documenting Mode D. |

No other files change. Modes A/B/C behavior is untouched.

---

### Task 1: Create `references/retrofit-transformation.md`

**Files:**
- Create: `.claude/skills/power-automate-error-handling/references/retrofit-transformation.md`

- [ ] **Step 1: Write the file with exactly this content**

````markdown
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
````

- [ ] **Step 2: Verify the file**

Run: `Get-Content ".claude\skills\power-automate-error-handling\references\retrofit-transformation.md" | Measure-Object -Line`
Expected: > 150 lines. Then grep the file for `HELPER_GET_ERROR_GUID`, `Failed,TimedOut`, and `entangled-initvar` — all three must be present.

- [ ] **Step 3: Commit**

```powershell
git add .claude/skills/power-automate-error-handling/references/retrofit-transformation.md
git commit -m "Add retrofit transformation reference for skill Mode D"
```

---

### Task 2: SKILL.md — frontmatter description and Q1 option D

**Files:**
- Modify: `.claude/skills/power-automate-error-handling/SKILL.md` (frontmatter lines 1–11, Q1 block ~line 43)

- [ ] **Step 1: Extend the frontmatter description**

In the YAML frontmatter `description`, after the phrase `or scaffold error handling helpers.`, insert:

```
Also triggers on adding error handling to EXISTING flows: "add error handling to
existing flows", "retrofit error handling", "wrap my flows in try-catch",
"add try-catch to flows in a solution".
```

(Keep it inside the same folded string; adjust line wrapping to match the file's style.)

- [ ] **Step 2: Add option D to Q1**

Replace:

```markdown
> (C) New solution, shared helpers — create a brand-new solution whose flows call the helper flows that live in a different solution"
```

with:

```markdown
> (C) New solution, shared helpers — create a brand-new solution whose flows call the helper flows that live in a different solution
> (D) Retrofit — add Try-Catch-Finally error handling to existing flows in an existing solution"
```

- [ ] **Step 3: Verify**

Grep SKILL.md for `(D) Retrofit` → exactly 1 match; grep for `retrofit error handling` in the frontmatter → 1 match.

- [ ] **Step 4: Commit**

```powershell
git add .claude/skills/power-automate-error-handling/SKILL.md
git commit -m "SKILL.md: add Mode D to Q1 and retrofit trigger phrases"
```

---

### Task 3: SKILL.md — Mode D solution pick and Q4b scope

**Files:**
- Modify: `.claude/skills/power-automate-error-handling/SKILL.md` (Q3 block, Q4b block)

- [ ] **Step 1: Add the Mode D bullet to Q3**

After the Mode C bullet in Q3 — its last line is exactly:
``    Ask: "Which solution contains the helper flows? (default: `ErrorHandling`)"``
(note the backticks around ErrorHandling) — insert:

```markdown
- **Mode D**: Run `pac solution list` and show ALL solutions EXCEPT Microsoft/system
  ones — exclude unique names starting with `msdyn`, `msft`, `Microsoft`, `Default`,
  `Active`, or `Basic`, and canvas-app auto-solutions (`Cr` + 5 hex chars). Do NOT
  filter by publisher prefix: retrofit targets often belong to other publishers.
  Show the unfiltered list if the user asks.
  Then ask: "Which solution contains the flows to retrofit?"
```

- [ ] **Step 2: Widen Q4b to Modes B and D**

Replace:

```markdown
**Q4b — Helper solution** (Mode B only):
```

with:

```markdown
**Q4b — Helper solution** (Modes B and D; for Mode D the default is `ErrorHandling`):
```

- [ ] **Step 3: Verify**

Grep SKILL.md for `Which solution contains the flows to retrofit` → 1 match; `Modes B and D` → 1 match; `(Mode B only)` → 0 matches.

- [ ] **Step 4: Commit**

```powershell
git add .claude/skills/power-automate-error-handling/SKILL.md
git commit -m "SKILL.md: Mode D solution listing (non-Microsoft) and Q4b scope"
```

---

### Task 4: SKILL.md — Step 2D section

**Files:**
- Modify: `.claude/skills/power-automate-error-handling/SKILL.md` (insert before `## Step 3 — Post-import`)

- [ ] **Step 1: Insert exactly this section before the `## Step 3 — Post-import` heading**

````markdown
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
action with an invalid hostname inside Try (see Critical rules — 4xx responses are
NOT extractable; use a nonexistent hostname) and confirm the notification arrives.

---
````

- [ ] **Step 2: Verify**

Grep SKILL.md: `## Step 2D` → 1 match; `2D-1` through `2D-6` → 1 match each; the section must appear BEFORE `## Step 3 — Post-import`.

- [ ] **Step 3: Commit**

```powershell
git add .claude/skills/power-automate-error-handling/SKILL.md
git commit -m "SKILL.md: add Step 2D retrofit workflow"
```

---

### Task 5: Cross-file consistency check

**Files:** none modified — verification only.

- [ ] **Step 1: Verify all cross-references resolve**

- Step 2D references `2B-1`, `2B-2`, `2C-1`, `2B-5`, `2B-7`, `Q3`, `Q4b`, `Step 3` — grep SKILL.md to confirm each referenced anchor still exists.
- `retrofit-transformation.md` references `flow-error-handling-template.json` — confirm that file still contains top-level `Catch` and `Finally` actions with the runAfter arrays `["Failed", "TimedOut"]` / `["Succeeded", "Failed", "Skipped", "TimedOut"]` and both `HELPER_*_GUID` tokens.

Expected: every reference resolves. If any does not, STOP and fix before proceeding.

---

### Task 6: Live test — transform the fixture flow (no import yet)

**Files:** none in repo — works in `$env:TEMP\PA_ErrorHandling\`.

Precondition: pac auth active profile points at `https://ruprect-development.crm4.dynamics.com` (`pac auth list`).

- [ ] **Step 1: Verify the fixture still exists**

```powershell
$fetch = "<fetch><entity name='workflow'><attribute name='name' /><attribute name='statecode' /><link-entity name='solutioncomponent' from='objectid' to='workflowid'><link-entity name='solution' from='solutionid' to='solutionid'><filter><condition attribute='uniquename' operator='eq' value='Flowswithouterrorhandling' /></filter></link-entity></link-entity></entity></fetch>"
pac env fetch --xml $fetch
```

Expected: `Get Mock Data for Canvas App` listed. If the solution or flow is missing, STOP and ask the user to recreate it.

- [ ] **Step 2: Export + unzip (2D-1), discover helper GUIDs (2D-2)**

Run the Step 2B-1/2B-2 script with `$solutionName = "Flowswithouterrorhandling"`, then the Step 2C-1 script with `$helperSolutionName = "ErrorHandling"`.
Expected: helper GUIDs printed = `9563b2ec-9366-4bce-b554-0deff90939a9` (Get Error Message) and `4edf92ac-20bb-4868-8658-bb07a19bbbb2` (Send Notification) — the current deployment. If different, use the discovered values (they are authoritative).

- [ ] **Step 3: Evaluate skip rules on the fixture flow (2D-3)**

Run the detection snippet from `retrofit-transformation.md` against the exported flow JSON.
Expected: no flags (the fixture is a plain mock-data flow). If flagged, STOP and report which rule fired.

- [ ] **Step 4: Transform (2D-4) and validate**

Apply the algorithm; run the validation script from `retrofit-transformation.md`.
Expected output: `OK: <file>`. Additionally diff the original vs transformed JSON to confirm original action names and `operationMetadataId` values are unchanged inside Try.

- [ ] **Step 5: Checkpoint — show the user the transformed JSON structure**

Present a compact summary (top-level action list with types and runAfter) and wait for go-ahead before importing.

---

### Task 7: Live test — import and verify

- [ ] **Step 1: Bump version, rezip, import (2D-5)**

Bump `1.0.0.0` → `1.0.1.0` in the exported solution.xml, rezip to `Flowswithouterrorhandling_import.zip`, run `pac solution import --path <zip> --activate-plugins --force-overwrite`.
Expected: `Solution Imported successfully.`

- [ ] **Step 2: Export back and re-validate**

Re-export the solution, unzip to a fresh directory, and run the validation script on the retrofitted flow.
Expected: `OK` — structure survived the import round-trip (Try/Catch/Finally present, helper GUIDs wired, original metadata intact).

- [ ] **Step 3: Commit the plan checkboxes / session notes so far**

```powershell
git add docs/superpowers/plans/2026-07-02-retrofit-error-handling-mode-d.md
git commit -m "Mode D: live import test passed"
```

---

### Task 8: End-to-end forced-failure test (user-interactive checkpoint)

- [ ] **Step 1: Ask the user to verify in the designer**

The user opens *Get Mock Data for Canvas App* in the Power Automate designer and confirms: variables (if any) → Try (original actions) → Catch → Finally render without broken child-flow references.

- [ ] **Step 2: Forced failure**

The user (in the designer) temporarily adds an HTTP action inside Try with URL `https://api.hostname-that-does-not-exist.example/x`, saves, runs the flow, then removes the action. Expected: flow run shows Failed; an error email arrives at the configured recipient with the real DNS error message (not `(no details)`).

- [ ] **Step 3: Update CLAUDE.md and finish**

Append a session-history entry (Mode D added to the skill; live-tested against `Flowswithouterrorhandling`; keep it free of emails/IDs per the public-repo scrub policy). Commit:

```powershell
git add CLAUDE.md
git commit -m "Document Mode D (retrofit error handling) in session history"
```

Then ask the user whether to push to `origin main`.
