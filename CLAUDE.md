# Porteadores Jacó — Claude Code instructions

## Mission
Audit and improve the existing Porteadores Jacó taxi-dispatch PWA without rebuilding it from scratch.

## Non-negotiable constraints
- Preserve the existing architecture and Supabase project integration.
- Do not replace the customer/central/driver flows with a different app.
- Do not reintroduce anonymous Supabase Auth for the customer flow.
- Customer public flow uses the cryptographically random `public_token` returned by the customer RPCs.
- Do not add localStorage or sessionStorage.
- Do not add a QR generator inside the PWA.
- Do not replace the Porteadores Jacó branding or approved visual identity.
- Do not remove working functionality merely to simplify the code.
- Do not create fake/demo backend functionality.
- Do not expose secrets or service-role keys in client code.
- Do not change Supabase schema/RPC security semantics unless a concrete security defect is demonstrated and the change is backward-compatible.

## Required working method
1. Inspect the entire repository first.
2. Map frontend, Supabase RPCs/schema, authentication, geolocation, realtime/polling, PWA/service worker, and deployment assumptions.
3. Run available static checks/tests/build checks before editing.
4. Identify concrete bugs, security weaknesses, UX problems, performance issues, and maintainability problems.
5. Fix only verified issues or high-value improvements consistent with the constraints above.
6. Verify every change with appropriate tests/checks.
7. Do not declare anything LIVE unless it has actually been deployed and tested end-to-end.
8. Produce a concise audit report with: findings, fixes, verification performed, remaining risks, and exact files changed.

## Important current architecture facts
- Public customer ride creation no longer depends on anonymous Auth.
- Customer RPC authorization uses `public_token`.
- `ride_requests.owner_user_id` may be null for public customer rides.
- Customer operations include create/reconnect/update-location/cancel/get-assigned-driver.
- The central dashboard has an operations UI with KPIs, map, request filters, driver grid and alerts.
- `index.html` embeds compressed copies of CSS and JS; if `styles.css` or `app.js` changes, the embedded payloads must also be updated or the browser may execute stale code.
- No deployment should be claimed from local checks alone.

## Priority order
1. Correctness and real dispatch functionality.
2. Security/privacy and Supabase authorization.
3. Geolocation reliability.
4. Central/driver operational usability.
5. Customer experience.
6. Performance/PWA reliability.
7. Visual polish.

## Do not do
- No wholesale framework migration.
- No speculative feature creep.
- No unnecessary dependencies.
- No paid third-party service requirement.
