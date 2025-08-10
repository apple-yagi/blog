---
https://developers.prtimes.jp/2024/08/14/replace-form-with-many-inputs-tips/
---

# 入力項目の多いフォームをReactにリプレイスする際に工夫したこと

**Author:** 柳 龍哉 ([@apple_yagi](https://twitter.com/apple_yagi))  
**Date:** 2024年8月14日

## はじめに

こんにちは、フロントエンドエンジニアのやなぎです。

先日、PR TIMES上で企業アカウントを作成する際に情報を入力する企業登録申請フォームをReact + Viteにリプレイスしました。

## 企業登録申請フォームをリプレイスする難しさ

企業登録申請フォームは入力項目が30項目あり、データ量が多いフォームとなっています。このようなフォームをシングルページアプリケーションで実装する時に難しくなるのが、フロントエンドとバックエンドのバリデーションをどのように同期させるかです。

## フロントエンドで使用しているライブラリ

PR TIMESではフォームを実装する際に以下のライブラリを使用しています：

- React Hook Form
- Zod
- @hookform/resolvers
- Tanstack Query
- openapi-generator

## バックエンドからのエラーレスポンスを入力項目にマッピングする

APIからのエラーレスポンスのスキーマ例：

```json
{
  "message": "Bad Request",
  "status": 400,
  "errors": [
    {
      "code": "required",
      "message": {
        "ja": "企業名を入力してください",
        "en": "Corporate name is required"
      },
      "field": "companyInfo.corporateName"
    }
  ]
}
```

## バリデーションエラーをNew Relicに送信

バリデーションエラーが発生した際は、New Relicにエラー情報を送信して監視できるようにしています。これにより、フロントエンドとバックエンドの両方でバリデーション問題を特定できます。

## バリデーション戦略

- Zodを使用してバリデーションルールを定義
- React Hook Formと@hookform/resolversを通じて統合
- バックエンドが具体的なエラー詳細を提供できる柔軟なバリデーションを実装

## まとめ

このアプローチはフロントエンドとバックエンド間にある程度の結合を生みますが、エラー処理の簡素化というメリットが潜在的なデメリットを上回ります。

慎重なエラーマッピングによりフロントエンドの複雑さが削減され、包括的なエラー追跡によりフォームの信頼性が向上します。
