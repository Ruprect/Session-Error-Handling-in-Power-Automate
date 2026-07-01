# TODO 06 — Clarify helper-GUID discovery in Mode B (Step 2B)

**Priority:** MEDIUM — Mode B produces broken child-flow references when the target solution
does not itself contain the helper flows.
**File to change:** `.claude/skills/power-automate-error-handling/SKILL.md`
**Type of change:** Replace one bullet and add one question. Exact text provided below.

## Context

Since the fresh-GUID policy was introduced, helper flow GUIDs are no longer fixed — they must be
discovered from the environment. Step 2B-4 currently says:

> Update the `workflowReferenceName` values inside the Catch and Finally scopes to point to the
> GUIDs of whichever helper flows are installed — discover them from the exported solution's
> customizations.xml (see Step 2C-1 pattern for how to parse it)

This only works when the TARGET solution (the one being added to) contains the helpers. In Mode B
the user can add a template flow to ANY solution — e.g. a business solution that calls helpers
living in `ErrorHandling`. In that case the exported target solution's customizations.xml does NOT
contain the helper flows, and the model has no instruction for where to find their GUIDs.

Secondary issue: Step 2B-3 generates the GUID with `.ToUpper()`, and a careless reader may paste
the uppercase GUID into solution.xml's `<RootComponent>` — which the skill's own critical rule
forbids (must be lowercase). Generate lowercase, uppercase only for the file name.

## Exact changes

### Change 1 — Step 1, after Q4

In `SKILL.md`, find the Q4 block (heading `**Q4 — Flow name(s)** (modes B and C):`). Directly
AFTER that block (before the `**Q5 — Confirm pac auth:**` heading), insert this new question:

```markdown
**Q4b — Helper solution** (Mode B only):
> "Which solution contains the helper flows (`Helper - Get Error Message`,
> `Helper - Send Notification`) that the new flow should call? (default: the same solution
> you are adding the flow to; otherwise typically `ErrorHandling`)"
```

### Change 2 — Step 2B-3

Find this code block in section `### 2B-3: Generate a GUID for each new flow`:

```powershell
$newGuid = [System.Guid]::NewGuid().ToString().ToUpper()
Write-Host $newGuid
```

Replace it with:

```powershell
# Lowercase for solution.xml/customizations.xml; uppercase ONLY in the JSON file name
$newGuid      = [System.Guid]::NewGuid().ToString()
$newGuidUpper = $newGuid.ToUpper()
Write-Host $newGuid
```

### Change 3 — Step 2B-4 helper discovery bullet

In section `### 2B-4: Create the new flow JSON file(s)`, find the bullet:

```
- Update the `workflowReferenceName` values inside the Catch and Finally scopes to point to the
  GUIDs of whichever helper flows are installed — discover them from the exported solution's
  customizations.xml (see Step 2C-1 pattern for how to parse it)
```

Replace it with:

```
- Update the `workflowReferenceName` values inside the Catch and Finally scopes to the current
  GUIDs of the installed helper flows. Discover them from the helper solution named in Q4b:
  - If Q4b = the target solution itself: parse the ALREADY-EXPORTED customizations.xml in
    `$exportDir` — find the `<Workflow>` elements whose `Name` contains "Get Error" and
    "Send Notification" and take their `WorkflowId` values (strip the braces).
  - If Q4b = a different solution: export THAT solution separately and parse its
    customizations.xml the same way — use the exact export-and-parse script from Step 2C-1.
  Confirm the discovered GUIDs with the user before writing any files.
```

## Verification

1. SKILL.md must contain the heading text `**Q4b — Helper solution** (Mode B only):`.
2. Step 2B-3 must no longer contain `.ToString().ToUpper()` on the NewGuid line.
3. Step 2B-4 must reference Q4b and describe both the same-solution and different-solution cases.
