---
description: イディオマティックなパターン、並行処理の安全性、エラーハンドリング、セキュリティに関する包括的なGoコードレビュー。go-reviewerエージェントを呼び出します。
---

# Go Code Review

このコマンドは、Go固有の包括的なコードレビューのために**go-reviewer**エージェントを呼び出します。

## このコマンドの機能

1. **Go変更の特定**: `git diff` で変更された `.go` ファイルを検出
2. **静的解析の実行**: `go vet`、`staticcheck`、`golangci-lint` を実行
3. **セキュリティスキャン**: SQLインジェクション、コマンドインジェクション、競合状態をチェック
4. **並行処理レビュー**: goroutineの安全性、チャネルの使用、mutexパターンを分析
5. **イディオマティックGoチェック**: コードがGoの規約とベストプラクティスに従っているか検証
6. **レポート生成**: 重要度別に問題を分類

## 使用タイミング

以下の場合に `/go-review` を使用:
- Goコードを書いたり変更した後
- Go変更をコミットする前
- Goコードを含むプルリクエストをレビューする際
- 新しいGoコードベースにオンボーディングする際
- イディオマティックなGoパターンを学ぶ際

## レビューカテゴリ

### CRITICAL（必ず修正）
- SQL/コマンドインジェクションの脆弱性
- 同期なしの競合状態
- Goroutineリーク
- ハードコードされた認証情報
- 安全でないポインタ使用
- 重要なパスでの無視されたエラー

### HIGH（修正すべき）
- コンテキストなしのエラー返却
- エラー返却の代わりにpanic
- Contextが伝播されていない
- デッドロックを引き起こすバッファなしチャネル
- インターフェース未実装エラー
- Mutex保護の欠如

### MEDIUM（検討）
- 非イディオマティックなコードパターン
- エクスポートされた項目にgodocコメントがない
- 非効率な文字列結合
- スライスの事前割り当てがない
- テーブル駆動テストが使用されていない

## 実行される自動チェック

```bash
# 静的解析
go vet ./...

# 高度なチェック（インストールされている場合）
staticcheck ./...
golangci-lint run

# 競合検出
go build -race ./...

# セキュリティ脆弱性
govulncheck ./...
```

## 使用例

```text
User: /go-review

Agent:
# Go Code Review Report

## レビューされたファイル
- internal/handler/user.go (modified)
- internal/service/auth.go (modified)

## 静的解析結果
✓ go vet: 問題なし
✓ staticcheck: 問題なし

## 検出された問題

[CRITICAL] 競合状態
File: internal/service/auth.go:45
Issue: 同期なしで共有マップにアクセス
```go
var cache = map[string]*Session{}  // 並行アクセス！

func GetSession(id string) *Session {
    return cache[id]  // 競合状態
}
```
Fix: sync.RWMutex または sync.Map を使用
```go
var (
    cache   = map[string]*Session{}
    cacheMu sync.RWMutex
)

func GetSession(id string) *Session {
    cacheMu.RLock()
    defer cacheMu.RUnlock()
    return cache[id]
}
```

[HIGH] エラーコンテキストの欠如
File: internal/handler/user.go:28
Issue: コンテキストなしでエラーを返却
```go
return err  // コンテキストなし
```
Fix: コンテキストでラップ
```go
return fmt.Errorf("get user %s: %w", userID, err)
```

## サマリー
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 0

Recommendation: ❌ CRITICAL問題が修正されるまでマージをブロック
```

## 承認基準

| ステータス | 条件 |
|--------|-----------|
| ✅ Approve | CRITICALまたはHIGH問題なし |
| ⚠️ Warning | MEDIUM問題のみ（注意してマージ） |
| ❌ Block | CRITICALまたはHIGH問題が検出された |

## 他のコマンドとの統合

- まず `/go-test` を使用してテストが合格することを確認
- ビルドエラーが発生した場合は `/go-build` を使用
- コミット前に `/go-review` を使用
- Go固有でない懸念事項には `/code-review` を使用

## 関連

- Agent: `agents/go-reviewer.md`
- Skills: `skills/golang-patterns/`, `skills/golang-testing/`
