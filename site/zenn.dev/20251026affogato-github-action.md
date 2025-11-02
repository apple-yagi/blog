---
https://zenn.dev/apple_yagi/articles/cd4fa9d5123cf8
---

# affogato で影響のあるテストだけ実行する GitHub Actions のワークフローを構築する

**Author:** やなぎ
**Published:** 2025/10/26

## はじめに

コードベースが大きくなり、ユニットテストの量が増えていくと、当然 CI の時間も長くなっていきます。そのため、[tj-actions/changed-files](https://github.com/tj-actions/changed-files) を使って「features 以外に変更があれば全テスト」「features に閉じていれば対象ディレクトリだけ実行」というワークフローを組むことがあります。

```yaml
jobs:
  detect-changed:
    runs-on: ubuntu-latest
    outputs:
      is-all-test-needed: ${{ steps.filter.outputs.any_modified }}
    steps:
      - uses: actions/checkout@v4
      - name: Detect non-feature changes
        id: filter
        uses: tj-actions/changed-files@v47
        with:
          files: |
            **
          files_ignore: |
            src/features/**

  unit-test-all:
    needs: detect-changed
    if: needs.detect-changed.outputs.is-all-test-needed == 'true'
    steps:
      - run: pnpm run test

  unit-test-features:
    needs: detect-changed
    if: needs.detect-changed.outputs.is-all-test-needed == 'false'
    steps:
      - uses: actions/checkout@v4
      - name: Detect modified feature directories
        id: filter
        uses: tj-actions/changed-files@v47
        with:
          files_yaml: |
            src/features/hoge:
              - src/features/hoge/**
            src/features/fuga:
              - src/features/fuga/**
      - name: Run feature tests
        if: steps.filter.outputs.modified_keys != ''
        run: pnpm run test ${{ steps.filter.outputs.modified_keys }}
```

しかしこの方法では新しい feature ディレクトリを追加するたびに `files_yaml` を更新しないと漏れが生まれてしまいます。また、utils などの共有ディレクトリ内のファイルを更新した場合は依存関係がわからないため、全テストを回さざるを得ません。例えば `src/utils/format-date.ts` を更新したとき、ディレクトリ単位のマッピングでは `hoge` と `fuga` それぞれのテストに影響することを定義しきれません。

```text
src/
  features/
    hoge/
    fuga/
  utils/
    format-date.ts
```

そこで、依存関係を解決して本当に影響を受けるテストだけを抽出する GitHub Action「affogato」を作成しました。

## affogato の使い方

最小構成は次のとおりです。`token` には `GITHUB_TOKEN` を渡し、出力された `affected_tests` をそのままテストコマンドに渡します。affected_tests には変更されたファイルに影響を受けるテストのファイル名が入っています。

```yaml
name: ci
on: pull_request

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: apple-yagi/affogato@v1
        id: affogato
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
      - run: pnpm install --frozen-lockfile
      - run: pnpm test ${{ steps.affogato.outputs.affected_tests }}
```

モノレポで `tsconfig.json` が複数ある場合は、対象となるプロジェクトのパスを指定します。

```yaml
- uses: apple-yagi/affogato@v1
  id: affogato
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    tsconfig: apps/web/tsconfig.json
```

デフォルトでは `*.test.ts(x)` と `*.spec.ts(x)` が対象ですが、Storybook Test を実行したいときなどは拡張子を指定することもできます。

```yaml
- uses: apple-yagi/affogato@v1
  id: affogato
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    test_patterns: stories # *.stories.ts(x) が抽出される
- run: pnpm test-storybook ${{ steps.affogato.outputs.affected_tests }}
```

## 仕組みの概要

まず、GitHub API と `git diff` でベース SHA とヘッド SHA 間の変更ファイルを取得します。

<https://github.com/apple-yagi/affogato/blob/main/src/get-changed-files.ts>

その次に [ts-morph](https://github.com/dsherret/ts-morph) を使用して TypeScript の依存グラフを構築し、import の逆依存を辿って影響を受けたファイルのテストファイルを列挙しています。

<https://github.com/apple-yagi/affogato/blob/main/src/get-affected-tests.ts>

## まとめ

affogato を導入することで影響のあるテストだけを実行することができるようになるのでぜひ試してみてください！

https://github.com/apple-yagi/affogato
