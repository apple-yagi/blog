---
https://developers.prtimes.jp/typescript-go-typecheck-9x-speedup/
---

# PR TIMES のフロントエンド環境に typescript-go を導入したら typecheck が 9 倍速くなりました

**Author:** 柳 龍哉 ([@apple_yagi](https://twitter.com/apple_yagi))
**Published:** 2025 年 11 月 12 日

こんにちは、フロントエンドエンジニアのやなぎ（[@apple_yagi](https://twitter.com/apple_yagi)）です。

先日 PR TIMES のフロントエンド環境に [typescript-go](https://github.com/microsoft/typescript-go) を導入し、 tsgo（ typescript-go の実行コマンド） で typecheck を実行するようにしました。その結果、typecheck の実行時間が最大で 9 倍速くなったので、導入に向けて行った対応と結果をご紹介します。

## typescript-go とは

typescript-go は Microsoft によって Go 言語で書き直されている TypeScript のコンパイラです。TypeScript 7 からはこのコンパイラが使用される予定で、従来よりも最大 10 倍高速になると発表されています。公式ブログの[アナウンス](https://devblogs.microsoft.com/typescript/typescript-native-port/)でもプロジェクトの背景が詳しく説明されています。

[README](https://github.com/microsoft/typescript-go?tab=readme-ov-file#what-works-so-far) には進捗が随時まとめられており、2025 年 11 月 12 日時点では Type checking や Type resolution などの実装が完了しています。そのため、PR TIMES のフロントエンド環境でも typecheck のみであれば導入ができると考えました。

## typescript-go の導入

typescript-go は `@typescript/native-preview` をインストールするだけで導入できます。

```bash
pnpm install @typescript/native-preview
```

しかし、PR TIMES のフロントエンド環境の `tsconfig.json` に typescript-go では使用できないオプションが 2 つあったため、先に設定を見直しました。

### baseUrl の指定を削除する

typescript-go では [baseUrl](https://www.typescriptlang.org/tsconfig/#baseUrl) をサポートしていないため、tsconfig から削除する必要があります。

```json
{
    "compilerOptions": {
        ...,
        "baseUrl": "."
    }
}
```

PR TIMES では `baseUrl` が `"."` だったので削除自体は簡単でしたが、paths alias の書き方と一部のファイルの import 文を修正する必要がありました。

paths alias を絶対パスで記述していた場合、相対パスで記述する必要があります。補足として TypeScript 4.1 より前のバージョンでは paths alias を指定する際に baseUrl が必要でしたが、4.1 以降は baseUrl なしで paths alias を設定することができます。

```json
{
    "compilerOptions": {
        ...,
        "paths": {
-           "@/*": ["src/*"],
+           "@/*": ["./src/*"],
        }
    }
}
```

一部のファイルの import 文の修正に関しては `baseUrl` からの絶対パスで import している箇所があったため、相対パスで import するように修正しました。

```ts
- import type { User } from "types";       // baseUrl からの絶対パスで import している
+ import type { User } from "../../types"; // 相対パスに変更

export const user: User = {
    id: 1,
    name: "山田太郎",
};
```

### moduleResolution を node から bundler に変更する

typescript-go では [moduleResolution](https://www.typescriptlang.org/tsconfig/#moduleResolution) に `node` を使用できなくなります。そのため、他のオプションと比べて影響が少ない `bundler` に変更しました。

```json
{
    "compilerOptions": {
        ...,
-       "moduleResolution": "node"
+       "moduleResolution": "bundler"
    }
}
```

`node` から `bundler` に変更すると、`exports` フィールドを前提とした解決が厳密になるため、以下のようにライブラリ内部のファイルを直接 import していた箇所でエラーが発生しました。`react-datepicker` では `dist` 以下が `exports` で閉じられており、`bundler` ではそこへ到達できないためです。

```ts
- import ReactDatePicker, {registerLocale} from 'react-datepicker';
- import {type ReactDatePickerCustomHeaderProps} from 'react-datepicker/dist/calendar';
+ import ReactDatePicker, {
+   registerLocale,
+   type ReactDatePickerCustomHeaderProps,
+ } from 'react-datepicker';
```

そのため、ルートで再エクスポートされている API をまとめて import する形に rewrite することで解決しました。

他にも typescript-go で使用できなくなる tsconfig オプションをいくつかありますが、PR TIMES では以上の修正で済みました。その他の tsconfig オプションについては [TypeScript #54500](https://github.com/microsoft/TypeScript/issues/54500) に記載されています。

## typescript-go で typecheck した結果

GitHub Actions 上で typecheck を行った結果、以下の通りになりました。PR TIMES では一部の workflow で [blacksmith](https://www.blacksmith.sh/) を runner として利用しているため、`ubuntu-24.04` と `blacksmith-4vcpu-ubuntu-2404` の両方を比較しています。

| Tool | Runner                       | Duration | Speedup vs. tsc |
| ---- | ---------------------------- | -------- | --------------- |
| tsc  | ubuntu-24.04                 | 42 秒    | 1.0x            |
| tsgo | ubuntu-24.04                 | 17 秒    | 2.5x            |
| tsc  | blacksmith-4vcpu-ubuntu-2404 | 28 秒    | 1.0x            |
| tsgo | blacksmith-4vcpu-ubuntu-2404 | 3 秒     | 9.3x            |

ubuntu-24.04 では tsc から tsgo に変えても約 2.5 倍の短縮でしたが、blacksmith-4vcpu-ubuntu-2404 では 9 倍以上速くなるという結果になりました。Go 製の tsgo は CPU を使い切ってこそ真価を発揮するので、Runner ごとのコア数などがスピードアップ幅を左右することが分かります。

## まとめ

PR TIMES のフロントエンド環境に typescript-go を導入し、blacksmith-4vcpu-ubuntu-2404 の環境において typecheck の実行速度が最大で約 9 倍速くなりました。速くなったこともメリットとしてはありましたが、TypeScript 7 で typescript-go が標準になることを考えると今のうちから typecheck などで導入することにより、バージョンアップがスムーズになると考えています。

## We are hiring!

フロントエンドエンジニアを含む各種ポジションでの採用を進めています。興味があればぜひご応募ください。

株式会社 PR TIMES
[採用情報](https://prtimes.co.jp/culture/recruit)
