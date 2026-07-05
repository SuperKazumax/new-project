# Sweet Land Mock Gateway Implementation Plan

## Purpose

実機をまだ動かさずに、Webサーバーから制御ゲートウェイへ命令を送る流れを確認する。

Mock gatewayは安全な開発用ゲートウェイ。  
コマンドを受け取り、検証し、ログに残すが、実機やサーボは動かさない。

## Scope

対象:

- 新規Mock gateway
- Sweet Land web serverからMock gatewayへの接続
- gatewayログ
- safe / armed / emergency mode
- トークン検証
- 連打制限
- 操作時間上限

対象外:

- 実サーボ制御
- 実機内部配線
- 会員管理
- 課金
- YouTubeログイン
- ユーザー許可UI

## Recommended Folder

実装本体側に追加する。

```text
C:\Users\kazuma\Desktop\AIの作業場\sweetland-gateway
```

または、既存の `sweetland-server` 配下に置く場合:

```text
C:\Users\kazuma\Desktop\AIの作業場\sweetland-server\gateway
```

どちらでもよいが、最初は `sweetland-gateway` として分けたほうが安全。

## Phase 1: Standalone Mock Gateway

### Goal

単体で起動し、HTTP APIに応答する。

### Files

候補:

- `gateway.ps1`
- `README.md`
- `gateway-state.json` optional
- `gateway.log` optional

### API

- `GET /health`
- `GET /state`
- `POST /command`
- `POST /mode`
- `POST /emergency-stop`
- `POST /reset`

### Acceptance

- `GET /health` が `ok: true` を返す。
- `GET /state` が現在modeを返す。
- 初期modeは `safe`。
- `POST /command` はsafe modeでは実行せず、ログだけ残す。

## Phase 2: Token Validation

### Goal

視聴者や外部から直接操作できないようにする。

### Tokens

- `serverToken`: 操作用
- `adminToken`: mode変更、緊急停止、reset用

### Rules

- `/command` は `serverToken` 必須。
- `/mode`、`/emergency-stop`、`/reset` は `adminToken` 必須。
- 不正トークンは `403 invalid_token`。
- トークンはログに残さない。

### Acceptance

- tokenなしの `/command` は拒否される。
- 間違ったtokenの `/command` は拒否される。
- 正しい `serverToken` の `/command` は受け付ける。
- 正しい `adminToken` の `/mode` は受け付ける。

## Phase 3: Modes

### Goal

安全状態を明確にする。

### Modes

- `safe`: 実機出力なし。ログのみ。
- `armed`: 実機出力予定として扱う。Mockでは `executed: true` を返す。
- `emergency`: 全操作拒否。

### Rules

- 初期値は `safe`。
- `safe` ではコマンドを受け取るが `executed: false`。
- `armed` では有効コマンドを `executed: true`。
- `emergency` では全コマンドを `423 emergency_stopped` または `409 emergency_stopped` で拒否。

### Acceptance

- `safe -> armed -> emergency -> safe` と切り替えられる。
- emergency中はleft/right/scoopが全部拒否される。

## Phase 4: Command Validation

### Goal

危険な命令や壊れた命令を実機へ通さない。

### Allowed Commands

- `left`
- `right`
- `scoop`
- `stop`

### Required Fields

For `left`, `right`, `scoop`:

- `command`
- `durationMs`
- `turnId`
- `userId`
- `serverToken`

For `stop`:

- `command`
- `reason`
- `serverToken` or `adminToken`

### Duration Limits

| Command | Min | Max |
| --- | --- | --- |
| left | 50ms | 300ms |
| right | 50ms | 300ms |
| scoop | 200ms | 1200ms |

### Acceptance

- unknown command is rejected.
- missing `turnId` is rejected.
- missing `userId` is rejected.
- too long duration is rejected.
- too short duration is rejected.

## Phase 5: Cooldown

### Goal

連打や暴走操作を防ぐ。

### Initial Cooldowns

| Command | Cooldown |
| --- | --- |
| left | 100ms |
| right | 100ms |
| scoop | 1000ms |
| stop | 0ms |

