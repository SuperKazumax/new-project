# Sweet Land GitHub Issues

GitHub Issuesに貼り付けるためのタスク一覧。  
必要に応じて1つずつIssue化する。

## Issue 1: 配信フェーズ状態を追加する

### Title

```text
Add stream phase state and admin flow buttons
```

### Body

```markdown
## Goal

配信中の状態を `21:00 開始 → 受付開始 → ラスト募集 → 終了準備 → 配信終了` の流れで管理できるようにする。

## References

- `SWEETLAND_STREAM_FLOW.md`
- `SWEETLAND_PUBLIC_COPY.md`

## Tasks

- [ ] サーバー状態に `streamPhase` を追加する
- [ ] `before_start`, `live_started`, `accepting_entries`, `last_call`, `closing`, `ended` を扱う
- [ ] 管理者だけが状態変更できる
- [ ] 状態ごとに次の主ボタンを出す
- [ ] 視聴者向けメッセージを状態ごとに切り替える
- [ ] 状態変更をログに残す

## Acceptance

- [ ] `受付開始` 後だけ新規順番待ちできる
- [ ] `終了準備` 後は新規順番待ちできない
- [ ] `配信終了` 後は操作できない
- [ ] 緊急停止中はどの状態でも操作できない
```

## Issue 2: ユーザー管理と手動許可を完成させる

### Title

```text
Add manual user permission and membership simulation
```

### Body

```markdown
## Goal

管理者がユーザーの参加可否とプレイ回数を手動で管理できるようにする。

## References

- `SWEETLAND_PAYPAY_MEMBERSHIP_PLAN.md`
- `SWEETLAND_TASKS.md`

## Tasks

- [ ] ユーザー一覧を管理画面に出す
- [ ] guest / allowed / paid などの状態を扱う
- [ ] 管理者が手動で参加許可できる
- [ ] 管理者がプレイ回数を付与できる
- [ ] ユーザーが自分で `paid` を名乗れない
- [ ] 参加拒否理由を画面に出す

## Acceptance

- [ ] 一般ユーザーは勝手に有料会員扱いになれない
- [ ] 管理者だけが許可と回数調整できる
- [ ] 許可されていないユーザーは順番待ちできない
```

## Issue 3: Mock gatewayを実装する

### Title

```text
Implement standalone Mock control gateway
```

### Body

```markdown
## Goal

実機を動かさず、Webサーバーから制御ゲートウェイへ操作命令を送る流れを確認する。

## References

- `SWEETLAND_CONTROL_GATEWAY_SPEC.md`
- `SWEETLAND_MOCK_GATEWAY_IMPLEMENTATION_PLAN.md`

## Tasks

- [ ] standalone Mock gatewayを作る
- [ ] `GET /health` を実装する
- [ ] `GET /state` を実装する
- [ ] `POST /command` を実装する
- [ ] `POST /mode` を実装する
- [ ] `POST /emergency-stop` を実装する
- [ ] `POST /reset` を実装する
- [ ] `serverToken` と `adminToken` を検証する
- [ ] safe / armed / emergency modeを実装する
- [ ] duration上限を実装する
- [ ] cooldownを実装する
- [ ] 操作ログを残す

## Acceptance

- [ ] safe modeでは `executed: false`
- [ ] armed modeでは有効コマンドが `executed: true`
- [ ] emergency modeでは全操作拒否
- [ ] tokenなし操作は拒否
- [ ] cooldown中の連打は拒否
- [ ] ログに操作結果が残る
```

## Issue 4: WebサーバーとMock gatewayを接続する

### Title

```text
Connect web server commands to Mock gateway
```

### Body

```markdown
## Goal

Webアプリの操作命令をMock gatewayへ送り、gateway側の安全判定を通してから状態更新する。

## References

- `SWEETLAND_MOCK_GATEWAY_IMPLEMENTATION_PLAN.md`
- `SWEETLAND_CONTROL_GATEWAY_SPEC.md`

## Tasks

- [ ] Webサーバーにgateway URL設定を追加する
- [ ] `/api/command` からgateway `/command` へ送信する
- [ ] gateway拒否時はユーザー操作も失敗扱いにする
- [ ] `scoop` はgateway成功後にターン終了する
- [ ] gateway状態を管理画面に表示する
- [ ] gateway emergency中は操作できない

## Acceptance

- [ ] 操作権ありユーザーのleft/right/scoopがgatewayへ送られる
- [ ] gatewayが拒否した操作はWeb側でも失敗になる
- [ ] gatewayログに操作が残る
- [ ] 実機なしで通しテストできる
```

