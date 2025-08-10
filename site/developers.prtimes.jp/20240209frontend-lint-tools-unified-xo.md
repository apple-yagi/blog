---
https://developers.prtimes.jp/2024/02/09/frontend-lint-tools-unified-xo/
---

# フロントエンドのLintツールをXOに統一した話

**Author:** 柳 龍哉 (Ryuya Yanagi)  
**Date:** 2024年2月9日

## はじめに

PR TIMESでは、フロントエンドのLintツールをESLintからXOに移行しました。この記事では移行の背景、実装方法、そして結果について説明します。

## 移行の背景

移行を決めた主な理由は以下の3点でした：

1. 既存のESLint設定が「継ぎはぎ」状態で、明確な理由付けが欠如していた
2. 開発者が自由にlintルールを変更しており、重要なチェックが回避される可能性があった
3. アドバイザーの1000ch氏からXOの使用を推奨された

## XOとは？

XOは「opinionated but configurable ESLint wrapper」で、以下の特徴があります：

- 厳格で読みやすいコードを強制
- Prettierを内包
- ESLint/Plugin のバージョン管理が不要

## 実装アプローチ

1. XOを初回インストールした時点で7,817個のエラーが発生
2. 問題のあるルールを一時的に全て無効化（181ルールをoff）
3. 段階的にPull Requestを通してルールを適用

## 遭遇した課題

### 対応困難なESLintプラグイン

1. `@typescript-eslint/strict-boolean-expressions`
2. `@typescript-eslint/prefer-nullish-coalescing`
3. `@typescript-eslint/no-floating-promises`

### 無効化したプラグイン

1. `capitalized-comments`
2. `import/extensions`
3. `n/file-extension-in-import`

## 結果

### 良い効果

- コードスタイルの統一
- 依存関係管理のオーバーヘッド削減
- コード品質の向上
- モダンなJavaScriptの書き方を学習

### 課題

- 実行速度の低下（約27万行のコードで2分間）

## まとめ

XOの導入により、PR TIMESのフロントエンド開発において、lintの標準化、メンテナンス複雑性の削減、そして全体的なコード品質の向上を実現できました。実行速度の問題はありますが、得られるメリットがそれを上回ると判断しています。