### Rules

- コマンド実行後、同じgateway全体にcooldownを設定する。
- cooldown中のコマンドは `429 cooldown_active`。
- `stop` はcooldownを無視して通せる。

### Acceptance

- left直後のleftは拒否される。
- scoop直後のscoopは拒否される。
- cooldown後は再度通る。
- stopはいつでも通る。

## Phase 6: Logging

### Goal

何が起きたか後で追えるようにする。

### Log Entry

```json
{
  "at": "2026-07-05T12:00:00Z",
  "type": "command",
  "command": "left",
  "durationMs": 150,
  "turnId": "turn-123",
  "userId": "user-123",
  "mode": "armed",
  "result": "executed"
}
```

### Log Events

- command accepted
- command rejected
- mode changed
- emergency stop
- reset
- heartbeat timeout

### Do Not Log

- serverToken
- adminToken
- private user details

### Acceptance

- accepted command appears in logs.
- rejected command appears in logs.
- mode changes appear in logs.
- tokens do not appear in logs.

## Phase 7: Web Server Integration

### Goal

Sweet Land web serverからMock gatewayへ操作命令を送る。

### Integration Point

Web serverの `/api/command` が成功判定する前に、gatewayへ命令を送る。

Flow:

```text
browser command
↓
web server validates user turn
↓
web server sends command to gateway
↓
gateway accepts/rejects
↓
web server updates turn state
```

### Rules

- Web serverが操作権を確認する。
- Gatewayが物理安全を確認する。
- Gatewayが拒否した場合、Web serverもユーザー操作を失敗扱いにする。
- `scoop` はgateway成功後にターン終了する。

### Acceptance

- 操作権のないユーザーはWeb server側で拒否される。
- 操作権のあるユーザーのleft/right/scoopがgatewayへ送られる。
- gateway emergency中はWeb server操作も失敗になる。
- gatewayログに操作が残る。

## Phase 8: Heartbeat

### Goal

Web serverが止まった時にgatewayが安全側へ戻る。

### API

`POST /heartbeat`

Request:

```json
{
  "serverToken": "...",
  "streamPhase": "accepting_entries",
  "emergencyStopped": false
}
```

### Rules

- Web serverは2秒ごとに送る。
- 6秒以上来なければgatewayは `safe` に戻る。
- emergencyStoppedがtrueならgatewayもemergencyへ入る。

### Acceptance

- heartbeatが来ている間は現在modeを維持する。
- heartbeat timeoutでsafeへ戻る。
- emergency heartbeatでemergencyになる。

## Manual Test Commands

PowerShell例。

```powershell
Invoke-RestMethod http://127.0.0.1:8790/health
```

```powershell
Invoke-RestMethod http://127.0.0.1:8790/state
```

```powershell
Invoke-RestMethod http://127.0.0.1:8790/mode -Method POST -ContentType "application/json" -Body '{"mode":"armed","adminToken":"dev-admin"}'
```

```powershell
Invoke-RestMethod http://127.0.0.1:8790/command -Method POST -ContentType "application/json" -Body '{"command":"left","durationMs":150,"turnId":"turn-1","userId":"user-1","serverToken":"dev-server"}'
```

```powershell
Invoke-RestMethod http://127.0.0.1:8790/emergency-stop -Method POST -ContentType "application/json" -Body '{"reason":"manual_test","adminToken":"dev-admin"}'
```

## Implementation Order For Codex

1. Create standalone Mock gateway.
2. Add `/health` and `/state`.
3. Add mode state and `/mode`.
4. Add token validation.
5. Add `/command` with validation only.
6. Add mode behavior.
7. Add cooldown.
8. Add logs.
9. Add heartbeat.
10. Integrate Sweet Land web server `/api/command` with gateway.

## Done Definition

- Mock gateway runs locally.
- Web server can send left/right/scoop to it.
- No real hardware moves.
- Logs show accepted and rejected commands.
- emergency mode blocks all commands.
- safe mode never outputs to hardware.
- Implementation can later swap Mock driver for Servo driver.

