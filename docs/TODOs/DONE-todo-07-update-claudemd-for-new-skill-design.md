# TODO 07 — Update CLAUDE.md to reflect fresh-GUID policy and wired connectors

**Priority:** LOW — documentation drift. CLAUDE.md still describes the OLD skill design
(fixed helper GUIDs, placeholder send actions) which now contradicts the skill.
**File to change:** `CLAUDE.md` (repo root)
**Type of change:** Small documentation edits. Preserve everything not listed below.

## Context

The skill was changed (2026-07-01, session 8) so that:
1. GUIDs are **generated fresh per deployment** with `[System.Guid]::NewGuid()` — never reused,
   because re-using a GUID after deleting a flow causes import conflicts with orphaned references.
   The three GUIDs listed in CLAUDE.md (`3fc9a2b1...`, `5ae8b3c2...`, `7cd4e5f6...`) are the GUIDs
   of the CURRENTLY-DEPLOYED flows in the Development environment, but are no longer hardcoded in
   the skill. Cross-solution references now discover current GUIDs by exporting the helper solution.
2. "Helper - Send Notification" now ships with REAL connector actions (Office 365
   `Send an Email (V2)` and Teams `Post adaptive card in a chat or channel`) plus connection
   references (`{prefix}_office365_errorhandling`, `{prefix}_teams_errorhandling`), configured by
   skill questions Q6–Q8. The placeholder-Compose approach only remains for channels the user
   chooses not to wire.

## Exact changes to CLAUDE.md

1. In the section `### Fixed GUIDs` — WAIT: this heading is in SKILL.md history, not CLAUDE.md.
   In CLAUDE.md, find the table under `### Flows in the Solution` and the per-flow `**GUID:**`
   lines. Do NOT delete the GUIDs (they document what is currently deployed). Instead, add this
   sentence directly under the `### Flows in the Solution` heading:

   > **Note (Session 8):** These GUIDs identify the flows currently deployed in the Development
   > environment. The skill no longer hardcodes them — each fresh deployment generates new GUIDs,
   > and cross-solution references discover the current GUIDs by exporting the helper solution.

2. In the section `#### 3. Helper - Send Notification`, find the sentence in the **Purpose** line:
   `The actual send action is a placeholder — callers wire up their own Email/Teams connector.`
   Replace it with:
   `The send actions are real connector calls (Office 365 Send an Email V2, Teams Post adaptive card) bound via connection references; the skill can alternatively emit placeholder Compose actions if the user opts out of a channel.`

3. In `## Known Limitations / Next Steps`, the first bullet (`**Wire up send actions:** ...`)
   describes replacing placeholder Composes. Replace that whole bullet with:
   `- **Connection references after import:** After importing a solution with wired connectors, link the connection references ({prefix}_office365_errorhandling, {prefix}_teams_errorhandling) to real connections in the solution's Connection References page.`

4. Append a new session entry at the end of `## What Was Built (Session History)`:

   ```markdown
   ### Session 8
   - Reworked the `power-automate-error-handling` skill:
     - Fresh GUIDs generated per deployment (no more fixed GUIDs — deleting and re-creating flows with reused GUIDs caused import conflicts)
     - "Helper - Send Notification" reference JSON now contains real Office 365 Send Email (V2) and Teams Post Adaptive Card actions with connection references; skill Q6–Q8 gather channel choice, email recipient, and Teams group/channel IDs
     - Mode C discovers current helper GUIDs by exporting the installed helper solution and parsing customizations.xml
   - Reviewed the skill and produced fix TODOs in `todos/` (case-mismatch bug, lowercase-GUID doc fix, per-channel pruning spec, Teams parameter verification, TimedOut runAfter, Mode B GUID discovery)
   ```

## Verification

1. CLAUDE.md still contains the three GUID values (as documentation of the deployed state).
2. CLAUDE.md no longer claims the send action is "a placeholder" as its primary description.
3. A `### Session 8` entry exists.
