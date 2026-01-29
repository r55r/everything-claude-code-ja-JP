---
description: GoのTDDワークフローを強制します。まずテーブル駆動テストを書き、その後実装します。go test -coverで80%以上のカバレッジを検証します。
---

# Go TDD Command

このコマンドは、イディオマティックなGoテストパターンを使用したGoコードのテスト駆動開発方法論を強制します。

## このコマンドの機能

1. **型/インターフェースの定義**: まず関数シグネチャをスキャフォールド
2. **テーブル駆動テストの作成**: 包括的なテストケースを作成（RED）
3. **テストの実行**: テストが正しい理由で失敗することを検証
4. **コードの実装**: 合格するための最小限のコードを記述（GREEN）
5. **リファクタリング**: テストがグリーンのまま改善
6. **カバレッジチェック**: 80%以上のカバレッジを確保

## 使用タイミング

以下の場合に `/go-test` を使用:
- 新しいGo関数を実装する際
- 既存コードにテストカバレッジを追加する際
- バグを修正する際（まず失敗するテストを書く）
- 重要なビジネスロジックを構築する際
- GoでのTDDワークフローを学ぶ際

## TDDサイクル

```
RED     → 失敗するテーブル駆動テストを書く
GREEN   → 合格するための最小限のコードを実装
REFACTOR → コードを改善、テストはグリーンのまま
REPEAT  → 次のテストケースへ
```

## セッション例

```text
User: /go-test メールアドレスを検証する関数が必要です

Agent:
# TDDセッション: Email Validator

## ステップ 1: インターフェースの定義

```go
// validator/email.go
package validator

// ValidateEmail は指定された文字列が有効なメールアドレスかどうかを確認します。
// 有効な場合はnilを返し、そうでない場合は問題を説明するエラーを返します。
func ValidateEmail(email string) error {
    panic("not implemented")
}
```

## ステップ 2: テーブル駆動テストの作成（RED）

```go
// validator/email_test.go
package validator

import (
    "testing"
)

func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        // 有効なメール
        {"simple email", "user@example.com", false},
        {"with subdomain", "user@mail.example.com", false},
        {"with plus", "user+tag@example.com", false},
        {"with dots", "first.last@example.com", false},

        // 無効なメール
        {"empty string", "", true},
        {"no at sign", "userexample.com", true},
        {"no domain", "user@", true},
        {"no local part", "@example.com", true},
        {"double at", "user@@example.com", true},
        {"spaces", "user @example.com", true},
        {"no tld", "user@example", true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.email)
            if tt.wantErr && err == nil {
                t.Errorf("ValidateEmail(%q) = nil; want error", tt.email)
            }
            if !tt.wantErr && err != nil {
                t.Errorf("ValidateEmail(%q) = %v; want nil", tt.email, err)
            }
        })
    }
}
```

## ステップ 3: テスト実行 - FAILを確認

```bash
$ go test ./validator/...

--- FAIL: TestValidateEmail (0.00s)
    --- FAIL: TestValidateEmail/simple_email (0.00s)
        panic: not implemented

FAIL
```

✓ テストは予想通り失敗（panic）。

## ステップ 4: 最小限のコードを実装（GREEN）

```go
// validator/email.go
package validator

import (
    "errors"
    "regexp"
)

var emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)

var (
    ErrEmailEmpty   = errors.New("email cannot be empty")
    ErrEmailInvalid = errors.New("email format is invalid")
)

func ValidateEmail(email string) error {
    if email == "" {
        return ErrEmailEmpty
    }
    if !emailRegex.MatchString(email) {
        return ErrEmailInvalid
    }
    return nil
}
```

## ステップ 5: テスト実行 - PASSを確認

```bash
$ go test ./validator/...

PASS
ok      project/validator    0.003s
```

✓ すべてのテストが合格！

## ステップ 6: カバレッジチェック

```bash
$ go test -cover ./validator/...

PASS
coverage: 100.0% of statements
ok      project/validator    0.003s
```

✓ カバレッジ: 100%

## TDD完了！
```

## テストパターン

### テーブル駆動テスト
```go
tests := []struct {
    name     string
    input    InputType
    want     OutputType
    wantErr  bool
}{
    {"case 1", input1, want1, false},
    {"case 2", input2, want2, true},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := Function(tt.input)
        // アサーション
    })
}
```

### 並列テスト
```go
for _, tt := range tests {
    tt := tt // キャプチャ
    t.Run(tt.name, func(t *testing.T) {
        t.Parallel()
        // テスト本体
    })
}
```

### テストヘルパー
```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db := createDB()
    t.Cleanup(func() { db.Close() })
    return db
}
```

## カバレッジコマンド

```bash
# 基本的なカバレッジ
go test -cover ./...

# カバレッジプロファイル
go test -coverprofile=coverage.out ./...

# ブラウザで表示
go tool cover -html=coverage.out

# 関数別カバレッジ
go tool cover -func=coverage.out

# 競合検出付き
go test -race -cover ./...
```

## カバレッジ目標

| コードタイプ | 目標 |
|-----------|--------|
| 重要なビジネスロジック | 100% |
| パブリックAPI | 90%以上 |
| 一般的なコード | 80%以上 |
| 生成されたコード | 除外 |

## TDDベストプラクティス

**すべきこと:**
- 実装の前にまずテストを書く
- 各変更後にテストを実行
- 包括的なカバレッジのためにテーブル駆動テストを使用
- 実装の詳細ではなく動作をテスト
- エッジケースを含める（空、nil、最大値）

**すべきでないこと:**
- テストの前に実装を書く
- REDフェーズをスキップ
- プライベート関数を直接テスト
- テストで `time.Sleep` を使用
- 不安定なテストを無視

## 関連コマンド

- `/go-build` - ビルドエラーを修正
- `/go-review` - 実装後にコードをレビュー
- `/verify` - 完全な検証ループを実行

## 関連

- Skill: `skills/golang-testing/`
- Skill: `skills/tdd-workflow/`
