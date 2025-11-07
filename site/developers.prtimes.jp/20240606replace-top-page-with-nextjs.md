---
https://developers.prtimes.jp/2024/06/06/replace-top-page-with-nextjs/
---

# PR TIMESのトップページをNext.jsにリプレイスしました

**Author:** 柳 龍哉 ([@apple_yagi](https://twitter.com/apple_yagi))
**Date:** 2024年6月6日

こんにちは、フロントエンドエンジニアのやなぎです。プレスリリース掲載ページ、キーワード検索ページに続き、PR TIMESの[トップページ](https://prtimes.jp/)を PHP + Smarty + jQuery から Next.js（Pages Router）にリプレイスしました。

## SP用ページで使用するカルーセルライブラリの選定

トップページの SP 版では横スワイプでタブを切り替えます。このスワイプ操作を実装するため、次のカルーセルライブラリを比較検討しました。

- Swiper
- Nuka Carousel
- React Slick
- Embla Carousel

最終的に Embla Carousel を選択した理由は以下の通りです。

- スクロール位置のリセット問題を解決
- スワイプの閾値が適切
- アクセシビリティの問題がない
- shadcn/uiやyamada-uiなどで使用実績がある

## BFCacheを有効化

トップページで BFCache を有効にし、次のメリットを得ました。

- ブラウザバック時に元のタブ位置を保持
- 新着タブの無限スクロールデータを保持

## キャッシュの設定

CDN で Server Side Rendering の HTML をキャッシュする戦略を採用しました。

- Surrogate-Control ヘッダーを使用
- キャッシュ設定は `max-age=5, stale-while-revalidate=10`
- Fastlyで約90%のキャッシュヒット率を実現
- 実装後はキャッシュヒット時に Fastly から直接 HTML を返せるようになり、レスポンス時間のばらつきが小さくなった

## まとめ

> 「行動者発の情報が、人の心を揺さぶる時代へ」

PR TIMESのトップページリプレイスにより、モダンなフロントエンド技術スタックへの移行を完了しました。

> 「今後はアプリケーション全体に目を向け、実現したいUX（ユーザーエクスペリエンス）を意識しながら、アーキテクトに取り組んでいきたいと思っています。」

このマイグレーションは、PR TIMESのフロントエンドインフラストラクチャを現代化する重要な一歩を表しています。
