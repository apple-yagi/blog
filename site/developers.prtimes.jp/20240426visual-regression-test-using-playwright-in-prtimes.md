---
https://developers.prtimes.jp/2024/04/26/visual-regression-test-using-playwright-in-prtimes/
---

# PR TIMESにおけるPlaywrightを用いたVisual Regression Test

**Author:** 柳 龍哉
**Date:** 2024年4月26日

## 前提

PR TIMESは、React + Vite製のアプリケーションとNext.js製のアプリケーションが存在します。本記事では、React + Vite製のアプリケーションに対するVisual Regression Test（VRT）について紹介します。

## VRTの実行環境

VRTは公式のDocker Imageを用いて、Docker上で実行しています。以下のようなscriptsを定義し、`pnpm run test`でVRTをDocker上で実行します。

```json
{
  "scripts": {
    "_docker": "docker run --rm --ipc=host -v $(pwd):/workspace mcr.microsoft.com/playwright:v$(node -e 'console.log(require(\"./package.json\").devDependencies[\"@playwright/test\"])')-jammy",
    "test": "pnpm run _docker npm run _test"
  }
}
```

Docker上で実行する理由:

- 実行環境のOSを統一するため
- 文字のフォントを揃えるため

## ReactアプリケーションのVRT向けの配信方法

PR TIMESでは、[hono](https://github.com/honojs/hono)を用いてアプリケーションを配信しています。これは、アプリケーションのルーティング方法や、クエリパラメータに応じて異なるHTMLを返す必要があるためです。

## Playwrightの設定

```javascript
const config: PlaywrightTestConfig = {
  webServer: {
    command: 'npm run serve',
    port: 9090,
    reuseExistingServer: true,
  },
  use: {
    baseURL: 'http://localhost:9090',
    video: isCi ? 'on-first-retry' : 'on',
    testIdAttribute: 'data-testid',
  },
  expect: {
    toHaveScreenshot: {
      animations: 'disabled',
      maxDiffPixelRatio: 0,
      threshold: 0.075,
    },
  },
  retries: isCi ? 1 : 0,
}
```

## テスト戦略

### APIのモック化

Playwrightの`fulfill`機能を使用してAPIレスポンスをモック化

### 画面比較

`toHaveScreenshot`を使用してスクリーンショットの安定した比較を実行

### 動的要素のマスク

画像などの動的要素をマスクして一貫した結果を確保

### 並列実行

テストの並列実行により効率化

## CI/CD統合

- GitHub Actionsワークフロー
- テスト結果をAWS S3にアップロード
- 詳細なHTMLレポートを生成
- Pull Requestコメントにレポートリンクを追加

## 成果

2024年4月現在、約250のスナップショットを管理し、以下のメリットを実現：

- テストのflakiness削減
- Visual Regression Testingの簡素化
- フロントエンド変更に対する信頼性向上
