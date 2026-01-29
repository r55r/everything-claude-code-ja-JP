---
name: eval-harness
description: Formal evaluation framework for Claude Code sessions implementing eval-driven development (EDD) principles
tools: Read, Write, Edit, Bash, Grep, Glob
---

# Evalハーネススキル

評価駆動開発（EDD）の原則を実装するClaude Codeセッション用の正式な評価フレームワーク。

## 哲学

評価駆動開発はevalsを「AI開発のユニットテスト」として扱います:
- 実装の前に期待される動作を定義
- 開発中に継続的にevalsを実行
- 各変更でのリグレッションを追跡
- 信頼性測定にpass@kメトリクスを使用

## Evalタイプ

### 能力Eval
以前できなかったことをClaudeができるかテスト:
```markdown
[CAPABILITY EVAL: feature-name]
Task: Claudeが達成すべきことの説明
Success Criteria:
  - [ ] 基準1
  - [ ] 基準2
  - [ ] 基準3
Expected Output: 期待される結果の説明
```

### リグレッションEval
変更が既存の機能を壊さないことを確認:
```markdown
[REGRESSION EVAL: feature-name]
Baseline: SHAまたはチェックポイント名
Tests:
  - existing-test-1: PASS/FAIL
  - existing-test-2: PASS/FAIL
  - existing-test-3: PASS/FAIL
Result: X/Y passed (以前はY/Y)
```

## グレーダータイプ

### 1. コードベースグレーダー
コードを使用した決定論的チェック:
```bash
# ファイルに期待されるパターンが含まれているか確認
grep -q "export function handleAuth" src/auth.ts && echo "PASS" || echo "FAIL"

# テストがパスするか確認
npm test -- --testPathPattern="auth" && echo "PASS" || echo "FAIL"

# ビルドが成功するか確認
npm run build && echo "PASS" || echo "FAIL"
```

### 2. モデルベースグレーダー
オープンエンドな出力を評価するためにClaudeを使用:
```markdown
[MODEL GRADER PROMPT]
以下のコード変更を評価してください:
1. 指定された問題を解決していますか？
2. 構造は適切ですか？
3. エッジケースは処理されていますか？
4. エラーハンドリングは適切ですか？

Score: 1-5 (1=悪い、5=優秀)
Reasoning: [説明]
```

### 3. 人間グレーダー
手動レビュー用にフラグ:
```markdown
[HUMAN REVIEW REQUIRED]
Change: 変更内容の説明
Reason: 人間のレビューが必要な理由
Risk Level: LOW/MEDIUM/HIGH
```

## メトリクス

### pass@k
「k回の試行で少なくとも1回成功」
- pass@1: 初回試行の成功率
- pass@3: 3回以内の成功
- 典型的な目標: pass@3 > 90%

### pass^k
「k回すべての試行が成功」
- 信頼性のより高い基準
- pass^3: 3回連続成功
- クリティカルパスに使用

## Evalワークフロー

### 1. 定義（コーディング前）
```markdown
## EVAL DEFINITION: feature-xyz

### 能力Eval
1. 新しいユーザーアカウントを作成できる
2. メール形式を検証できる
3. パスワードを安全にハッシュ化できる

### リグレッションEval
1. 既存のログインが引き続き動作する
2. セッション管理が変更されていない
3. ログアウトフローがそのまま

### 成功メトリクス
- 能力evalsでpass@3 > 90%
- リグレッションevalsでpass^3 = 100%
```

### 2. 実装
定義されたevalsをパスするためのコードを書く。

### 3. 評価
```bash
# 能力evalsを実行
[各能力evalを実行し、PASS/FAILを記録]

# リグレッションevalsを実行
npm test -- --testPathPattern="existing"

# レポートを生成
```

### 4. レポート
```markdown
EVAL REPORT: feature-xyz
========================

能力Eval:
  create-user:     PASS (pass@1)
  validate-email:  PASS (pass@2)
  hash-password:   PASS (pass@1)
  Overall:         3/3 passed

リグレッションEval:
  login-flow:      PASS
  session-mgmt:    PASS
  logout-flow:     PASS
  Overall:         3/3 passed

メトリクス:
  pass@1: 67% (2/3)
  pass@3: 100% (3/3)

Status: READY FOR REVIEW
```

## 統合パターン

### 実装前
```
/eval define feature-name
```
`.claude/evals/feature-name.md` にeval定義ファイルを作成

### 実装中
```
/eval check feature-name
```
現在のevalsを実行してステータスをレポート

### 実装後
```
/eval report feature-name
```
完全なevalレポートを生成

## Evalストレージ

プロジェクト内にevalsを保存:
```
.claude/
  evals/
    feature-xyz.md      # Eval定義
    feature-xyz.log     # Eval実行履歴
    baseline.json       # リグレッションベースライン
```

## ベストプラクティス

1. **コーディング前にevalsを定義** - 成功基準についての明確な思考を強制
2. **頻繁にevalsを実行** - リグレッションを早期に発見
3. **pass@kを時系列で追跡** - 信頼性の傾向を監視
4. **可能な限りコードグレーダーを使用** - 決定論的 > 確率的
5. **セキュリティには人間のレビュー** - セキュリティチェックを完全に自動化しない
6. **evalsを高速に保つ** - 遅いevalsは実行されない
7. **evalsをコードと一緒にバージョン管理** - evalsは第一級成果物

## 例: 認証の追加

```markdown
## EVAL: add-authentication

### フェーズ1: 定義（10分）
能力Eval:
- [ ] ユーザーがメール/パスワードで登録できる
- [ ] ユーザーが有効な資格情報でログインできる
- [ ] 無効な資格情報が適切なエラーで拒否される
- [ ] セッションがページリロード後も永続化される
- [ ] ログアウトでセッションがクリアされる

リグレッションEval:
- [ ] 公開ルートが引き続きアクセス可能
- [ ] APIレスポンスが変更されていない
- [ ] データベーススキーマが互換性を保持

### フェーズ2: 実装（可変）
[コードを書く]

### フェーズ3: 評価
実行: /eval check add-authentication

### フェーズ4: レポート
EVAL REPORT: add-authentication
==============================
能力: 5/5 passed (pass@3: 100%)
リグレッション: 3/3 passed (pass^3: 100%)
Status: SHIP IT
```
