# Claude Code — Audit and improvement task

You are the second engineering reviewer for the existing Porteadores Jacó taxi-dispatch PWA.

Work on the repository as it exists. Do NOT rebuild it from scratch.

## Goal
Perform a production-oriented audit and, where justified, improve the existing project so it is stronger for a real sales demonstration and eventual LIVE operation.

## First: understand before changing
Inspect all source files, especially:
- `index.html`
- `app.js`
- `styles.css`
- `config.js`
- `manifest.webmanifest`
- `sw.js`
- `supabase/schema.sql`
- `supabase/optional_cron.sql`
- `README.md`
- `AUDIT-V8.md`

Trace the actual customer, central and driver flows end-to-end.

## Mandatory audit areas
### 1. Customer flow
Verify:
- QUIERO UN TAXI triggers browser geolocation correctly.
- Public customer ride creation does not require anonymous Auth.
- The returned `public_token` is retained in the current application flow without browser local/session storage.
- Reconnect, location update, cancellation and assigned-driver lookup use the token correctly.
- GPS errors are handled clearly.

### 2. Supabase security
Inspect every customer RPC and RLS/policy involved.
Look for:
- privilege escalation
- token leakage
- weak authorization checks
- unsafe SECURITY DEFINER behavior
- search_path issues
- unintended anon access
- exposed service-role credentials

Do not weaken the public customer flow merely because it uses anon access. Instead, recommend concrete hardening where appropriate.

### 3. Central dispatch
Verify:
- requests render correctly
- assignment/reassignment/cancellation operations match backend behavior
- driver availability/GPS status is coherent
- stale GPS is detected correctly
- map markers and operational state stay synchronized
- polling/realtime cannot create obvious race conditions

### 4. Driver flow
Verify:
- online/offline state
- location permission/error handling
- assigned rides
- accept/reject/update behavior
- map and GPS state

### 5. PWA integrity
Verify:
- service worker behavior
- manifest
- installability assumptions
- cache strategy
- stale-cache risk
- offline behavior versus claims made by the UI

### 6. Embedded build architecture
This project embeds compressed CSS and JavaScript inside `index.html`.
If you modify `app.js` or `styles.css`, update the embedded payloads in `index.html` and verify that decoded embedded files match the external source files.

### 7. UX / professional quality
Improve only where it materially improves the real dispatch workflow. Do not redesign the brand.

## Verification requirements
After edits:
- run syntax/static checks available in the repository
- verify the embedded JS/CSS payloads are synchronized
- verify no `signInAnonymously`, `ensureAnon`, `localStorage`, or `sessionStorage` was accidentally reintroduced
- verify no secret/service-role key is client-exposed
- verify ZIP/build integrity if a package is produced
- report exactly what was tested and what could not be tested

## Output
Return:
1. Executive verdict: READY / NEEDS FIXES / BLOCKED
2. Findings grouped by severity
3. Exact fixes made
4. Tests/checks actually run and results
5. Remaining risks before LIVE
6. Files changed
7. Recommended next action

Do not fabricate browser/device/end-to-end results you did not actually perform.
