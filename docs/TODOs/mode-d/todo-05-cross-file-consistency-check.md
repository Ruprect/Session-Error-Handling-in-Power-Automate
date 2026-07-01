# TODO 05 — Cross-file consistency check

**Kind:** Verification only — NO file edits. If any check fails, STOP, write what you
found at the bottom of this file, and do not proceed to TODO 06.

## Checks

All paths relative to the repo root; the skill directory is
`.claude/skills/power-automate-error-handling/`.

1. **Step 2D cross-references resolve.** SKILL.md must contain all of these headings
   (they are referenced by the new Step 2D section):
   - `### 2B-1: Export the current solution`
   - `### 2B-2: Unzip the export`
   - `### 2B-5: Update solution.xml`
   - `### 2B-7: Rezip and import`
   - `### 2C-1: Discover the helper flow GUIDs from the installed solution`
   - `## Step 3 — Post-import`

2. **Reference file exists and is referenced.**
   - `references/retrofit-transformation.md` exists.
   - SKILL.md mentions `retrofit-transformation.md` at least twice.

3. **Template JSON still matches what the transformation copies.** In
   `references/flow-error-handling-template.json`:
   - A top-level action `"Catch"` exists whose `runAfter` is
     `{ "Try": ["Failed", "TimedOut"] }`.
   - A top-level action `"Finally"` exists whose `runAfter` is
     `{ "Catch": ["Succeeded", "Failed", "Skipped", "TimedOut"] }`.
   - The strings `HELPER_GET_ERROR_GUID` and `HELPER_SEND_NOTIF_GUID` both occur
     exactly once.

4. **Mode A/B/C untouched.** Run `git diff HEAD~3 -- .claude/skills/power-automate-error-handling/SKILL.md`
   (the three commits from TODOs 02–04) and confirm no lines were REMOVED from the
   Step 2A, Step 2B, or Step 2C sections other than the two exact replacements
   specified in TODO 02/03 (the Q1 option C line and the Q4b heading line).

## On success

Rename this file with the `DONE-` prefix. No commit needed (nothing changed) — but if
you fixed anything during the check, commit it with message:
`Fix Mode D cross-reference found during consistency check` and note what you fixed
at the bottom of this file.
