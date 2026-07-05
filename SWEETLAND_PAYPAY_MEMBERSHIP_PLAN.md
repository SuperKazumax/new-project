# Sweet Land PayPay And Membership Plan

## Purpose

Sweet Land配信で、PayPay支払いを使って参加権を付与するための設計。

最初は自動連携ではなく、管理者がPayPay入金を確認して手動許可する。  
安全に運用できるようになってから、PayPay API連携や自動付与を検討する。

## First Release Policy

最初の公開テストでは、決済とアプリ権限を完全自動連携しない。

理由:

- 決済ミス時の返金や問い合わせ対応が必要。
- ユーザー名と支払い者を紐づける運用が必要。
- 景品機の有料遠隔操作はトラブル時の説明責任が重い。
- 不正利用やなりすましを防ぐ必要がある。

初期運用:

```text
PayPayで支払い
↓
管理者が入金確認
↓
管理画面でユーザーに権限付与
↓
ユーザーが当日プレイ
```

## Plan Types

## 1. Guest

無料視聴者。

- YouTube視聴: OK
- 順番待ち: NG
- 操作: NG
- 管理者のお試し許可: 任意

## 2. Trial Play

お試し参加。

- 例: 1回だけ
- 管理者が手動付与
- 配信テストやキャンペーン向け

## 3. Play Ticket

金額に応じて回数を付与する。

例:

| Plan | Example Price | Permission |
| --- | --- | --- |
| 1回券 | 100円 | 1 play |
| 3回券 | 300円 | 3 plays |
| 5回券 | 500円 | 5 plays |

注意:

- 金額は仮。実際の景品原価、送料、手数料、運用負担を見て決める。
- 返金ルールを必ず決める。

## 4. Day Pass

その日だけ遊べるプラン。

例:

| Plan | Example Price | Permission |
| --- | --- | --- |
| 1日ライト | 500円 | 当日5回まで |
| 1日たっぷり | 1000円 | 当日20回まで |
| 1日やり放題風 | 1500円 | 当日上限ありの多回数 |

重要:

「無制限」と書くと、混雑・景品切れ・トラブル時に危ない。  
実際には `fair use limit` を設ける。

表現例:

```text
1日たっぷりプラン
本日の配信中、上限回数まで何度でも参加できます。
混雑時は順番制です。
```

避けたい表現:

```text
完全無制限
絶対取り放題
必ず景品が取れる
```

## 5. Monthly Member

月額会員。

- 毎配信で一定回数付与
- 優先参加
- 会員限定日
- 名前表示やバッジ

初期段階では後回しでよい。

## Recommended Initial Plans

最初はこの3つで十分。

| Plan | Use |
| --- | --- |
| Guest | 視聴のみ |
| 3回券 | 少額テスト |
| 1日パス | 配信日限定のメイン商品 |

## User Permission Model

ユーザーごとに権限を持つ。

```json
{
  "userId": "user-123",
  "displayName": "kazuma",
  "permission": {
    "type": "day_pass",
    "validDate": "2026-07-05",
    "dailyLimit": 20,
    "dailyUsed": 3,
    "expiresAt": "2026-07-05T23:59:59+09:00"
  }
}
```

## Permission Types

- `guest`
- `trial`
- `ticket`
- `day_pass`
- `monthly_member`
- `admin_grant`
- `blocked`

## Admin Operations

管理画面に必要な操作。

- ユーザー検索
- ユーザーを手動許可
- 回数券を付与
- 1日パスを付与
- 回数を追加
- 回数を減らす
- 期限を変更
- ユーザーを停止
- 支払いメモを残す

## Manual PayPay Flow

## Step 1: User Pays

ユーザーに案内する。

```text
PayPayで支払い後、表示名と支払い時刻をコメントまたはフォームで送ってください。
管理者が確認後、当日の参加権を付与します。
```

## Step 2: Admin Confirms

管理者が確認する。

- PayPay for Business等で入金確認
- 金額確認
- 支払い時刻確認
- 表示名確認

## Step 3: Admin Grants Permission

アプリ管理画面で付与。

```text
displayName: kazuma
plan: day_pass
dailyLimit: 20
validDate: today
memo: PayPay 1000 yen 21:05
```

## Step 4: User Plays

ユーザーは順番待ちに参加できる。

## Payment Record

最低限これを記録する。

```json
{
  "paymentId": "manual-20260705-001",
  "provider": "paypay_manual",
  "displayName": "kazuma",
  "amount": 1000,
  "plan": "day_pass",
  "paidAt": "2026-07-05T21:05:00+09:00",
  "confirmedBy": "admin",
  "memo": "PayPay app confirmed"
}
```

## Refund Policy Draft

公開前に明記する。

例:

```text
通信障害、実機トラブル、運営都合でプレイできなかった場合は、状況に応じて回数の再付与または返金対応を行います。
ユーザー都合の途中退出、順番待ち放棄、操作ミスによる返金は原則できません。
```

## Public Notice Draft

```text
有料参加はPayPayで受け付けます。
支払い後、管理者が確認して参加権を付与します。
反映まで少し時間がかかる場合があります。
混雑時は順番制です。
トラブル時は回数再付与または返金で対応します。
```

## Future API Integration

手動運用が安定してから検討する。

自動化したい流れ:

```text
ユーザーがプラン選択
↓
PayPay決済画面へ遷移
↓
決済完了
↓
アプリが決済結果を確認
↓
ユーザーに権限を自動付与
```

必要になるもの:

- PayPay加盟店契約
- PayPay開発者/API利用設定
- 決済完了通知または結果確認
- ユーザーIDと決済IDの紐づけ
- 返金処理の運用
- 不正決済・二重付与対策

## Security Rules

- 支払い完了はユーザー自己申告だけで確定しない。
- 管理者確認または決済API確認を必須にする。
- 同じ支払いIDを二重に使えないようにする。
- 管理者だけが権限付与できる。
- 支払いメモに個人情報を入れすぎない。
- トークンやAPIキーをブラウザへ出さない。

## Legal And Operations Notes

確認が必要なもの:

- PayPay加盟店規約
- 景品や送料の扱い
- 返金ルール
- 未成年利用の扱い
- 特定商取引法の表示が必要か
- 利用規約
- プライバシーポリシー
- YouTube側の規約

## Implementation Tasks

1. 手動許可の管理画面を完成させる。
2. `ticket` と `day_pass` の権限タイプを追加する。
3. 回数上限と期限をサーバー側で判定する。
4. 支払いメモを残せるようにする。
5. 公開ページに料金と注意事項を表示する。
6. 返金/再付与ルールを文言化する。
7. 手動PayPay運用でリハーサルする。
8. 安定後にPayPay API自動連携を検討する。

## Initial Acceptance Tests

- 管理者が3回券を付与できる。
- 3回使うと操作できなくなる。
- 管理者が1日パスを付与できる。
- 1日パスは当日だけ有効。
- 期限切れユーザーは操作できない。
- ブロックされたユーザーは操作できない。
- 支払いメモが残る。
- 一般ユーザーは自分で権限を変更できない。

