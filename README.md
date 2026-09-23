# 決算前スクリーナー (J-Quants × 四季報)

GitHub Actions で自動実行し、結果を Slack と Actions の実行結果ページに出します。

| ワークフロー | いつ | 何をする |
|---|---|---|
| backtest (過去検証) | 手動 | 「進捗率が高い銘柄は次の決算で上振れし、株価も上がるか」を過去2年で集計 |
| screener (毎朝スクリーニング) | 平日7:30 + 手動 | 四季報強気リスト × 進捗率 × 決算日 × 流動性 で候補を上位15件に絞る |

## 初期設定

1. **Secrets を登録**: Settings → Secrets and variables → Actions → New repository secret
   - `JQUANTS_API_KEY` : J-Quants ダッシュボードで発行した API キー
   - `SLACK_WEBHOOK_URL` : (任意) Slack の Incoming Webhook URL。未設定なら Slack 投稿だけ省略
2. **過去検証を実行**: Actions → backtest (過去検証) → Run workflow
   - Free プラン(5回/分)だと初回は 2〜3 時間。取得データはキャッシュされ、2回目以降は速い
   - 結果: 実行ページの Summary に表、Artifacts に CSV

## 四季報リストの更新 (年4回)

四季報オンラインで「大幅強気」「会社比強気」の銘柄を抽出し、`shikiho/shikiho_list.csv` を GitHub 上で編集:

```
Code,Mark,Issue
7203,大幅強気,2026秋号
6758,会社比強気,2026秋号
```

リストが空なら、全銘柄を進捗率だけで評価します。

## 有料プランに上げたら

Settings → Secrets and variables → Actions → **Variables** タブで追加:

| 名前 | Light の値 | 意味 |
|---|---|---|
| `JQUANTS_RATE_PER_MIN` | 60 | 1分あたりのリクエスト上限 |
| `DATA_DELAY_WEEKS` | 0 | データ遅延(Free は 12) |

Free プランはデータが12週遅れるため、毎朝のスクリーニングは「動作確認用」です。実運用は有料プランで。

## 指標の意味

- **進捗率** = 累計営業利益 ÷ 会社の通期営業利益予想
- **超過pt** = 進捗率 − 前年同期の進捗率(なければ 25/50/75% の単純按分)
- **窓A** = 次の決算の5営業日前に買い → 決算反応日の引けで売り
- **窓B** = 前回決算の翌日に買い → 次の決算反応日の引けで売り
- リターンはすべて全銘柄の中央値との差。手数料・スリッページは含みません

投資判断はご自身の責任で行ってください。
