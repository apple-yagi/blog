---
https://developers.prtimes.jp/2025/06/10/migrate-from-happy-css-modules-to-css-modules-kit/
---

# happy-css-modulesからcss-modules-kitに移行しました

こんにちは、フロントエンドエンジニアのやなぎ（[@apple\_yagi](https://twitter.com/apple_yagi)）です。

先月公開したエントリーでもご紹介した通り、PR TIMESのフロントエンドは現在CSS Modulesを使用してスタイリングを行っています。

あわせて読みたい

上記のエントリーでは[happy-css-modules](https://github.com/mizdra/happy-css-modules)を使用してCSS Modulesの型定義ファイルを生成していましたが、先日[css-modules-kit](https://github.com/mizdra/css-modules-kit/)に移行したのでご紹介します。

目次

## css-modules-kit とは

css-modules-kitはCSS Modulesを便利にするためのツールキットで以下の機能を提供しています。

* CSS Modulesの型定義ファイルを生成する機能
* VS CodeやNeovimなどのエディタに言語機能を提供する機能
* CSS Modules向けのESLint、Stylelintプラグイン

happy-css-modulesは、上記の機能のうち型定義ファイルを生成する機能のみを提供していました。一方、css-modules-kitはそれに加えて、エディタの言語機能を拡張するTypeScript PluginやLintルールも提供しており、CSS Modulesを記述する際の開発者体験をhappy-css-modulesよりも向上させています。

## css-modules-kitのメリット

これまで使用していたhappy-css-modulesではコードを変更するたびに型定義ファイルを再生成する必要があるため、ターミナルでwatchオプションを常時起動させておく必要がありました。

```
# 開発中は以下のコマンドを常に実行しておく
$ pnpm hcm -w
```

happy-css-modulesが生成した型定義ファイルにより、定義されていないCSSセレクタの使用を防止できたり、コードジャンプが可能となったりするメリットがありましたが、ターミナルで常時型定義ファイルを生成するコマンドを実行しておくのは少々面倒でした。

GitHub

A toolkit for making CSS Modules useful. Contribute to mizdra/css-modules-kit development by creating an account on GitHub.

またどのようにこの機能を実現しているかは先日あったTSKaigiで発表されていたので気になる方はぜひご覧ください。

## 移行手順

happy-css-modulesからの移行はとても簡単で以下のように移行できました。

1. VS Codeに <https://marketplace.visualstudio.com/items?itemName=mizdra.css-modules-kit-vscode> をインストールする
2. happy-css-modulesをuninstallし、既存の（happy-css-modulesで生成された）型定義ファイルを削除する
3. [@css-modules-kit/codegen](https://www.npmjs.com/package/%40css-modules-kit/codegen)をinstallし、型定義ファイルを生成するコマンドを追加する
4. tsconfig.jsonの `compilerOptions.rootDirs` を修正する

弊社のフロントエンドメンバーはVS Codeを使用しているメンバーがほとんどのためVS Codeのプラグインをインストールしていますが、他のエディターでも利用可能です。

GitHub

A toolkit for making CSS Modules useful. Contribute to mizdra/css-modules-kit development by creating an account on GitHub.

`@css-modules-kit/codegen` を使用することでhappy-css-modulesのように型定義ファイルを生成することは可能ですが、型定義ファイルのデフォルトの生成場所が `generated` ディレクトリになっているため、tsconfig.jsonの `compilerOptions.rootDirs` を以下のように修正します。

```
{
  "compilerOptions": {
    "rootDirs": [".", "generated"]
  }
}
```

`compilerOptions.rootDirs` は、複数のディレクトリをまとめてひとつの仮想的なルートディレクトリとして扱うための設定です。指定した複数のディレクトリ（たとえば `"."` と `"generated"`）を、同じ階層にあるものとしてTypeScriptが認識してくれるようになります。これにより、型定義ファイルが別ディレクトリにあってもCSS Modulesの型として使用することが可能になります。

型定義ファイルを生成するディレクトリを変更したい場合は `cmkOptions.dtsOutDir` を設定することで変更できます。例えば、 `cmk` ディレクトリに生成するように変更したい場合、以下のような設定ができます。

```
{
  "compilerOptions": {
    "rootDirs": [".", "cmk"]
  },
  "cmkOptions": {
    "dtsOutDir": "cmk" // generatedからcmkに変更
  }
}
```

PR TIMESでは以前からhappy-css-modulesのoutDirオプションを使用して `generated`ディレクトリに型定義を生成していたため、特に影響はありませんでした。happy-css-modulesのoutDirについては以下のエントリーで紹介しているので合わせてご覧ください。

あわせて読みたい

## まとめ

happy-css-modulesからcss-modules-kitに移行したことにより、CSS Modulesを書く際の開発者体験がより一層良くなりました。移行してから1週間ほど経ちますが、不具合などの連絡もあがっておらず、happy-css-modulesよりもさらにhappyにCSS Modulesを書くことができるようになりました。

## We are hiring!

フロントエンドエンジニアを含む各種ポジションでの採用を進めています！興味があればぜひご応募ください。

株式会社PR TIMES

株式会社PR TIMESでは現在03-4. 開発部 フロントエンドエンジニアを募集しています。

株式会社PR TIMES

株式会社PR TIMESが公開している、02．開発部 の求人一覧です
