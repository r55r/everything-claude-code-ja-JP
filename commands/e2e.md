---
description: Playwrightでエンドツーエンドテストを生成・実行します。テストジャーニーを作成し、テストを実行し、スクリーンショット/ビデオ/トレースをキャプチャし、アーティファクトをアップロードします。
---

# E2E Command

このコマンドは**e2e-runner**エージェントを呼び出し、Playwrightを使用してエンドツーエンドテストの生成、保守、実行を行います。

## このコマンドの機能

1. **テストジャーニーの生成** - ユーザーフロー用のPlaywrightテストを作成
2. **E2Eテストの実行** - 複数のブラウザでテストを実行
3. **アーティファクトのキャプチャ** - 失敗時のスクリーンショット、ビデオ、トレース
4. **結果のアップロード** - HTMLレポートとJUnit XML
5. **不安定なテストの特定** - 不安定なテストを隔離

## 使用タイミング

以下の場合に `/e2e` を使用:
- 重要なユーザージャーニーのテスト（ログイン、取引、決済）
- マルチステップフローがエンドツーエンドで動作することの検証
- UIインタラクションとナビゲーションのテスト
- フロントエンドとバックエンド間の統合の検証
- 本番デプロイメントの準備

## 動作の仕組み

e2e-runnerエージェントは以下を実行します:

1. **ユーザーフローの分析**とテストシナリオの特定
2. Page Object Modelパターンを使用した**Playwrightテストの生成**
3. 複数のブラウザ（Chrome、Firefox、Safari）での**テスト実行**
4. スクリーンショット、ビデオ、トレースでの**失敗のキャプチャ**
5. 結果とアーティファクトを含む**レポートの生成**
6. **不安定なテストの特定**と修正の推奨

## 使用例

```
User: /e2e マーケット検索と表示フローをテスト

Agent (e2e-runner):
# E2Eテスト生成: マーケット検索と表示フロー

## 特定されたテストシナリオ

**ユーザージャーニー:** マーケット検索 → 結果表示 → マーケットクリック → 詳細表示

**テストカバレッジ:**
1. マーケットページへ移動
2. セマンティック検索を実行
3. 検索結果を検証
4. 最初の結果をクリック
5. マーケット詳細ページが読み込まれることを検証
6. チャートがレンダリングされることを検証

## 生成されたテストコード

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('Market Search and View Flow', () => {
  test('user can search markets and view details', async ({ page }) => {
    // 1. マーケットページへ移動
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // ページが読み込まれたことを検証
    await expect(page).toHaveTitle(/Markets/)
    await expect(page.locator('h1')).toContainText('Markets')

    // 2. セマンティック検索を実行
    await marketsPage.searchMarkets('election')

    // APIレスポンスを待機
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 3. 検索結果を検証
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // 検索結果のスクリーンショットを撮影
    await page.screenshot({ path: 'artifacts/search-results.png' })

    // 4. 最初の結果をクリック
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 5. マーケット詳細ページが読み込まれることを検証
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.marketDescription).toBeVisible()

    // 6. チャートがレンダリングされることを検証
    await expect(detailsPage.priceChart).toBeVisible()

    // マーケット名が一致することを検証
    const detailsTitle = await detailsPage.marketName.textContent()
    expect(detailsTitle?.toLowerCase()).toContain(
      firstMarketTitle?.toLowerCase().substring(0, 20) || ''
    )

    // マーケット詳細のスクリーンショットを撮影
    await page.screenshot({ path: 'artifacts/market-details.png' })
  })

  test('search with no results shows empty state', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 存在しないマーケットを検索
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // 空の状態を検証
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /no.*results|no.*markets/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('can clear search and see all markets again', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 初期のマーケット数
    const initialCount = await marketsPage.marketCards.count()

    // 検索を実行
    await marketsPage.searchMarkets('trump')
    await page.waitForLoadState('networkidle')

    // フィルタリングされた結果を検証
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // 検索をクリア
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // すべてのマーケットが再び表示されることを検証
    const finalCount = await marketsPage.marketCards.count()
    expect(finalCount).toBe(initialCount)
  })
})
```

## テストの実行

```bash
# 生成されたテストを実行
npx playwright test tests/e2e/markets/search-and-view.spec.ts

Running 3 tests using 3 workers

  ✓  [chromium] › search-and-view.spec.ts:5:3 › user can search markets and view details (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › search with no results shows empty state (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › can clear search and see all markets again (2.9s)

  3 passed (9.1s)

Artifacts generated:
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

## テストレポート

```
╔══════════════════════════════════════════════════════════════╗
║                    E2E Test Results                          ║
╠══════════════════════════════════════════════════════════════╣
║ Status:     ✅ ALL TESTS PASSED                              ║
║ Total:      3 tests                                          ║
║ Passed:     3 (100%)                                         ║
║ Failed:     0                                                ║
║ Flaky:      0                                                ║
║ Duration:   9.1s                                             ║
╚══════════════════════════════════════════════════════════════╝

