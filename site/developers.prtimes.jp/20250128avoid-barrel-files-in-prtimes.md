---
https://developers.prtimes.jp/2025/01/28/avoid-barrel-files-in-prtimes/
---

# eslint-plugin-no-barrel-filesを導入してBarrel filesをやめた話

こんにちは、フロントエンドエンジニアのやなぎ（[@apple\_yagi](https://twitter.com/apple_yagi)）です。

PR TIMESでは[フロントエンドのReactリプレイス当初](https://developers.prtimes.jp/2021/11/10/replace-react/)より Barrel file を作成するルールがありました。しかし、先日そのルールを廃止することに決めたため、その経緯についてご紹介します。

目次

## Barrel filesとは

Barrel filesとは複数のモジュールを1つのファイル（Barrel file）にまとめてexportすることを指します。以下の例では`utils/test.ts` 、`utils/fuga.ts` 、`utils/hoge.ts` で export されている定数を `utils/index.ts` で再exportしています。

```
// utils/test.ts
export const test = 1;

// utils/fuga.ts
export const fuga = 2;

// utils/hoge.ts
export const hoge = 3;

// utils/index.ts
export * from './test';
export * from './fuga';
export * from './hoge';
```

これによって、`utils/test.ts`、`utils/fuga.ts`、`utils/hoge.ts`の各ファイルを使用する際に、import文をスッキリ書くことが可能です。

```
import {test, fuga, hoge} from './utils';

// Barrel fileを使用しない場合
import {test} from './utils/test';
import {fuga} from './utils/fuga';
import {hoge} from './utils/hoge';
```

## Barrel files をやめた経緯

前述した通り、PR TIMESではReactリプレイス当初から Barrel file の作成をルールとしていました。しかし、それから数年が経過した現在、そのルールはあまり浸透しておらず、メンバー間で実施の有無が分かれていました。これは、明文化されたコーディングルールがなく、Pull Requestのレビューでの指摘に依存していたためです。

Reactリプレイスを始めた当時はフロントエンドメンバーがわずか3人でしたが、今ではバックエンドエンジニアもフロントエンド開発に参加するようになり、チームは10人を超える規模になりました。そのため、レビューを通じてのみルールを徹底させるのは現実的ではなくなっています。

あわせて読みたい

さらに、昨年[lintツールをXOに統一した](https://developers.prtimes.jp/2024/02/09/frontend-lint-tools-unified-xo/)際、Barrel filesが原因で[import/no-cycle](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-cycle.md)のlintエラーが多発していました。一時的にはinlineでルールを無効にして対処していましたが、Barrel filesが循環参照のリスクを引き起こすことが明らかになりました。以下の記事にもこのことについて言及されています。

あわせて読みたい

Why you probably shouldn't be adding index.ts files everywhere

また、最近ではBarrel filesのデメリットについて言及する記事が増え、Next.jsやJest、Vitestなどのテストランナーでパフォーマンスの劣化が生じるとの報告もありました。現在のPR TIMESではBarrel fileの数は多くないため、パフォーマンスに影響は感じていませんが、将来的に顕著になる可能性があります。

あわせて読みたい

Vercel

How solving barrel files led to faster cold boots and build times.

これらの理由を踏まえ、Barrel filesの使用をやめ、[eslint-plugin-no-barrel-files](https://github.com/art0rz/eslint-plugin-no-barrel-files)を導入することにしました。

## eslint-plugin-no-barrel-filesの選定理由

Barrel filesを禁止するlintルールを提供しているeslint pluginはeslint-plugin-no-barrel-files以外にも以下のようにいくつか存在します。

* <https://github.com/thepassle/eslint-plugin-barrel-files>
* <https://github.com/gajus/eslint-plugin-canonical>

1つ目の eslint-plugin-barrel-files はflat config化対応がまだされていなかったため、使用しませんでした。

GitHub

2つ目の eslint-plugin-canonical については、今回のように Barrel files の使用を制限したいという目的に対して付属しているルールが多く、fatだと判断したため使用しませんでした。しかし、今後 eslint-plugin-canonical にある他のルールを適用したいと思ったタイミングで移行する可能性がありそうです。

## まとめ

今回、レビューでのみ運用していたBarrel filesのルールを見直し、廃止することにしました。また、機械的にルールを運用できるようにeslint pluginを追加し、レビューでの指摘のみに頼らないルール運用を開始しました。

PR TIMESのフロントエンドのコーディングルールはドキュメントを作って運用とはしておらず、ESLintを正としてコーディングルールを運用しています。今回のBarrel filesのルールはESLintで運用されていなかったため、曖昧な状態で運用されていました。この教訓を生かして、今回のような曖昧なルールを見つけた際はESLintによって管理していきたいと思います。

## We are hiring

フロントエンドエンジニアを含む各種ポジションでの採用を進めています！興味があればぜひご応募ください。

株式会社PR TIMES

株式会社PR TIMESでは現在03-4. 開発部 フロントエンドエンジニアを募集しています。

株式会社PR TIMES

株式会社PR TIMESが公開している、02．開発部 の求人一覧です
