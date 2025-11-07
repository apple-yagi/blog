---
https://developers.prtimes.jp/2023/08/10/reduced-billable-time-for-github-actions/
---

# 並列で実行していたGitHub ActionsのJobをまとめ、Billable timeを削減した話

**Author:** 柳 龍哉  
**Date:** 2023年8月10日

## 経緯

PR TIMESでは、GitHub Enterpriseプランを契約しており、月50,000分のGitHub Actionsを使用できます。先月（2023/07）は使用時間の上限に達し、一時的にGitHub Actionsが使用できない状況が発生しました。

この問題を受けて、Billable timeの削減に取り組むことになりました。

## 問題の分析

### Billable timeの消費が激しいWorkflow

元の Workflow では各 Job の Billable time が個別に計算されており、例えば次のような差が生じていました。

- **Total duration** は 5 分 20 秒
- **Billable time** は 21 分

この大きな差の原因は、GitHub Actionsが分単位で課金するため、各Jobの実行時間が切り上げられることにありました。

### 具体的な問題点

1. **分単位の切り上げ**では 30 秒の Job も 1 分として課金される。
2. **セットアップのオーバーヘッド**として各 Job で node_modules の復元などが重複実行されていた。
3. **並列実行による無駄**で短時間の Job を並列実行すると切り上げによるロスが増大する。

## 改善策の実装

### Jobの統合

Job をある程度まとめて直列で実行するように変更しました。

**Before**

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]  # 実行時間: 45秒 → Billable: 1分
  
  test-unit:
    runs-on: ubuntu-latest
    steps: [...]  # 実行時間: 1分30秒 → Billable: 2分
  
  test-integration:
    runs-on: ubuntu-latest
    steps: [...]  # 実行時間: 2分15秒 → Billable: 3分
```

**After**

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - # setup steps
      - # lint
      - # unit test
      - # integration test
    # 実行時間: 5分11秒 → Billable: 6分
```

## 結果

Job をまとめて直列で実行するように変更した結果は次の通りです。

- **Total duration** は 5 分 11 秒（若干短縮）
- **Billable time** は 14 分（7 分削減、33% 改善）

## 主な発見

この取り組みを通じて以下のことが分かりました。

1. **分単位切り上げの影響**では、Job を細かく分割すると分単位の切り上げによって余分なコストが発生する。

2. **セットアップのオーバーヘッド**では、パッケージのインストールなどを含む Job を分割すると各 Job に大きなオーバーヘッドが発生する。

3. **キャッシュの効果**として、Setup ステップ（node_modules の restore）が実際の処理時間より長引くケースがある。

## 教訓とベストプラクティス

### Workflowを分割する際の判断基準

1. **実行時間の短縮効果があるか**を確認し、並列化で Total duration が実際に短くなるかを評価する。
2. **セットアップコストとのバランス**では各 Job のセットアップ時間と実処理時間の比率を見る。
3. **分単位切り上げの影響**を意識し、短時間の Job は統合した方がコスト効率が良い場合が多い。

### 推奨アプローチ

- 関連性の高いタスク（lint、unit test、integration testなど）は同一Jobで実行
- 本当に時間短縮効果がある場合のみ並列化を検討
- Billable timeとTotal durationの両方をモニタリング

## まとめ

GitHub ActionsのBillable time削減には、並列実行の見直しが効果的でした。Workflowを分割する際は、Total durationの短縮に有効かどうかを慎重に検討し、各Jobのオーバーヘッドと実際の処理時間のバランスを考慮することが重要です。

この改善により 1 回の Workflow で Billable time を 21 分から 14 分まで削減し、月 50,000 分の上限に余裕を持てるようになりました。
