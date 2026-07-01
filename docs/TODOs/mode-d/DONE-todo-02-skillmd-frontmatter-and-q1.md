# TODO 02 — SKILL.md: frontmatter trigger phrases + Q1 option D

**Kind:** Doc edit — two exact string replacements
**File to change:** `.claude/skills/power-automate-error-handling/SKILL.md`

## Change 1 — frontmatter description

In the YAML frontmatter at the top of the file, replace:

- old:
  ```
    or scaffold error handling helpers. Triggers on phrases like "set up error
  ```
- new:
  ```
    or scaffold error handling helpers, or add error handling to EXISTING flows
    ("add error handling to existing flows", "retrofit error handling", "wrap my
    flows in try-catch", "add try-catch to flows in a solution"). Triggers on phrases like "set up error
  ```

(The description is a folded YAML string — the inserted lines keep the same two-space
indentation as the surrounding lines.)

## Change 2 — Q1 option D

Replace:

- old:
  ```
  > (C) New solution, shared helpers — create a brand-new solution whose flows call the helper flows that live in a different solution"
  ```
- new:
  ```
  > (C) New solution, shared helpers — create a brand-new solution whose flows call the helper flows that live in a different solution
  > (D) Retrofit — add Try-Catch-Finally error handling to existing flows in an existing solution"
  ```

Note: the closing double-quote moves from the end of the (C) line to the end of the
new (D) line. The `—` characters are em dashes, not hyphens — copy them exactly.

## Verification

1. Search SKILL.md for `(D) Retrofit` — exactly 1 match.
2. Search SKILL.md for `retrofit error handling` — exactly 1 match (in the frontmatter).
3. Search SKILL.md for `different solution"` — 0 matches (the quote moved to the D line).
4. The YAML frontmatter must still parse: the file starts with `---`, and the
   `description: >` block lines are all indented by at least two spaces.

## Commit

```
git add .claude/skills/power-automate-error-handling/SKILL.md
git commit -m "SKILL.md: add Mode D to Q1 and retrofit trigger phrases"
```
