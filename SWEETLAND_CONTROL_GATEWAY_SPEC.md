# Sweet Land Control Gateway Spec

## Purpose

WebアプリからSweet Land 4実機へ直接命令を出さない。  
間に制御ゲートウェイを置き、安全確認、連打制限、緊急停止、ログ記録を行う。

## Position

```text
Viewer browser
↓
Sweet Land web server
↓
Control gateway
↓
Actuator controller
↓
External servo / solenoid
↓
Sweet Land 4 physical buttons
```

## First Target

最初の実機連携では、実機内部の配線を触らず、外付けサーボまたはソレノイドで既存ボタンを押す。

制御対象:

- left button
- right button
- scoop button
- actuator power enable
- emergency stop

## Responsibilities

Control gatewayが担当すること:

- Webサーバーからの操作命令を受ける。
- 操作権がある命令だけ実行する。
- 緊急停止中は全操作を拒否する。
- 1回の操作時間を制限する。
- 連打を制限する。
- 実機へ出した命令をログに残す。
- アクチュエータの状態を返す。
- 通信切断時に安全側へ倒す。

Control gatewayが担当しないこと:

- YouTubeログイン管理
- 会員判定
- 決済
- 順番待ちUI
- 配信画面

## Recommended Runtime

最初はローカルネットワーク内で動かす。

候補:

- Raspberry Pi
- 小型Windows PC
- 既存PC上のローカルプロセス

最初の推奨:

```text
既存PCでモック動作
↓
Raspberry Piでサーボ制御
↓
実機近くに固定
```

## Network Rule

Control gatewayはインターネットへ直接公開しない。

許可する接続:

- Sweet Land web serverからのローカル接続
- 管理者PCからのローカル確認

拒否する接続:

- 視聴者ブラウザからの直接接続
- 外部インターネットからの直接接続

## Gateway State

```json
{
  "ok": true,
  "mode": "safe",
  "emergencyStopped": false,
  "actuatorPower": false,
  "lastCommand": null,
  "lastCommandAt": null,
  "cooldownUntil": null,
  "hardwareConnected": false
}
```

## Modes

## Safe

実機へ出力しない状態。開発、画面確認、リハーサル用。

- 命令は受け取る。
- ログには残す。
- 実機は動かさない。

## Armed

実機へ出力できる状態。

- 管理者が明示的に切り替える。
- 緊急停止解除済みであること。
- カメラ確認済みであること。

## Emergency

全操作停止。

- アクチュエータ電源を切る。
- 全コマンドを拒否する。
- 管理者だけが解除できる。

## Commands

## Move Left

```json
{
  "command": "left",
  "durationMs": 150,
  "turnId": "turn-123",
  "userId": "user-123"
}
```

## Move Right

```json
{
  "command": "right",
  "durationMs": 150,
  "turnId": "turn-123",
  "userId": "user-123"
}
```

## Scoop

```json
{
  "command": "scoop",
  "durationMs": 500,
  "turnId": "turn-123",
  "userId": "user-123"
}
```

## Stop

```json
{
  "command": "stop",
  "reason": "admin_stop"
}
```

## Limits

初期値。

| Command | Min | Max | Cooldown |
| --- | --- | --- | --- |
| left | 50ms | 300ms | 100ms |
| right | 50ms | 300ms | 100ms |
| scoop | 200ms | 1200ms | 1000ms |
| stop | 0ms | 0ms | 0ms |

ルール:

- `durationMs` が上限を超える場合は拒否する。
- クールダウン中の命令は拒否する。
- `scoop` 後はターン終了扱いにするのはWebサーバー側。

## API

## GET /health

ゲートウェイの生存確認。

Response:

```json
{
  "ok": true,
  "hardwareConnected": true,
  "mode": "safe"
}
```

## GET /state

現在状態。

Response:

```json
{
  "ok": true,
  "mode": "armed",
  "emergencyStopped": false,
  "actuatorPower": true,
  "cooldownUntil": null
}
```

