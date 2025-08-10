---
https://developers.prtimes.jp/2022/08/22/build_monorepo_using_yarn-workspaces/
---

# Yarn Workspacesを利用したMonorepo環境の構築

**Author:** 柳 龍哉 (Ryuya Yanagi)  
**Date:** 2022年8月22日

## はじめに

この記事では、PR TIMESのフロントエンドチームが**Yarn Workspaces**を利用して、デザインシステムのコンポーネントやスタイルを複数のReactアプリケーション間で集約・標準化するためのMonorepo環境を構築した経験について紹介します。

## 背景と動機

### 課題

PR TIMESのフロントエンド開発において、以下のような課題がありました：

- **デザインシステムコンポーネントの散在**: 異なるプロジェクト間でコンポーネントが分散
- **重複実装**: 同様の機能を持つコンポーネントが複数のプロジェクトで個別に実装
- **一貫性の欠如**: 統一されたフロントエンド開発アプローチが不足

### 目標

これらの課題を解決するために、以下を目指しました：

1. デザインシステムコンポーネントの一元管理
2. 重複コードの削減
3. より統一されたフロントエンド開発アプローチの実現

## Monorepo構成

### ディレクトリ構造

新しいディレクトリ構造は以下のようになりました：

```
.
├── apps
│   ├── app1
│   └── app2
├── packages
│   ├── design-system
│   └── tsconfig
└── package.json
```

### 構成要素の説明

- **`apps/`**: 実際のアプリケーションを配置
  - `app1`, `app2`: 各々のReactアプリケーション
- **`packages/`**: 共有パッケージを配置
  - `design-system`: 共有デザインシステムコンポーネント
  - `tsconfig`: TypeScript設定の共有

## 実装手順

### 1. ディレクトリの再構成

既存のプロジェクト構造を共有パッケージとアプリケーションに分離しました。

### 2. package.jsonの設定

ワークスペース設定をルートの`package.json`に追加：

```json
{
  "name": "prtimes-frontend-monorepo",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "devDependencies": {
    "yarn": "^1.22.0"
  }
}
```

### 3. 依存関係の設定

各アプリケーションの`package.json`で共有パッケージを依存関係として追加：

```json
{
  "name": "app1",
  "dependencies": {
    "@prtimes/design-system": "*",
    "react": "^18.0.0"
  },
  "devDependencies": {
    "@prtimes/tsconfig": "*"
  }
}
```

### 4. デザインシステムパッケージの作成

```json
// packages/design-system/package.json
{
  "name": "@prtimes/design-system",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "peerDependencies": {
    "react": ">=16.8.0",
    "react-dom": ">=16.8.0"
  }
}
```

## Yarn Workspacesの活用

### コマンドの実行例

```bash
# 全パッケージの依存関係をインストール
yarn install

# 特定のワークスペースでコマンド実行
yarn workspace app1 start
yarn workspace @prtimes/design-system build

# 全ワークスペースで同じコマンドを実行
yarn workspaces run test
```

### 依存関係の管理

```bash
# 特定のワークスペースに依存関係を追加
yarn workspace app1 add lodash
yarn workspace @prtimes/design-system add -D typescript

# ルートに開発依存関係を追加
yarn add -W -D eslint prettier
```

## 実現した効果

### 1. コンポーネントの一元管理

- **Before**: 各プロジェクトで個別にボタン、フォーム、モーダルなどのコンポーネントを実装
- **After**: `@prtimes/design-system`パッケージで一元管理

### 2. 重複コードの削減

```typescript
// Before: 各アプリで個別実装
// apps/app1/src/components/Button.tsx
// apps/app2/src/components/Button.tsx

// After: 共有パッケージから利用
import { Button } from '@prtimes/design-system';
```

### 3. 一貫性の向上

- **スタイル**: 統一されたデザインシステム
- **TypeScript設定**: 共有の`tsconfig`
- **コーディング規約**: 一元化されたESLint/Prettier設定

### 4. 依存関係管理の簡素化

- **ホイスティング**: 共通の依存関係をルートで管理
- **バージョン統一**: 各パッケージ間でのライブラリバージョンの一貫性

## 開発体験の改善

### Hot Module Replacement (HMR)

```bash
# 開発時は両方を並行して実行
yarn workspace @prtimes/design-system dev &
yarn workspace app1 start
```

### TypeScriptサポート

```typescript
// 型安全な import
import { Button, Modal, Form } from '@prtimes/design-system';

// 共有型定義の利用
import type { Theme, ComponentProps } from '@prtimes/design-system';
```

## 課題と解決策

### 1. ビルド順序の管理

**課題**: パッケージ間の依存関係によるビルド順序

**解決策**:

```json
{
  "scripts": {
    "build": "yarn workspace @prtimes/design-system build && yarn workspaces run build"
  }
}
```

### 2. 開発時の変更反映

**課題**: デザインシステムの変更がアプリケーションに即座に反映されない

**解決策**: watch モードとシンボリックリンクの活用

### 3. パッケージ間のバージョン管理

**課題**: 互換性の維持

**解決策**: セマンティックバージョニングの厳格な運用

## まとめ

Yarn Workspacesを使用したMonorepo環境の構築により、以下を実現しました：

### 技術的成果

1. **デザインシステムの一元管理**
2. **重複コードの大幅削減**
3. **アプリケーション間の一貫性向上**
4. **依存関係管理の簡素化**

### 開発プロセスの改善

1. **開発効率の向上**: 共有コンポーネントの再利用
2. **保守性の向上**: 一箇所の変更で全アプリケーションに反映
3. **品質の向上**: 統一されたコンポーネントによる一貫したUX

### 今後の展望

この基盤を活用して、さらなるフロントエンド開発環境の改善を進めていく予定です：

- **テスト環境の統一**: 共有テストユーティリティの導入
- **デプロイパイプラインの最適化**: Monorepo対応のCI/CD構築
- **ドキュメンテーションの強化**: Storybookによるコンポーネントカタログ

PR TIMESのフロントエンドチームは、このMonorepo環境により、より効率的で一貫性のある開発を実現し、将来の改善に向けた強固な基盤を築くことができました。
