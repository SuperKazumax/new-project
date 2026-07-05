# Sweet Land Implementation Tasks

この文書は、実装担当Codexに渡すための作業分解です。上から順番に進める。

## Task 1: 操作APIをPOST中心にする

### 目的

URLを開くだけで「並ぶ」「開始」「操作」「緊急停止」が実行されないようにする。

### 対象

- `sweetland-server/server.ps1`
- `sweetland-server/public/app.js`

### やること

- `/api/state` だけGETを許可する。
- 状態を変えるAPIはPOSTのみ許可する。
- フロント側の `fetch` はPOSTでJSON bodyを送る。
- GETで状態変更APIを叩いた場合は `405 method_not_allowed` を返す。

### POST化するAPI

- `/api/user`
- `/api/join`
- `/api/leave`
- `/api/start-next`
- `/api/end-turn`
- `/api/command`
- `/api/grant`
- `/api/config`
- `/api/emergency`
- `/api/reset`

### 受け入れ条件

- ブラウザ操作で今まで通り並べる。
- ブラウザ操作で今まで通り左右操作とすくう操作ができる。
- `GET /api/join` は拒否される。
- `GET /api/emergency` は拒否される。
- `GET /api/state` は今まで通り状態を返す。

### 注意

これはセキュリティの入口であり、認証ではない。次のTask 2で管理者認証を追加する。

## Task 2: 管理者認証を追加する

### 目的

一般ユーザーが管理APIを実行できないようにする。

### 対象

- `sweetland-server/server.ps1`
- `sweetland-server/public/app.js`
- 必要なら管理者用UI

### 管理API

- `/api/start-next`
- `/api/end-turn`
- `/api/grant`
- `/api/config`
- `/api/emergency`
- `/api/reset`

### やること

- 管理者用の簡易トークンをサーバー起動時に設定できるようにする。
- 管理APIでは管理者トークンを確認する。
- トークンがない、または違う場合は `403 admin_required` を返す。
- 一般ユーザー画面では管理ボタンを表示しない。

### 受け入れ条件

- 一般ユーザーは管理APIを実行できない。
- 管理者トークンありの場合だけ管理APIを実行できる。
- 管理者ボタンが一般ユーザーに見えない。
- 緊急停止は管理者だけが実行できる。

## Task 3: 会員状態をサーバー側で判定する

### 目的

ユーザーが自分で `paid` を名乗るだけでプレイできる状態を防ぐ。

### 対象

- `sweetland-server/server.ps1`
- `sweetland-server/public/app.js`

### やること

- フロントから送られた `memberStatus` を信用しない。
- サーバー側でユーザーIDごとの状態を管理する。
- 試作段階では管理者だけがユーザーを `paid` にできる形でよい。
- 将来はYouTubeメンバー、ログイン、決済情報などに接続する前提にする。

### 受け入れ条件

- 一般ユーザーが自分で `paid` を送っても権限が変わらない。
- 管理者操作で会員状態を変えられる。
- `guest` は順番待ちまたは操作を拒否される。

## Task 4: 視聴者向けトップページを追加する

### 目的

YouTube配信を見に来た人が、見方・遊び方・参加条件をすぐ理解できるようにする。

### 対象

- `sweetland-server/public/index.html`
- `sweetland-server/public/styles.css`
- 必要なら `sweetland-server/public/app.js`

### 入れる内容

- YouTube配信エリア
- 「遊ぶ」または「順番待ちに参加」への導線
- 遊び方3ステップ
- 参加条件
- 回数制限
- 注意事項
- 問い合わせ/SNSリンク

### 受け入れ条件

- スマホの最初の画面で、何のサイトか分かる。
- YouTube配信への導線が分かる。
- 初めて来た人でも参加方法が分かる。
- 操作画面と管理画面が混ざって見えない。

## Task 5: 配信前テストを通す

### 目的

公開前に、最低限の事故を減らす。

### テスト

- 1人が並んで開始、操作、終了できる。
- 2人が同じレーンに並ぶと順番が守られる。
- 2人が別レーンに並んでも状態が混ざらない。
- 残り回数0で操作できない。
- 緊急停止中は操作できない。
- 管理者以外は管理APIを実行できない。
- スマホ幅でボタンが押しやすい。

### 受け入れ条件

- テスト結果を記録する。
- 未解決の問題を公開前TODOに残す。

## Task 6: 配信中の流れをアプリ化する

### 目的

21:00 開始、受付開始、ラスト募集、終了準備、配信終了を管理者がボタン一発で切り替えられるようにする。

### 参照

- `SWEETLAND_STREAM_FLOW.md`

### 対象

- `sweetland-server/server.ps1`
- `sweetland-server/public/index.html`
- `sweetland-server/public/styles.css`
- `sweetland-server/public/app.js`

### やること

- サーバー状態に `streamPhase` を追加する。
- 管理者だけが `streamPhase` を変更できるようにする。
- 現在状態に応じて、参加可否と操作可否を変える。
- 管理画面に次の主ボタンを表示する。
- 視聴者画面に現在の配信状態メッセージを出す。
- 状態変更をログに残す。

### 受け入れ条件

- `21:00 開始` で配信開始状態になる。
- `受付開始` で新規順番待ちが可能になる。
- `ラスト募集` で締め切り前メッセージが出る。
- `終了準備` で新規順番待ちが止まる。
- `配信終了` で操作と順番待ちが止まる。
- 緊急停止中は、どの配信状態でも操作できない。
