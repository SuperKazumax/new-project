# Sweet Land Implementation Notes

## 2026-07-05: PayPay/Membership Prototype Hardening

Implemented in local app folder:

```text
C:\Users\kazuma\Desktop\AIの作業場\sweetland-server
```

Changed files:

- `server.ps1`
- `public/app.js`

## What Changed

- Added an `AdminToken` server parameter.
- Falls back to `dev-admin` for local prototype use when no token is configured.
- New users now start as `guest`, not `paid`.
- `/api/user` no longer accepts `memberStatus` self-reporting from the browser.
- Admin-only API paths now require `adminToken`.
- Browser admin actions prompt for an admin token and store it in localStorage.
- Normal user profile updates send only `userId` and `name`.

## Admin-Protected API Paths

- `/api/start-next`
- `/api/end-turn`
- `/api/skip-current`
- `/api/toggle-lane`
- `/api/queue-remove`
- `/api/queue-prioritize`
- `/api/queue-move`
- `/api/grant`
- `/api/admin-user`
- `/api/admin-user-grant`
- `/api/admin-user-reset-usage`
- `/api/config`
- `/api/broadcast`
- `/api/flow`
- `/api/rehearsal`
- `/api/emergency`
- `/api/reset`

## Verified

Manual local test on port `8797`:

- User calling `/api/user` with `memberStatus=paid` remains `guest`.
- Admin API without `adminToken` returns `403`.
- Admin API with `adminToken=dev-admin` can set user to `paid` and `approved`.

Observed result:

```text
self_member=guest
self_approval=pending
admin_no_token=403
after_member=paid
after_approval=approved
```

## Remaining Risks

- This is still prototype security.
- Default `dev-admin` must be replaced before any public test.
- State-changing APIs still support GET in the current prototype.
- Admin UI is still visible in the page; server-side token protects the action, but UI separation should still be added.
- Real authentication is still needed before public use.

