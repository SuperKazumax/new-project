# Sweet Land Next Actions

## Purpose

今すぐ進める順番を明確にする。  
別VS Code/Codexが実装中でも、この順番を見れば次に渡す作業が分かるようにする。

## Current Assumption

別VS Code側では、ユーザー管理、手動許可、有料会員シミュレーションを進めている。  
こちらでは、ぶつからないように次の実装候補と検証項目を整理する。

## Priority 1: Finish User Permission Prototype

担当: 実装側Codex

参照:

- `SWEETLAND_PAYPAY_MEMBERSHIP_PLAN.md`
- `SWEETLAND_GITHUB_ISSUES.md` Issue 2

完了条件:

- 管理者がユーザーを手動許可できる。
- ユーザーが自分で `paid` を名乗れない。
- 残り回数を管理できる。
- 許可されていないユーザーは順番待ちできない。

次に渡す文:

```text
SWEETLAND_PAYPAY_MEMBERSHIP_PLAN.md と SWEETLAND_GITHUB_ISSUES.md の Issue 2 を読んで、ユーザー管理・手動許可・有料会員シミュレーションを完成させてください。
```

## Priority 2: Add Stream Phase Flow

担当: 実装側Codex

参照:

- `SWEETLAND_STREAM_FLOW.md`
- `SWEETLAND_PUBLIC_COPY.md`
- `SWEETLAND_GITHUB_ISSUES.md` Issue 1

完了条件:

- `21:00 開始`
- `受付開始`
- `ラスト募集`
- `終了準備`
- `配信終了`

上記を管理ボタンで切り替えられる。

次に渡す文:

```text
SWEETLAND_STREAM_FLOW.md と SWEETLAND_GITHUB_ISSUES.md の Issue 1 を読んで、配信フェーズ状態と管理ボタンを実装してください。
```

## Priority 3: Implement Mock Gateway

担当: 実装側Codexまたは別Codex

参照:

- `SWEETLAND_CONTROL_GATEWAY_SPEC.md`
- `SWEETLAND_MOCK_GATEWAY_IMPLEMENTATION_PLAN.md`
- `SWEETLAND_GITHUB_ISSUES.md` Issue 3

完了条件:

- Mock gatewayが単体起動する。
- safe / armed / emergency modeがある。
- tokenなし操作を拒否する。
- cooldownで連打を拒否する。
- ログが残る。

次に渡す文:

```text
SWEETLAND_MOCK_GATEWAY_IMPLEMENTATION_PLAN.md と SWEETLAND_GITHUB_ISSUES.md の Issue 3 を読んで、standalone Mock gatewayを実装してください。実機やサーボはまだ動かさないでください。
```

## Priority 4: Connect Web Server To Mock Gateway

担当: 実装側Codex

参照:

- `SWEETLAND_GITHUB_ISSUES.md` Issue 4
- `SWEETLAND_CONTROL_GATEWAY_SPEC.md`

完了条件:

- Webサーバーの操作命令がMock gatewayへ送られる。
- gateway拒否時はWeb側でも失敗扱いになる。
- `scoop` はgateway成功後にターン終了する。
- gateway状態が管理画面に見える。

次に渡す文:

```text
SWEETLAND_GITHUB_ISSUES.md の Issue 4 を読んで、Webサーバーの操作命令をMock gatewayへ接続してください。
```

## Priority 5: Run No-Hardware Rehearsal

担当: kazuma / 管理者 / テスト担当Codex

参照:

- `SWEETLAND_REHEARSAL_TEST_SHEET.md`
- `SWEETLAND_LIVE_RUNBOOK.md`

完了条件:

- 2端末以上で順番待ちを試す。
- 配信フェーズを最後まで通す。
- 緊急停止を試す。
- スマホ表示を確認する。
- NG/HOLDを記録する。

次に渡す文:

```text
SWEETLAND_REHEARSAL_TEST_SHEET.md に沿って、実機なしの配信リハーサルを実施し、OK/NG/HOLDを記録してください。
```

## Priority 6: Prepare Real Machine Purchase

担当: kazuma

参照:

- `SWEETLAND_PURCHASE_CHECKLIST.md`
- `SWEETLAND_REAL_MACHINE_PLAN.md`
- `SWEETLAND_HARDWARE_SHOPPING_LIST.md`

完了条件:

- 候補機の動作動画を確認する。
- 鍵、電源、サイズ、搬入を確認する。
- 遠隔操作化しやすい個体か判断する。

次に使う文:

```text
このSweet Land 4候補を買って大丈夫か、SWEETLAND_PURCHASE_CHECKLIST.md に沿って確認してください。
```

## Do First Today

今日やるならこの順番。

1. PayPay/会員シミュレーションのUI整理を実装側Codexへ渡す。
2. `/api/user` 送信から `memberStatus` を外す。
3. 一般ユーザーの `memberStatus` セレクトを読み取り専用表示へ変える。
4. 管理パネルをadminToken入力後だけ表示する。
5. その後、Issue 1の配信フェーズ状態またはIssue 3のMock gatewayを進める。

## Do Not Do Yet

- 実機内部配線を触る。
- PayPay APIの本接続をする。
- gatewayをインターネットに公開する。
- 認証なしで実機操作を公開する。
- 無人運用する。