## POST /command

実機操作。

Request:

```json
{
  "command": "left",
  "durationMs": 150,
  "turnId": "turn-123",
  "userId": "user-123",
  "serverToken": "..."
}
```

Response:

```json
{
  "ok": true,
  "executed": true,
  "mode": "armed"
}
```

## POST /mode

管理者がゲートウェイのモードを切り替える。

Request:

```json
{
  "mode": "armed",
  "adminToken": "..."
}
```

Allowed modes:

- `safe`
- `armed`
- `emergency`

## POST /emergency-stop

即時停止。

Request:

```json
{
  "reason": "camera_lost",
  "adminToken": "..."
}
```

## POST /reset

緊急停止後の復旧準備。  
実機の状態確認後に管理者だけが実行する。

Request:

```json
{
  "adminToken": "..."
}
```

## Authentication

Control gatewayは最低2種類のトークンを持つ。

- `serverToken`: Webサーバーからの操作命令用
- `adminToken`: 管理者操作用

ルール:

- 視聴者ブラウザにトークンを渡さない。
- `serverToken` はWebサーバーだけが知る。
- `adminToken` は管理者だけが知る。
- ログにトークンを出さない。

## Rejection Rules

次の場合は操作しない。

- emergency mode
- safe modeで実機出力を要求された
- actuator power off
- unknown command
- duration too long
- cooldown active
- missing serverToken
- invalid serverToken
- missing turnId
- missing userId
- hardware disconnected

Example:

```json
{
  "ok": false,
  "error": "cooldown_active"
}
```

## Logging

最低限残す。

```json
{
  "at": "2026-07-05T12:00:00Z",
  "command": "left",
  "durationMs": 150,
  "turnId": "turn-123",
  "userId": "user-123",
  "result": "executed"
}
```

ログに残すもの:

- command
- durationMs
- turnId
- userId
- result
- error
- mode
- timestamp

ログに残さないもの:

- serverToken
- adminToken
- 個人情報

## Hardware Output Strategy

## Mock Driver

開発用。

- 実機を動かさない。
- コマンドをログに残す。
- テストが簡単。

## Servo Driver

最初の実機用。

- サーボで既存ボタンを押す。
- 実機内部配線に触らない。
- ボタンごとに押す角度と戻す角度を設定する。

## Relay Driver

後の段階。

- 実機のボタン入力をリレーで代替する。
- 実機仕様を確認してから使う。
- 絶縁を必須にする。

## Fail-Safe Behavior

安全側に倒す条件:

- Webサーバーから一定時間ハートビートがない。
- 管理者が緊急停止した。
- ハードウェア出力に失敗した。
- カメラ確認が取れない。
- ゲートウェイ内部エラー。

安全側の動作:

- 実機出力停止
- actuator power off
- modeを `emergency` または `safe` にする
- ログ記録

## Heartbeat

Webサーバーは定期的にゲートウェイへハートビートを送る。

```json
{
  "serverToken": "...",
  "streamPhase": "accepting_entries",
  "emergencyStopped": false
}
```

推奨:

- interval: 2秒
- timeout: 6秒

timeout時:

- ゲートウェイは実機操作を停止する。
- modeを `safe` に戻す。

## Implementation Tasks

1. Mock gatewayを作る。
2. WebサーバーからMock gatewayへ操作命令を送る。
3. Mock gatewayで拒否条件をテストする。
4. サーボ1個で1ボタンだけ押す。
5. left/right/scoopの3ボタンへ拡張する。
6. 物理緊急停止を追加する。
7. リハーサルテストを通す。

## Acceptance Tests

- safe modeでは実機が動かない。
- armed modeでは有効なコマンドだけ実行される。
- emergency modeでは全操作が拒否される。
- cooldown中の連打が拒否される。
- duration上限超過が拒否される。
- serverTokenなしの操作が拒否される。
- adminTokenなしのmode変更が拒否される。
- heartbeat timeoutでsafeに戻る。
- すべての操作がログに残る。

