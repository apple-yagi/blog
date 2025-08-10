---
https://developers.prtimes.jp/2024/11/07/happy-css-modules-enhancements-implementation/
---

# happy-css-modulesに機能追加して実際にプロダクトに適用した話

**Author:** 柳 龍哉 ([@apple_yagi](https://twitter.com/apple_yagi))
**Date:** 2024年11月7日

## はじめに

こんにちは、フロントエンドエンジニアのやなぎです。PR TIMESではEmotionからCSS Modulesへの移行作業を進めており、その過程で[happy-css-modules](https://github.com/mizdra/happy-css-modules)を使用しています。

## happy-css-modulesで改善したかった点

従来のhappy-css-modulesには2つの主な課題がありました：

1. CSS Modulesの型定義ファイルを同じディレクトリに生成するため、ディレクトリが視認性の低いものになっていた
2. CSSファイルの名前変更時に古い型定義ファイルが残ってしまう

## 実装した機能: outDirオプション

Pull Requestで`outDir`オプションを実装し、型定義ファイルを任意のディレクトリに出力できるようにしました。

## プロダクトへの導入

PR TIMESでは、以下のように設定しました：

```json
{
  "scripts": {
    "hcm": "hcm 'src/**/*.module.css' -o generated/hcm",
    "hcm:w": "hcm -w 'src/**/*.module.css' -o generated/hcm"
  },
  "compilerOptions": {
    "rootDirs": ["src", "generated/hcm/src"]
  }
}
```

## monorepo環境での注意点

pnpmを利用したmonorepo環境では、別パッケージのコンポーネントをインポートする際に適切な設定が必要です。
