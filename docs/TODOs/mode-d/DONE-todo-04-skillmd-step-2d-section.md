# TODO 04 — SKILL.md: insert Step 2D section

**Kind:** Doc edit — one insertion, content copied from the plan
**File to change:** `.claude/skills/power-automate-error-handling/SKILL.md`

## What to do

1. Open the plan file:
   `docs/superpowers/plans/2026-07-02-retrofit-error-handling-mode-d.md`
2. Find the section heading `### Task 4: SKILL.md — Step 2D section`.
3. Under its **Step 1**, there is a single fenced block opened with four backticks
   followed by `markdown` and closed by four backticks. Copy the ENTIRE content
   between those fence lines (excluding the fence lines). The content begins with
   `## Step 2D — Retrofit error handling onto existing flows` and ends with a `---`
   horizontal rule line.
4. In SKILL.md, find the line:
   ```
   ## Step 3 — Post-import
   ```
5. Insert the copied content directly ABOVE that line, followed by a blank line,
   so `## Step 3 — Post-import` remains on its own line after the inserted block.

## Verification

1. Search SKILL.md for `## Step 2D` — exactly 1 match.
2. Each of `### 2D-1`, `### 2D-2`, `### 2D-3`, `### 2D-4`, `### 2D-5`, `### 2D-6` —
   exactly 1 match each.
3. `## Step 2D` appears at a LOWER line number than `## Step 3 — Post-import`.
4. Search SKILL.md for `retrofit-transformation.md` — at least 2 matches
   (in the "Read these files" list and in 2D-3/2D-4).

## Commit

```
git add .claude/skills/power-automate-error-handling/SKILL.md
git commit -m "SKILL.md: add Step 2D retrofit workflow"
```
