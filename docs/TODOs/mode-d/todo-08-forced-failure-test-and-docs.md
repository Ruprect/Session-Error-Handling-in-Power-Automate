# TODO 08 — Forced-failure test + CLAUDE.md session entry

**Kind:** ENVIRONMENT REQUIRED + USER-INTERACTIVE. The failure test happens in the
Power Automate designer and needs the user. Do not fabricate results — ask the user
and record what they report.

## Steps

### 1. Designer verification (user)

Ask the user to open **Get Mock Data for Canvas App** (solution
`Flowswithouterrorhandling`) in the Power Automate designer and confirm:

- Structure renders as: variables (if any) → **Try** (the original actions) →
  **Catch** → **Finally**.
- The child-flow actions in Catch (Get Error Message) and Finally (Send
  Notification) show resolved helper flows, not broken references.

### 2. Forced failure (user)

Ask the user to temporarily add an HTTP action INSIDE the Try scope with URL
`https://api.hostname-that-does-not-exist.example/x`, save, run the flow, then
remove the action and save again.

Expected result (user reports): the run shows **Failed**, and an error email arrives
at the configured recipient showing the real DNS error message — NOT `(no details)`.

Important context: a 404/4xx response would NOT work for this test — the helper's
XPath can only extract network-level failures (DNS, connection refused). The invalid
hostname is deliberate.

### 3. CLAUDE.md session entry

Append a new session entry at the end of the `## What Was Built (Session History)`
section in `CLAUDE.md` (repo root) summarizing:

- Mode D (retrofit error handling) added to the `power-automate-error-handling`
  skill: Q1 option D, Mode D solution listing, Q4b widened, Step 2D, new
  `references/retrofit-transformation.md`
- Live-tested against the `Flowswithouterrorhandling` solution: transformation,
  import round-trip, and forced-failure notification all verified

**Scrub policy:** do NOT include email addresses, Teams IDs, tenant/environment IDs,
or URLs in the entry. Reference `.env.local` variable names instead if needed.

### 4. Commit and push

```
git add CLAUDE.md
git commit -m "Document Mode D (retrofit error handling) in session history"
```

Then ask the user whether to push to `origin main`; push only on confirmation.

## On success

Rename this file `DONE-`.
