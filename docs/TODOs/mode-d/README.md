# Mode D Implementation TODOs — power-automate-error-handling

This folder contains implementation tasks for adding **Mode D (Retrofit error handling
onto existing flows)** to the skill at `.claude/skills/power-automate-error-handling/`.

Source documents (read only if a TODO explicitly points at them):
- Plan: `docs/superpowers/plans/2026-07-02-retrofit-error-handling-mode-d.md`
- Spec: `docs/superpowers/specs/2026-07-01-retrofit-error-handling-design.md`

## How to process these TODOs (instructions for the executing model)

1. Process files in numeric order (`todo-01` first). Each file states exactly what to
   change and how to verify it. Follow instructions literally. Do NOT improvise beyond
   what the TODO says.
2. After completing a TODO, rename its file by adding the prefix `DONE-`
   (e.g. `todo-01-....md` → `DONE-todo-01-....md`) so progress is visible.
3. If an exact `old` string is not found in the target file, STOP on that TODO, add a
   note at the bottom of the TODO file explaining what you found instead, and move to
   the next TODO. Do not guess at a similar-looking replacement.
4. Commit after each TODO with the commit message given in the TODO. End every commit
   message with the line:
   `Co-Authored-By: Claude <noreply@anthropic.com>`
5. TODOs marked **ENVIRONMENT REQUIRED** need an authenticated pac CLI session against
   the Development environment and live Dataverse access. If you cannot run pac
   commands, STOP at that TODO and leave the remaining ones for a supervised session.
   Do not mark them DONE without executing them.

## Overview

| # | Kind | Title |
|---|---|---|
| 01 | Doc edit | Create `references/retrofit-transformation.md` from the plan |
| 02 | Doc edit | SKILL.md: frontmatter trigger phrases + Q1 option D |
| 03 | Doc edit | SKILL.md: Mode D solution listing + widen Q4b |
| 04 | Doc edit | SKILL.md: insert Step 2D section from the plan |
| 05 | Verification | Cross-file consistency check (no edits) |
| 06 | ENVIRONMENT REQUIRED | Live test: export, discover GUIDs, transform fixture flow |
| 07 | ENVIRONMENT REQUIRED | Live test: import and round-trip verification |
| 08 | ENVIRONMENT REQUIRED + user-interactive | Forced-failure test + CLAUDE.md entry |
