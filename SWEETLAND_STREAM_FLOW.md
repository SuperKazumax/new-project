# Sweet Land Stream Operation Flow

## Goal

配信中の状態を、管理者がボタン一発で切り替えられるようにする。

想定する基本の流れ:

```text
21:00 開始
↓
受付開始
↓
ラスト募集
↓
終了準備
↓
配信終了
```

## Stream States

## 1. Before Start

配信前の待機状態。

### Viewer Message

- まもなく配信開始です。
- 受付開始までお待ちください。

### User Actions

- 順番待ち: できない
- 操作: できない
- 画面表示: 配信待機、ルール確認

### Admin Actions

- `21:00 開始` ボタンで `Live Started` に進む。
- 配信前チェックリストを確認する。

## 2. Live Started

配信は始まっているが、まだ参加受付は開いていない状態。

### Viewer Message

- 配信開始しました。
- 受付開始まで少しお待ちください。

### User Actions

- 順番待ち: できない
- 操作: できない
- 画面表示: YouTube配信、遊び方、参加条件

### Admin Actions

- `受付開始` ボタンで `Accepting Entries` に進む。
- 緊急停止はいつでも押せる。

## 3. Accepting Entries

通常受付中。視聴者が順番待ちに入れる状態。

### Viewer Message

- 受付中です。
- レーンを選んで順番待ちに参加できます。

### User Actions

- 順番待ち: できる
- 操作: 自分の番だけできる
- レーン選択: できる

### Admin Actions

- `次の人を開始`
- `ターン終了`
- `1回付与`
- `ラスト募集` ボタンで `Last Call` に進む。
- 緊急停止

## 4. Last Call

新規参加の締め切り前。最後の参加者を募集する状態。

### Viewer Message

- ラスト募集です。
- まもなく新規受付を終了します。

### User Actions

- 順番待ち: できる
- 操作: 自分の番だけできる
- 画面表示: 締め切りが近いことを強調

### Admin Actions

- `終了準備` ボタンで `Closing` に進む。
- 必要なら `受付中に戻す` で `Accepting Entries` に戻せる。
- 緊急停止

## 5. Closing

新規受付を止め、残っている順番だけ消化する状態。

### Viewer Message

- 新規受付は終了しました。
- 現在並んでいる方のプレイを順番に進めます。

### User Actions

- 新規順番待ち: できない
- 既に並んでいる人の操作: 自分の番だけできる
- 順番待ちから抜ける: できる

### Admin Actions

- `次の人を開始`
- `ターン終了`
- `配信終了` ボタンで `Ended` に進む。
- 必要なら `受付再開` で `Accepting Entries` に戻せる。
- 緊急停止

## 6. Ended

配信終了状態。

### Viewer Message

- 本日の配信は終了しました。
- 次回配信をお待ちください。

### User Actions

- 順番待ち: できない
- 操作: できない
- 画面表示: 終了案内、次回案内、SNS/YouTubeリンク

### Admin Actions

- ログ確認
- 結果メモ
- 次回準備
- 必要なら `配信前に戻す`

## Admin Flow Buttons

管理画面には、状態ごとに主ボタンを1つだけ目立たせる。

| Current State | Primary Button | Next State |
| --- | --- | --- |
| Before Start | 21:00 開始 | Live Started |
| Live Started | 受付開始 | Accepting Entries |
| Accepting Entries | ラスト募集 | Last Call |
| Last Call | 終了準備 | Closing |
| Closing | 配信終了 | Ended |
| Ended | 配信前に戻す | Before Start |

常時表示する管理ボタン:

- 緊急停止
- ターン終了
- 次の人を開始

ただし `Before Start`、`Live Started`、`Ended` では、ユーザー操作系ボタンは無効でもよい。

## Server State Model

サーバー状態に `streamPhase` を追加する。

```json
{
  "streamPhase": "before_start"
}
```

候補値:

- `before_start`
- `live_started`
- `accepting_entries`
- `last_call`
- `closing`
- `ended`

## Rules

- `before_start`: 新規参加不可、操作不可。
- `live_started`: 新規参加不可、操作不可。
- `accepting_entries`: 新規参加可、操作可。
- `last_call`: 新規参加可、操作可。
- `closing`: 新規参加不可、既存キューと現在プレイヤーは操作可。
- `ended`: 新規参加不可、操作不可。
- `emergencyStopped` が true の場合は、どの状態でも操作不可。

## API Proposal

### State Change

`POST /api/stream-phase`

Request:

```json
{
  "phase": "accepting_entries",
  "adminToken": "..."
}
```

Response:

```json
{
  "ok": true,
  "state": {}
}
```

### Validation

- 管理者だけが変更できる。
- 未知の `phase` は `400 unknown_phase`。
- 状態変更はログに残す。

## UI Requirements

### Viewer Area

- 現在の配信状態を大きく表示する。
- `accepting_entries` と `last_call` のときだけ参加導線を強調する。
- `closing` では「新規受付終了」を明確に出す。
- `ended` では次回案内に切り替える。

### Admin Area

- 現在状態を表示する。
- 次に押すべき主ボタンを1つだけ強調する。
- 戻し操作は小さめの副ボタンにする。
- 緊急停止は常に赤で表示する。

## Test Cases

- `before_start` では順番待ちに入れない。
- `live_started` では順番待ちに入れない。
- `accepting_entries` では順番待ちに入れる。
- `last_call` では順番待ちに入れる。
- `closing` では新規順番待ちに入れない。
- `closing` でも既に順番待ち済みの人はプレイできる。
- `ended` では順番待ちも操作もできない。
- 緊急停止中は、どの状態でも操作できない。
- 管理者以外は `streamPhase` を変更できない。
- 状態変更ログに時刻と変更先が残る。

