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
- Admin UI is still visible in the page; server-side token protects the action, but UI separation should still be added.
- Real authentication is still needed before public use.

## 2026-07-05: Review After POST/API Hardening

Reviewed local app folder:

```text
C:\Users\kazuma\Desktop\AIの作業場\sweetland-server
```

Verified:

- `GET /api/state` is allowed.
- State-changing API requests require POST.
- `GET /api/join` returns `405`.
- User calling `/api/user` with `memberStatus=paid` remains `guest`.
- Admin API without `adminToken` returns `403`.
- Admin API with `adminToken=dev-admin` can set user to `paid/approved`.

Observed result:

```text
selfMember=guest
getJoin=405
adminNoToken=403
after=paid/approved
```

Remaining review findings:

- `public/app.js` still sends `memberStatus` to `/api/user`; the server ignores it, but the client should stop sending it.
- `public/index.html` still shows a user-facing `memberStatus` select; it should be removed or changed to read-only server status.
- The admin panel is visible in the general page; admin actions are server-protected, but the panel should be shown only after admin token entry.
- `dev-admin` is acceptable for local prototype only. Public tests must require `SWEETLAND_ADMIN_TOKEN`.

Next prompt for implementation Codex:

```text
レビュー結果です。POST化とadminToken保護はOKです。
次はUI整理をお願いします。

1. public/app.js の /api/user 送信から memberStatus を完全に外してください。
2. 一般ユーザー欄の memberStatus セレクトを消すか、サーバー状態の読み取り専用表示にしてください。
3. 管理パネルは管理者トークン入力後だけ表示するようにしてください。
4. dev-admin はローカル専用としてREADMEに明記し、公開テスト前は SWEETLAND_ADMIN_TOKEN 必須にしてください。
```
