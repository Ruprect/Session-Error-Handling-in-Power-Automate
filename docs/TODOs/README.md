# Skill Review TODOs — power-automate-error-handling

This folder contains fix tasks for the skill at
`.claude/skills/power-automate-error-handling/`, produced by a review on 2026-07-01.

## How to process these TODOs (instructions for the executing model)

1. Process files in numeric order (`todo-01` first). Each file is fully self-contained —
   you do not need to read the other TODO files to complete one.
2. Each TODO specifies: the file(s) to change, the exact old text, the exact new text,
   and a verification step. Follow them literally. Do NOT improvise beyond what the TODO says.
3. After completing a TODO, rename its file by adding the prefix `DONE-`
   (e.g. `todo-01-....md` → `DONE-todo-01-....md`) so progress is visible.
4. If an exact `old` string is not found in the target file, STOP on that TODO, add a note
   at the bottom of the TODO file explaining what you found instead, and move to the next TODO.
   Do not guess at a similar-looking replacement.
5. TODOs marked **Priority: verify-first** require checking Microsoft documentation or a real
   solution export before editing. Do not apply their changes blindly.

## Overview

| # | Priority | Title |
|---|---|---|
| 01 | HIGH (runtime bug) | Fix PascalCase/lowercase mismatch in errorObject property access |
| 02 | HIGH (import failure) | Lowercase GUIDs in solution-xml.md cross-solution examples |
| 03 | HIGH (import failure) | Specify per-channel pruning for Q6 connector choices |
| 04 | MEDIUM (verify-first) | Verify Teams connector operationId/parameters and runtimeSource |
| 05 | LOW | Add TimedOut to Catch/Finally runAfter in template flow |
| 06 | MEDIUM | Clarify helper-GUID discovery in Mode B (Step 2B) |
| 07 | LOW | Update CLAUDE.md to reflect fresh-GUID policy and wired connectors |
| 08 | LOW (optional) | Clean up redundant expressions in Get Error Message helper |
