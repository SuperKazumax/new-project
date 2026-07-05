# Sweet Land Project Roadmap

## Purpose

Sweet Land YouTube配信プロジェクトを、実装、配信運用、実機連携の順で迷わず進めるためのロードマップ。

## North Star

視聴者がYouTube配信を見ながら順番待ちに参加し、自分の番だけ安全にSweet Land 4を操作できる状態を作る。

## Current Workstreams

| Workstream | Goal | Main Docs |
| --- | --- | --- |
| Web app | 順番待ち、操作、管理画面を作る | `SWEETLAND_TASKS.md` |
| Stream flow | 21:00開始から配信終了までを状態管理する | `SWEETLAND_STREAM_FLOW.md` |
| User management | 手動許可、有料会員シミュレーションを作る | `SWEETLAND_PAYPAY_MEMBERSHIP_PLAN.md` |
| Gateway | 実機操作前の安全係を作る | `SWEETLAND_CONTROL_GATEWAY_SPEC.md` |
| Hardware | 実機購入、カメラ、外付けサーボを準備する | `SWEETLAND_REAL_MACHINE_PLAN.md` |
| Operations | 当日台本、文言、リハーサルを整える | `SWEETLAND_LIVE_RUNBOOK.md` |

## Milestone 1: Web Prototype

目的: 実機なしで、配信アプリとしての基本動作を確認する。

必須:

- [ ] 受付開始、ラスト募集、終了準備、配信終了を切り替えられる。
- [ ] ユーザーが順番待ちに入れる。
- [ ] 管理者が次の人を開始できる。
- [ ] 操作権のあるユーザーだけ操作できる。
- [ ] 緊急停止中は操作できない。
- [ ] スマホで操作しやすい。

完了条件:

- 2台以上の端末で順番待ちテストが通る。
- 配信中フローを最後まで通せる。

## Milestone 2: User Permission Prototype

目的: 誰でも遊べる状態ではなく、管理者が参加可否を制御できるようにする。

必須:

- [ ] ユーザー一覧を見られる。
- [ ] 管理者が手動で参加許可できる。
- [ ] guest / allowed / paid などの状態を扱える。
- [ ] ユーザーが自分で `paid` を名乗れない。
- [ ] 残り回数を管理できる。

完了条件:

- 一般ユーザーは勝手に有料会員扱いになれない。
- 管理者だけが許可と回数調整をできる。

## Milestone 3: Mock Gateway

目的: 実機を動かさず、Webアプリから制御ゲートウェイへ命令を送る流れを確認する。

必須:

- [ ] Mock gatewayが起動する。
- [ ] `/health` と `/state` が返る。
- [ ] `left`、`right`、`scoop` を受け取れる。
- [ ] safe / armed / emergency modeがある。
- [ ] トークンなし操作を拒否する。
- [ ] 連打を拒否する。
- [ ] ログが残る。

完了条件:

- 実機なしで操作命令の成功/拒否を確認できる。
- Webアプリ側がgateway拒否をユーザーへ表示できる。

## Milestone 4: Stream Rehearsal

目的: 実機なしで、配信当日の人間オペレーションを練習する。

必須:

- [ ] YouTube限定公開でテスト配信する。
- [ ] 21:00開始から配信終了まで通す。
- [ ] 固定コメントと概要欄を使う。
- [ ] 2人以上で順番待ちを試す。
- [ ] 緊急停止の文言を読む。
- [ ] リハーサル結果を記録する。

完了条件:

- `SWEETLAND_REHEARSAL_TEST_SHEET.md` がOK中心で埋まる。
- HOLD/NG項目が次回TODOになっている。

## Milestone 5: Real Machine Purchase

目的: 遠隔操作化しやすいSweet Land 4を安全に入手する。

必須:

- [ ] 動作確認動画がある。
- [ ] 鍵がある。
- [ ] 電源仕様、サイズ、重さが分かる。
- [ ] 搬入できる。
- [ ] 通常プレイできる。
- [ ] ボタンが正常。

完了条件:

- 実機が無改造のまま通常プレイできる。
- カメラ設置位置の候補が決まる。

## Milestone 6: Camera And No-Mod Test

目的: 実機を改造せず、配信映像と手動プレイを成立させる。

必須:

- [ ] 盤面カメラを設置する。
- [ ] 景品落とし口が見える。
- [ ] シャベル位置が見える。
- [ ] 映像遅延を測る。
- [ ] 手動プレイを配信画面で確認する。

完了条件:

- 視聴者目線で操作判断できる映像になっている。
- 遅延込みで遊べるか判断できる。

## Milestone 7: External Servo Test

目的: 実機内部配線を触らず、外付けサーボで物理ボタンを押す。

必須:

- [ ] サーボ机上テストが通る。
- [ ] 緊急停止でサーボ電源を切れる。
- [ ] サーボがボタン以外に干渉しない。
- [ ] 1ボタンだけ短時間テストする。
- [ ] left / right / scoop へ拡張する。

完了条件:

- `SWEETLAND_SERVO_SAFETY_CHECKLIST.md` の主要項目がOK。
- 押しっぱなしや暴走がない。

## Milestone 8: Closed Beta Stream

目的: 身内だけで実機遠隔操作を通しテストする。

必須:

- [ ] 限定公開で配信する。
- [ ] 管理者が常駐する。
- [ ] 実機近くに確認者がいる。
- [ ] 緊急停止を実際に試す。
- [ ] 30分以上通す。
- [ ] ログを確認する。

完了条件:

- 大きな事故なく通せる。
- 未解決リスクが明確になっている。

## Milestone 9: Public Test Stream

目的: 少人数向けに公開テストする。

必須:

- [ ] 参加条件を明記する。
- [ ] 回数上限を設定する。
- [ ] 荒らし対応を準備する。
- [ ] トラブル時の文言を準備する。
- [ ] 終了準備と配信終了が機能する。

完了条件:

- 管理者が無理なく運用できる。
- 視聴者が迷わず参加できる。

## Immediate Next Actions

優先順。

1. 別VS Code側のユーザー管理作業が完了したら差分確認。
2. Mock gatewayを実装する。
3. WebアプリからMock gatewayへ操作命令を送る。
4. リハーサルシートで実機なし通しテスト。
5. Sweet Land 4購入候補をチェックリストで確認。

## Do Not Mix Yet

今はまだ混ぜない。

- 実機内部配線
- 本決済
- 無人運用
- 外部公開されたgateway
- 認証なしの実機操作

