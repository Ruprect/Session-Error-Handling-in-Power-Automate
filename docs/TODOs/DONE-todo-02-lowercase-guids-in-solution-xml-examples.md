# TODO 02 — Lowercase GUIDs in solution-xml.md cross-solution examples

**Priority:** HIGH — the skill's own critical rule says RootComponent GUIDs MUST be lowercase
(uppercase causes a false "component is not declared as a root component" import error), but the
cross-solution examples in the reference file show an UPPERCASE GUID. A model following the example
literally will produce a failing import.
**File to change:** `.claude/skills/power-automate-error-handling/references/solution-xml.md`
**Type of change:** Targeted find-and-replace. No judgment required.

## The rule being enforced

- `<RootComponent id="{...}">` in solution.xml → GUID must be **lowercase**
- `WorkflowId="{...}"` in customizations.xml → GUID must be **lowercase**
- The GUID embedded in `<JsonFileName>` (the flow's file name) → stays **UPPERCASE**
  (that is the established file-naming convention; do not change it)

## Exact changes

In `references/solution-xml.md`:

1. In the "Cross-solution scenario (Mode C): solution.xml" example, replace:
   - old: `<RootComponent type="29" id="{A1B2C3D4-E5F6-7890-ABCD-EF1234567890}" behavior="0" />`
   - new: `<RootComponent type="29" id="{a1b2c3d4-e5f6-7890-abcd-ef1234567890}" behavior="0" />`

   This string occurs TWICE in the file (once in the standalone `<RootComponents>` snippet if
   present, once in the full solution.xml example). Replace all occurrences.

2. In the "Cross-solution scenario: customizations.xml" example, replace:
   - old: `<Workflow WorkflowId="{A1B2C3D4-E5F6-7890-ABCD-EF1234567890}" Name="My Business Process">`
   - new: `<Workflow WorkflowId="{a1b2c3d4-e5f6-7890-abcd-ef1234567890}" Name="My Business Process">`

3. Do NOT change this line (file names keep uppercase GUIDs):
   `<JsonFileName>/Workflows/MyBusinessProcess-A1B2C3D4-E5F6-7890-ABCD-EF1234567890.json</JsonFileName>`

4. Add a one-line comment directly above the `<RootComponents>` element in the cross-solution
   solution.xml example:
   `<!-- GUIDs in RootComponent id MUST be lowercase -->`

## Verification

1. Search the file for `id="{A` — there must be ZERO matches.
2. Search the file for `WorkflowId="{A` — there must be ZERO matches.
3. The `<JsonFileName>` lines must still contain the UPPERCASE GUID form.
