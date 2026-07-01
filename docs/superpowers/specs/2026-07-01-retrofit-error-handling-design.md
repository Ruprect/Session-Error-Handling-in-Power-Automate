# Design: Mode D — Retrofit error handling onto existing flows

**Date:** 2026-07-01
**Status:** Approved by user (sections presented and confirmed in session)
**Decision:** Extend the existing `power-automate-error-handling` skill with a new Mode D.
A separate skill was rejected: it would duplicate the auth/export/import mechanics and
critical rules (guaranteeing documentation drift) and its trigger phrases would overlap
with the existing skill, making wrong-skill invocation a real failure mode.

## Goal

Add a fourth mode to `.claude/skills/power-automate-error-handling/` that takes an
**existing solution** containing flows without error handling (e.g. the test solution
`Flowswithouterrorhandling`, but any solution) and retrofits the standard
Try-Catch-Finally pattern onto selected flows in place — wiring them to the shared
helper flows in the `ErrorHandling` solution via cross-solution GUID references.

## Requirements (from user Q&A)

1. **Transformation:** full Try-Catch-Finally wrap — move existing actions into a Try
   scope, append the standard Catch (calls Helper - Get Error Message) and Finally
   (condition → Send Notification → Terminate Failed). Not the lightweight
   run-after-only pattern.
2. **Helpers:** referenced cross-solution from the shared helper solution (default
   `ErrorHandling`), discovered at run time — never copied into the target solution,
   never hardcoded.
3. **Flow selection:** the skill lists the solution's flows and the user picks;
   flows already containing a top-level `Try` scope are flagged and excluded by
   default (overridable).

## User-facing flow (Step 2D)

1. Q1 gains option **D**: "Retrofit — add Try-Catch-Finally error handling to existing
   flows in an existing solution."
2. Reused questions: Q2 (publisher prefix), Q3 (solution pick, Mode B style), a
   Q4b-style helper-solution question (default `ErrorHandling`), Q5 (auth check).
   Q6–Q8 are **not** asked — helpers are already deployed with their connectors.
3. Export the target solution (reuse Step 2B-1/2B-2 verbatim). The export zip is the
   rollback backup; restore = re-import it.
4. Discover helper GUIDs from the helper solution (reuse Step 2C-1 verbatim). Confirm
   discovered GUIDs with the user.
5. **New:** parse the exported customizations.xml, list flows with an
   already-has-Try flag, ask the user which flows to retrofit.
6. Transform each selected flow's JSON in place (algorithm below).
7. Bump solution version, rezip, import (reuse Step 2B-5 version-bump and 2B-7
   verbatim). `solution.xml` RootComponents are unchanged — the flows already belong
   to the solution.
8. Post-import: open each flow in the designer, confirm child-flow actions resolve,
   test with a forced failure.

Notification defaults match the template: `messageService=EMAIL`, `severity=ERROR`.

## Transformation algorithm (new reference file)

Documented in `references/retrofit-transformation.md`:

1. **Partition top-level actions.** `InitializeVariable` actions cannot live inside a
   Scope (platform restriction) — they stay at top level in original order.
   Everything else moves into a new `Try` scope.
2. **Rewire runAfter.**
   - Top-level `InitializeVariable` actions are re-chained sequentially in their
     original order (first gets `runAfter: {}`, each subsequent one runs after the
     previous `["Succeeded"]`) — this drops any dependency they had on a moved
     action while keeping initialization order deterministic.
   - `Try` runs after the last `InitializeVariable` (`["Succeeded"]`), or
     `runAfter: {}` if the flow has none.
   - Moved actions keep their runAfter relationships with each other. A dependency
     on an action now outside the scope (an InitializeVariable) is removed; if that
     empties the runAfter, the action becomes a Try entry point.
3. **Append Catch and Finally,** copied from
   `references/flow-error-handling-template.json` (single source of truth — no
   duplicated scope JSON), with exactly two kinds of substitution:
   - discovered helper GUIDs into the `workflowReferenceName` fields
   - fresh `operationMetadataId` GUIDs for the newly added actions only
   Catch: `runAfter Try ["Failed", "TimedOut"]`. Finally:
   `runAfter Catch ["Succeeded", "Failed", "Skipped", "TimedOut"]`.
4. **Skip rules.** Flagged and excluded by default (overridable only for the first
   case):
   - Flows with an existing top-level `Try` scope — already has error handling.
   - Flows with ANY top-level action named `Try`, `Catch`, or `Finally` regardless
     of type — appending the new scopes would collide on action names.
   - Flows where any `InitializeVariable`'s `runAfter` references a
     non-`InitializeVariable` action, or whose variable `value` expression
     references a moved action's outputs — re-chaining would change semantics or
     break at runtime. These need manual retrofitting.
   - Flows with no actions: skipped outright.
5. **Preserved untouched:** flow GUID (in-place update — no re-linking), trigger,
   `connectionReferences` object, every existing action's name and
   `operationMetadataId`. A `Response` action moving into Try is acceptable: if Try
   fails before responding, the caller sees a failure, which is correct.

## Skill file changes

| File | Change |
|---|---|
| `SKILL.md` | Q1 option D; a ~40-line Step 2D section pointing at reused steps 2B-1/2B-2/2C-1/2B-5/2B-7 and the new reference file; frontmatter description gains trigger phrases ("add error handling to existing flows", "retrofit error handling", "wrap my flows in try-catch") |
| `references/retrofit-transformation.md` | New — the transformation algorithm above, with a worked before/after JSON example |

No other files change. No changes to Modes A/B/C behavior.

## Testing

Run Mode D against `Flowswithouterrorhandling` (contains one Draft flow,
*Get Mock Data for Canvas App*, GUID `68d65bb2-9575-f111-ab0d-70a8a532f695`).
The fixture exists only in the Development environment, not the repo — verify it is
still present before the test run (query Dataverse) and recreate it if not:

1. Import succeeds; solution version bumped.
2. Post-import export shows the flow's structure as: variables (if any) → Try
   (original actions) → Catch → Finally, with the discovered helper GUIDs in the
   child-flow actions and original action metadata untouched.
3. Designer renders the flow without broken references.
4. A forced failure inside Try produces the email notification with real error
   details (not `(no details)`).

## Out of scope (YAGNI)

- Retrofitting flows that already have partial error handling (custom runAfter
  chains) — they are flagged and skipped, not merged.
- Per-flow notification channel/severity choices — defaults match the template;
  change in the designer afterwards if needed.
- Undo command — the export zip is the documented rollback path.