## Issue 5: 配信前リハーサルを実施する

### Title

```text
Run pre-stream rehearsal and record results
```

### Body

```markdown
## Goal

実機なし、または安全モードで配信当日の流れを通し確認する。

## References

- `SWEETLAND_REHEARSAL_TEST_SHEET.md`
- `SWEETLAND_LIVE_RUNBOOK.md`

## Tasks

- [ ] YouTube限定公開でテスト配信する
- [ ] 21:00開始から配信終了まで通す
- [ ] 2人以上で順番待ちを試す
- [ ] 緊急停止を試す
- [ ] スマホで参加する
- [ ] 結果を `SWEETLAND_REHEARSAL_TEST_SHEET.md` に記録する
- [ ] HOLD/NGを次回TODOにする

## Acceptance

- [ ] 主要フローが最後まで通る
- [ ] 緊急停止手順を全員が理解している
- [ ] 公開前に直すべき問題が一覧化されている
```

## Issue 6: 視聴者向けトップページを整える

### Title

```text
Add viewer-facing Sweet Land live page copy and layout
```

### Body

```markdown
## Goal

初めて来た視聴者が、何のサイトか、どう参加するか、今参加できるかをすぐ理解できるようにする。

## References

- `SWEETLAND_PUBLIC_COPY.md`
- `SWEETLAND_STREAM_FLOW.md`

## Tasks

- [ ] YouTube配信エリアを用意する
- [ ] 現在の配信状態を大きく表示する
- [ ] 遊び方3ステップを表示する
- [ ] 参加条件を表示する
- [ ] 回数上限を表示する
- [ ] 受付中だけ参加導線を強調する
- [ ] 終了時は次回案内に切り替える

## Acceptance

- [ ] スマホの最初の画面で何のサイトか分かる
- [ ] 受付中かどうか分かる
- [ ] 初見でも参加方法が分かる
```

## Issue 7: Sweet Land 4購入候補をチェックする

### Title

```text
Evaluate Sweet Land 4 purchase candidate
```

### Body

```markdown
## Goal

Sweet Land 4を購入する前に、遠隔操作化しやすく安全に導入できる個体か確認する。

## References

- `SWEETLAND_PURCHASE_CHECKLIST.md`
- `SWEETLAND_REAL_MACHINE_PLAN.md`

## Tasks

- [ ] 動作確認動画を確認する
- [ ] 鍵の有無を確認する
- [ ] 電源仕様を確認する
- [ ] サイズと重さを確認する
- [ ] 搬入経路を確認する
- [ ] 操作ボタンの状態を確認する
- [ ] カメラ設置位置を想定する
- [ ] 出品者へ質問する

## Acceptance

- [ ] 通常プレイできると判断できる
- [ ] 搬入できる
- [ ] 遠隔操作化の初期テストに向いている
- [ ] 赤信号項目がない
```

## Issue 8: サーボ安全テストを行う

### Title

```text
Run external servo safety test before real machine operation
```

### Body

```markdown
## Goal

実機内部配線を触らず、外付けサーボで物理ボタンを安全に押せるか確認する。

## References

- `SWEETLAND_SERVO_SAFETY_CHECKLIST.md`
- `SWEETLAND_HARDWARE_SHOPPING_LIST.md`

## Tasks

- [ ] サーボ単体を机上テストする
- [ ] 緊急停止でサーボ電源を切れるようにする
- [ ] 実機に固定しても干渉しないか確認する
- [ ] 1ボタンだけ短時間押す
- [ ] left/right/scoopへ拡張する
- [ ] カメラ遅延を測る
- [ ] 押しっぱなしにならないことを確認する

## Acceptance

- [ ] サーボ電源を切れば止まる
- [ ] ボタン以外に干渉しない
- [ ] emergencyで操作が止まる
- [ ] 30分の身内テストへ進める
```

## Issue Labels

おすすめラベル:

- `web-app`
- `admin`
- `gateway`
- `hardware`
- `stream`
- `security`
- `test`
- `docs`
- `priority-high`
- `blocked`

