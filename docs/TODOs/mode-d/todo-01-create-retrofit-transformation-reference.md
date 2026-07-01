# TODO 01 — Create `references/retrofit-transformation.md`

**Kind:** Doc edit (content is fully specified — no judgment required)
**File to create:** `.claude/skills/power-automate-error-handling/references/retrofit-transformation.md`

## What to do

1. Open the plan file:
   `docs/superpowers/plans/2026-07-02-retrofit-error-handling-mode-d.md`
2. Find the section heading `### Task 1: Create \`references/retrofit-transformation.md\``.
3. Under its **Step 1**, there is a single fenced block opened with four backticks
   followed by `markdown` (````` ````markdown `````) and closed by four backticks.
   Copy the ENTIRE content BETWEEN those two four-backtick fence lines (do not include
   the fence lines themselves). The content begins with the line
   `# Retrofit Transformation (Mode D)` and ends with the paragraph that ends
   `operationMetadataId\` values.`
4. Write that content, byte for byte, to the new file
   `.claude/skills/power-automate-error-handling/references/retrofit-transformation.md`.
   The inner three-backtick fences (```` ```powershell ````, ```` ```json ````) are part
   of the content — keep them.

## Verification

1. The new file's first line is exactly `# Retrofit Transformation (Mode D)`.
2. The file contains all of these strings (search for each):
   - `HELPER_GET_ERROR_GUID`
   - `Failed,TimedOut`
   - `entangled-initvar`
   - `## Worked example`
3. Line count is greater than 150.
4. The file contains NO four-backtick sequences (` ```` `) — if it does, you copied the
   outer fences by mistake.

## Commit

```
git add .claude/skills/power-automate-error-handling/references/retrofit-transformation.md
git commit -m "Add retrofit transformation reference for skill Mode D"
```
