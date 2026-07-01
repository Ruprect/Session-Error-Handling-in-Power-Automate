# TODO 03 — SKILL.md: Mode D solution listing + widen Q4b

**Kind:** Doc edit — two exact string replacements
**File to change:** `.claude/skills/power-automate-error-handling/SKILL.md`

## Change 1 — add the Mode D bullet to Q3

In the Q3 block, the Mode C bullet ends with this line (note the backticks around
ErrorHandling and the 4-space indent):

```
    Ask: "Which solution contains the helper flows? (default: `ErrorHandling`)"
```

Replace that line with:

```
    Ask: "Which solution contains the helper flows? (default: `ErrorHandling`)"

- **Mode D**: Run `pac solution list` and show ALL solutions EXCEPT Microsoft/system
  ones — exclude unique names starting with `msdyn`, `msft`, `Microsoft`, `Default`,
  `Active`, or `Basic`, and canvas-app auto-solutions (`Cr` + 5 hex chars). Do NOT
  filter by publisher prefix: retrofit targets often belong to other publishers.
  Show the unfiltered list if the user asks.
  Then ask: "Which solution contains the flows to retrofit?"
```

## Change 2 — widen Q4b to Modes B and D

Replace:

- old:
  ```
  **Q4b — Helper solution** (Mode B only):
  ```
- new:
  ```
  **Q4b — Helper solution** (Modes B and D; for Mode D the default is `ErrorHandling`):
  ```

## Verification

1. Search SKILL.md for `Which solution contains the flows to retrofit` — exactly 1 match.
2. Search SKILL.md for `Modes B and D` — exactly 1 match.
3. Search SKILL.md for `(Mode B only)` — 0 matches.
4. The new Mode D bullet appears inside the Q3 section, i.e. BEFORE the line starting
   `**Q4 — Flow name(s)**`.

## Commit

```
git add .claude/skills/power-automate-error-handling/SKILL.md
git commit -m "SKILL.md: Mode D solution listing (non-Microsoft) and Q4b scope"
```
