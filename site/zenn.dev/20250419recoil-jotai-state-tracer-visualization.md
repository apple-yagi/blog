---
https://zenn.dev/apple_yagi/articles/e62341a64d0599
---

# Recoil・jotaiのデータフロー図を可視化するツール state-tracer の紹介

**Author:** やなぎ  
**Date:** 2025/04/19

## 概要

この記事では、RecoilとJotai状態管理ライブラリのデータフロー図を可視化するツール `state-tracer` を紹介します。

## デモ

このツールは簡単なコマンドで使用できます：

```bash
# Recoil
npx @state-tracer/recoil src

# jotai
npx @state-tracer/jotai src
```

これにより、アプリケーション全体の状態構造を理解するのに役立つデータフロー図が生成されます。

## 使用技術

- 言語: TypeScript
- ライブラリ:
  - oxc-parser
  - oxc-walker
  - @hpcc-js/wasm-graphviz

## 動作原理

このツールは以下のステップで動作します：

1. ファイルからすべての状態定義を抽出
2. 状態の相互作用を分析して依存関係配列を構築
3. 依存関係をDOT言語のグラフ構造に変換
4. Graphvizを使用してSVG可視化を生成

## まとめ

大規模なRecoil/Jotaiアプリケーションでは、このツールが興味深い「スパゲティ」状態構造を明らかにできると著者は提案しています。

**GitHubリポジトリ:** <https://github.com/apple-yagi/state-tracer/tree/main>
