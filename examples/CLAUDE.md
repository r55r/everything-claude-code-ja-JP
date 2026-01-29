# プロジェクト CLAUDE.md の例

これはプロジェクトレベルの CLAUDE.md ファイルの例です。プロジェクトのルートに配置してください。

## プロジェクト概要

[プロジェクトの簡単な説明 - 何をするか、技術スタック]

## 重要なルール

### 1. コードの構成

- 少数の大きなファイルより多数の小さなファイル
- 高凝集・疎結合
- 通常200〜400行、最大800行/ファイル
- タイプ別ではなく機能/ドメイン別に整理

### 2. コードスタイル

- コード、コメント、ドキュメントに絵文字を使用しない
- 常にイミュータビリティ - オブジェクトや配列を変更しない
- 本番コードに console.log を使用しない
- try/catch による適切なエラーハンドリング
- Zod などによる入力バリデーション

### 3. テスト

- TDD: テストを先に書く
- 最低80%のカバレッジ
- ユーティリティにはユニットテスト
- APIにはインテグレーションテスト
- 重要なフローにはE2Eテスト

### 4. セキュリティ

- ハードコードされたシークレットは禁止
- 機密データには環境変数を使用
- すべてのユーザー入力をバリデーション
- パラメータ化クエリのみ使用
- CSRF保護を有効化

## ファイル構成

```
src/
|-- app/              # Next.js app router
|-- components/       # 再利用可能なUIコンポーネント
|-- hooks/            # カスタムReactフック
|-- lib/              # ユーティリティライブラリ
|-- types/            # TypeScript型定義
```

## 主要パターン

### APIレスポンス形式

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
}
```

### エラーハンドリング

```typescript
try {
  const result = await operation()
  return { success: true, data: result }
} catch (error) {
  console.error('Operation failed:', error)
  return { success: false, error: 'User-friendly message' }
}
```

## 環境変数

```bash
# 必須
DATABASE_URL=
API_KEY=

# オプション
DEBUG=false
```

## 利用可能なコマンド

- `/tdd` - テスト駆動開発ワークフロー
- `/plan` - 実装計画の作成
- `/code-review` - コード品質レビュー
- `/build-fix` - ビルドエラーの修正

## Gitワークフロー

- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- mainに直接コミットしない
- PRにはレビューが必要
- マージ前にすべてのテストをパス
