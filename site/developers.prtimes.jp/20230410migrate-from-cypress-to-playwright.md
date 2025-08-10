---
https://developers.prtimes.jp/2023/04/10/migrate-from-cypress-to-playwright/
---

# CypressからPlaywrightに移行しました

**Author:** 柳 龍哉  
**Date:** 2023年4月10日

## なぜ移行したのか

PR TIMESでは、E2Eテストツールを**Cypress**から**Playwright**に移行しました。主な移行理由は以下の3点でした：

### 1. テスト実行時間の問題

Cypressは並列テスト実行ができず、テスト完了に時間がかかっていました。

### 2. Visual Regression Test (VRT)の課題

Visual Regression Testを単体で簡単に実施できませんでした。

### 3. 開発体験の向上

async/await でコードを記述でき、Codegen機能が使用できることが魅力的でした。

## 移行作業について

### 作業規模と期間

- **元々のテストケース**: 30個程度
- **移行期間**: 1週間で完了
- **移行の難易度**: Playwrightは Jest や Vitest に似た関数が用意されており、テストコード作成が容易

### 移行作業の流れ

1. Playwright環境のセットアップ
2. 既存Cypressテストの分析
3. Playwrightでのテスト書き換え
4. Visual Regression Testの実装
5. CI/CD環境での動作確認

## テスト実行結果の比較

実行時間の劇的な改善が得られました：

- **Cypress**: 136秒
- **Playwright**: 39秒

**約3.5倍の高速化**を実現しました。

## Visual Regression Test (VRT)の実装

Playwrightでは、VRTの実装が非常にシンプルになりました：

```javascript
test('VRT', async ({ page }) => {
  await page.goto('/');
  
  // タイトルが表示されるか確認する
  await expect(page.getByText('タイトル')).toBeVisible();
  
  // VRT（Visual Regression Test）
  await expect(page).toHaveScreenshot('VRT.png');
});
```

### VRTの特徴

- **簡単な実装**: `toHaveScreenshot()`メソッドで一行で実装可能
- **差分検出**: 自動でピクセル単位の差分を検出
- **ベースライン管理**: スクリーンショットのベースラインを自動管理

## Playwrightの主要メリット

### 1. 並列実行によるパフォーマンス向上

```bash
# 複数のワーカーで並列実行
npx playwright test --workers=4
```

### 2. モダンなAPIとDeveloper Experience

```javascript
// async/awaitベースの直感的なAPI
await page.click('button[data-testid="submit"]');
await expect(page.locator('.result')).toContainText('成功');
```

### 3. Codegen機能

```bash
# ブラウザ操作を自動でコード生成
npx playwright codegen https://example.com
```

### 4. 複数ブラウザ対応

- Chromium
- Firefox
- Safari (WebKit)

## 移行で得られた効果

### 定量的効果

- **テスト実行時間**: 136秒 → 39秒（71%短縮）
- **VRT導入**: 0個 → 全テストケースで実装

### 定性的効果

- **開発者体験の向上**: より直感的なテスト記述
- **デバッグの効率化**: 詳細なエラー情報とトレース
- **保守性の向上**: テストコードの可読性向上

## 移行時の注意点

### 学習コスト

- 新しいAPIの習得が必要
- VRTのベースライン画像の初回作成

### CI/CD環境の調整

- Docker環境での実行設定
- スクリーンショット差分の管理

## まとめ

CypressからPlaywrightへの移行により、以下を実現しました：

1. **大幅な実行時間短縮**（約3.5倍の高速化）
2. **Visual Regression Testの簡易導入**
3. **より良い開発者体験**

移行作業は1週間程度で完了し、得られるメリットが移行コストを大きく上回る結果となりました。E2Eテストの実行時間やVRTの導入を検討している場合、Playwrightへの移行を強く推奨します。
