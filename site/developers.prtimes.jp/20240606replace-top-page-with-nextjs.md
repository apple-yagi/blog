---
https://developers.prtimes.jp/2024/06/06/replace-top-page-with-nextjs/
---

# PR TIMESのトップページをNext.jsにリプレイスしました

**Author:** 柳 龍哉 ([@apple_yagi](https://twitter.com/apple_yagi))
**Date:** 2024年6月6日

こんにちは、フロントエンドエンジニアのやなぎです。プレスリリース掲載ページ、キーワード検索ページに続き、PR TIMESの[トップページ](https://prtimes.jp/)を PHP + Smarty + jQuery から Next.js（Pages Router）にリプレイスしました。

## SP用ページで使用するカルーセルライブラリの選定

トップページのSP版では、横スワイプでタブを切り替えることができます。このスワイプ操作を実装するため、以下のカルーセルライブラリを検討しました：

- Swiper
- Nuka Carousel
- React Slick
- Embla Carousel

最終的に、Embla Carouselを選択した理由は以下の通りです：

- スクロール位置のリセット問題を解決
- スワイプの閾値が適切
- アクセシビリティの問題がない
- shadcn/uiやyamada-uiなどで使用実績がある

## BFCacheを有効化

トップページでBFCacheを有効にし、以下のメリットを実現：

- ブラウザバック時に元のタブ位置を保持
- 新着タブの無限スクロールデータを保持

## キャッシュの設定

CDNでServer Side RenderingのHTMLをキャッシュする戦略を採用：

- Surrogate-Control ヘッダーを使用
- キャッシュ設定：`max-age=5, stale-while-revalidate=10`
- Fastlyで約90%のキャッシュヒット率を実現
- 実装後、レスポンス時間が大幅に改善

## まとめ

> 「行動者発の情報が、人の心を揺さぶる時代へ」

PR TIMESのトップページリプレイスにより、モダンなフロントエンド技術スタックへの移行を完了しました。

> 「今後はアプリケーション全体に目を向け、実現したいUX（ユーザーエクスペリエンス）を意識しながら、アーキテクトに取り組んでいきたいと思っています。」

このマイグレーションは、PR TIMESのフロントエンドインフラストラクチャを現代化する重要な一歩を表しています。
