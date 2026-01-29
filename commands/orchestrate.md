# オーケストレートコマンド

複雑なタスクのための順次エージェントワークフロー。

## 使用方法

`/orchestrate [workflow-type] [task-description]`

## ワークフロータイプ

### feature
完全な機能実装ワークフロー：
```
planner -> tdd-guide -> code-reviewer -> security-reviewer
```

### bugfix
バグ調査と修正ワークフロー：
```
explorer -> tdd-guide -> code-reviewer
```

### refactor
安全なリファクタリングワークフロー：
```
architect -> code-reviewer -> tdd-guide
```

### security
セキュリティ重視のレビュー：
```
security-reviewer -> code-reviewer -> architect
```

## 実行パターン

ワークフロー内の各エージェントに対して：

1. **エージェントを呼び出す** - 前のエージェントからのコンテキストを使用
2. **出力を収集** - 構造化された引き継ぎドキュメントとして
3. **次のエージェントに渡す** - チェーン内で
4. **結果を集約** - 最終レポートにまとめる

## 引き継ぎドキュメントフォーマット

エージェント間で引き継ぎドキュメントを作成：

```markdown
## HANDOFF: [previous-agent] -> [next-agent]

### コンテキスト
[何が行われたかの要約]

### 発見事項
[主要な発見や決定]

### 変更されたファイル
[操作したファイルのリスト]

### 未解決の質問
[次のエージェントへの未解決項目]

### 推奨事項
[提案される次のステップ]
```

## 例: 機能ワークフロー

```
/orchestrate feature "ユーザー認証を追加"
```

実行内容：

1. **プランナーエージェント**
   - 要件を分析
   - 実装計画を作成
   - 依存関係を特定
   - 出力: `HANDOFF: planner -> tdd-guide`

2. **TDDガイドエージェント**
   - プランナーの引き継ぎを読む
   - 最初にテストを書く
   - テストが通るように実装
   - 出力: `HANDOFF: tdd-guide -> code-reviewer`

3. **コードレビューアーエージェント**
   - 実装をレビュー
   - 問題をチェック
   - 改善を提案
   - 出力: `HANDOFF: code-reviewer -> security-reviewer`

4. **セキュリティレビューアーエージェント**
   - セキュリティ監査
   - 脆弱性チェック
   - 最終承認
   - 出力: 最終レポート

## 最終レポートフォーマット

```
オーケストレーションレポート
====================
ワークフロー: feature
タスク: ユーザー認証を追加
エージェント: planner -> tdd-guide -> code-reviewer -> security-reviewer

サマリー
-------
[1段落の要約]

エージェント出力
-------------
プランナー: [要約]
TDDガイド: [要約]
コードレビューアー: [要約]
セキュリティレビューアー: [要約]

変更ファイル
-------------
[変更されたすべてのファイルのリスト]

テスト結果
------------
[テスト成功/失敗の要約]

セキュリティステータス
---------------
[セキュリティ発見事項]

推奨
--------------
[SHIP / NEEDS WORK / BLOCKED]
```

## 並列実行

独立したチェックの場合、エージェントを並列で実行：

```markdown
### 並列フェーズ
同時に実行:
- code-reviewer (品質)
- security-reviewer (セキュリティ)
- architect (設計)

### 結果をマージ
出力を単一のレポートに統合
```

## 引数

$ARGUMENTS:
- `feature <description>` - 完全な機能ワークフロー
- `bugfix <description>` - バグ修正ワークフロー
- `refactor <description>` - リファクタリングワークフロー
- `security <description>` - セキュリティレビューワークフロー
- `custom <agents> <description>` - カスタムエージェントシーケンス

## カスタムワークフローの例

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "キャッシュレイヤーを再設計"
```

## ヒント

1. **複雑な機能にはplannerから始める**
2. **マージ前には必ずcode-reviewerを含める**
3. **認証/決済/個人情報にはsecurity-reviewerを使用**
4. **引き継ぎは簡潔に** - 次のエージェントが必要なものに焦点を当てる
5. **必要に応じてエージェント間で検証を実行**
