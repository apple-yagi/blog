---
https://developers.prtimes.jp/2022/04/14/web-front-end-development-enviroment-in-prtimes-inc/
---

# PR TIMESにおけるフロントエンド開発基盤の構築

**Author:** 柳 龍哉
**Date:** 2022年4月14日

## 概要

この記事では、PR TIMESにおけるフロントエンド開発基盤の変革について説明します。特に次のポイントに焦点を当てています。

- Reactへの移行
- フロントエンドコードの単一モノレポへの統合
- デプロイメントワークフローの改善

## 以前の構成での主な課題

1. フロントエンドの実装が複数のリポジトリに分散していた
2. 開発者がバンドルファイルを手動でビルドしてGitHubにコミットする必要があった
3. プロジェクト間でのバージョン競合やコードの乖離の可能性

## 新しいインフラストラクチャアプローチ

### リポジトリ構造

```
.
├── .githooks
├── .github
├── apps
│   └── prtimes
│       ├── package.json
│       ├── src/
│       └── webpack configs
├── Makefile
└── yarn.lock
```

### デプロイメントフローの変更

- オンプレミスサーバーからS3への移行
- ビルドとデプロイメントのためのGitHub Actionsの実装
- コンテンツ配信をCloudFrontからFastlyに切り替え

## 今後の改善予定

- より多くのプロジェクトのReactへの移行
- yarn workspacesの実装
- webpackからesbuildへの置き換え
- Storybookの導入

筆者は、これが継続的な改善が期待される進行中のプロセスであることを強調しています。