Artifacts:
📸 Screenshots: 2 files
📹 Videos: 0 files (only on failure)
🔍 Traces: 0 files (only on failure)
📊 HTML Report: playwright-report/index.html

View report: npx playwright show-report
```

✅ E2Eテストスイートがci/cd統合の準備完了！
```

## テストアーティファクト

テスト実行時、以下のアーティファクトがキャプチャされます:

**すべてのテストで:**
- タイムラインと結果を含むHTMLレポート
- CI統合用のJUnit XML

**失敗時のみ:**
- 失敗状態のスクリーンショット
- テストのビデオ録画
- デバッグ用のトレースファイル（ステップバイステップのリプレイ）
- ネットワークログ
- コンソールログ

## アーティファクトの表示

```bash
# ブラウザでHTMLレポートを表示
npx playwright show-report

# 特定のトレースファイルを表示
npx playwright show-trace artifacts/trace-abc123.zip

# スクリーンショットはartifacts/ディレクトリに保存されます
open artifacts/search-results.png
```

## 不安定なテストの検出

テストが断続的に失敗する場合:

```
⚠️  FLAKY TEST DETECTED: tests/e2e/markets/trade.spec.ts

Test passed 7/10 runs (70% pass rate)

Common failure:
"Timeout waiting for element '[data-testid="confirm-btn"]'"

Recommended fixes:
1. Add explicit wait: await page.waitForSelector('[data-testid="confirm-btn"]')
2. Increase timeout: { timeout: 10000 }
3. Check for race conditions in component
4. Verify element is not hidden by animation

Quarantine recommendation: Mark as test.fixme() until fixed
```

## ブラウザ設定

テストはデフォルトで複数のブラウザで実行されます:
- ✅ Chromium（デスクトップChrome）
- ✅ Firefox（デスクトップ）
- ✅ WebKit（デスクトップSafari）
- ✅ モバイルChrome（オプション）

ブラウザの調整は `playwright.config.ts` で設定します。

## CI/CD統合

CIパイプラインに追加:

```yaml
# .github/workflows/e2e.yml
- name: Install Playwright
  run: npx playwright install --with-deps

- name: Run E2E tests
  run: npx playwright test

- name: Upload artifacts
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

## PMX固有の重要フロー

PMXでは、これらのE2Eテストを優先してください:

**🔴 CRITICAL（必ず合格する必要あり）:**
1. ユーザーがウォレットを接続できる
2. ユーザーがマーケットを閲覧できる
3. ユーザーがマーケットを検索できる（セマンティック検索）
4. ユーザーがマーケット詳細を表示できる
5. ユーザーが取引を行える（テスト資金で）
6. マーケットが正しく解決される
7. ユーザーが資金を引き出せる

**🟡 IMPORTANT:**
1. マーケット作成フロー
2. ユーザープロファイルの更新
3. リアルタイム価格更新
4. チャートレンダリング
5. マーケットのフィルタリングとソート
6. モバイルレスポンシブレイアウト

## ベストプラクティス

**すべきこと:**
- ✅ 保守性のためにPage Object Modelを使用
- ✅ セレクターにdata-testid属性を使用
- ✅ 任意のタイムアウトではなくAPIレスポンスを待機
- ✅ 重要なユーザージャーニーをエンドツーエンドでテスト
- ✅ mainへのマージ前にテストを実行
- ✅ テスト失敗時にアーティファクトを確認

**すべきでないこと:**
- ❌ 脆弱なセレクターを使用（CSSクラスは変更される可能性あり）
- ❌ 実装の詳細をテスト
- ❌ 本番環境に対してテストを実行
- ❌ 不安定なテストを無視
- ❌ 失敗時のアーティファクトレビューをスキップ
- ❌ すべてのエッジケースをE2Eでテスト（ユニットテストを使用）

## 重要な注意事項

**PMXにとって重要:**
- 実際のお金を伴うE2Eテストはテストネット/ステージングのみで実行する必要があります
- 本番環境に対して取引テストを実行しないでください
- 金融テストには `test.skip(process.env.NODE_ENV === 'production')` を設定
- 少額のテスト資金のみを持つテストウォレットを使用

## 他のコマンドとの統合

- `/plan` を使用してテストすべき重要なジャーニーを特定
- `/tdd` をユニットテストに使用（より高速で詳細）
- `/e2e` を統合テストとユーザージャーニーテストに使用
- `/code-review` を使用してテスト品質を検証

## 関連エージェント

このコマンドは以下の場所にある `e2e-runner` エージェントを呼び出します:
`~/.claude/agents/e2e-runner.md`

## クイックコマンド

```bash
# すべてのE2Eテストを実行
npx playwright test

# 特定のテストファイルを実行
npx playwright test tests/e2e/markets/search.spec.ts

# ヘッドモードで実行（ブラウザを表示）
npx playwright test --headed

# テストをデバッグ
npx playwright test --debug

# テストコードを生成
npx playwright codegen http://localhost:3000

# レポートを表示
npx playwright show-report
```
