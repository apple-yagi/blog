---
https://developers.prtimes.jp/2022/06/01/build-time-improvements-using-esbuild/
---

# esbuildに乗り換えたらビルド時間が劇的に改善された話

**Author:** 柳 龍哉
**Date:** 2022年6月1日

## はじめに

PR TIMESでは、React/TypeScriptのコードをWebpackでビルドしていましたが、esbuildに乗り換えることでビルド時間が劇的に改善されました。この記事では、その取り組みについて紹介します。

## esbuildとは

esbuildは、Go言語で実装されたJavaScript/TypeScriptのビルドツールです。主な特徴を以下にまとめます。

- キャッシュなしでの高速ビルド
- ES6およびCommonJSのサポート
- Tree shaking
- TypeScriptおよびJSXのサポート
- ソースマップの生成
- コードの最小化

## 実装

チームでは、次の点に特に注意してesbuildを設定しました。

- React 17のサポート
- Emotion CSS-in-JSライブラリの対応
- カスタムビルド設定

## 結果

ビルド時間の比較結果は次の通りです。

- Webpack + ts-loader（従来）は 39.85 秒。
- Webpack + esbuild-loader では 28.09 秒。
- Pure esbuild にすると 2.61 秒。

### 課題

ビルド時間は 39.85 秒から 2.61 秒まで短縮できましたが、次の点に注意が必要でした。

- バンドルファイルサイズの増加
- 互換性の問題の可能性

## まとめ

チームでは開発環境でesbuildを採用し、開発者体験の向上を実感しています。開発者からは「ビルド待ち時間がほぼゼロ」との声があり、開発に集中できるようになりました。

今後もesbuildの開発を注視し、将来的な完全移行を検討していく予定です。
