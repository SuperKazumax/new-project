# Sweet Land Handoff Prompt

別のCodex、iPhoneのChatGPT、または別スレッドで続きを頼むときは、この内容を貼る。

## Short Prompt

Sweet LandのYouTube配信/遠隔操作サイトを開発中です。  
管理用リポジトリは `https://github.com/SuperKazumax/new-project` です。  
まず `SWEETLAND_RELEASE_CHECKLIST.md` と `SWEETLAND_TASKS.md` を読んで、上から順番に進めてください。  
実装本体は `C:\Users\kazuma\Desktop\AIの作業場\sweetland-server` 側です。  
最優先は、操作APIをGETで実行できないようにして、状態変更APIをPOST中心にすることです。  
その次に管理者認証、会員状態のサーバー判定、視聴者向けトップページ、配信前テストの順で進めてください。

## Longer Prompt

あなたはSweet Land遠隔操作/YouTube配信用Webサイトの開発支援担当です。

目的:

- YouTube配信を見ながら、視聴者が順番待ちしてスイートランドを操作できるようにする。
- 管理者が開始、終了、緊急停止、回数調整を安全にできるようにする。
- 公開前にセキュリティ、荒らし対策、スマホ操作性を確認する。

参照する管理資料:

- `https://github.com/SuperKazumax/new-project`
- `SWEETLAND_RELEASE_CHECKLIST.md`
- `SWEETLAND_TASKS.md`

実装本体:

- `C:\Users\kazuma\Desktop\AIの作業場\sweetland-server`

優先順:

1. 状態変更APIをGETからPOST中心にする。
2. 管理APIに管理者認証を入れる。
3. 会員状態をユーザー入力ではなくサーバー側で判定する。
4. 管理画面を一般ユーザーから隠す。
5. YouTube配信用の視聴者向けトップページを追加する。
6. 配信前テストを実施して結果を残す。

注意:

- 他のCodexが同時に作業している可能性があるため、同じファイルを触る前に現在の差分を確認する。
- `main` に直接入れず、できればブランチとPull Requestで進める。
- 本番公開前は、認証なしの操作APIを残さない。

