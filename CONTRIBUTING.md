# Everything Claude Code への貢献

貢献にご興味をお持ちいただきありがとうございます。このリポジトリは Claude Code ユーザーのためのコミュニティリソースとなることを目指しています。

## 募集している内容

### エージェント

特定のタスクを適切に処理する新しいエージェント:
- 言語固有のレビュアー（Python、Go、Rust）
- フレームワークエキスパート（Django、Rails、Laravel、Spring）
- DevOps スペシャリスト（Kubernetes、Terraform、CI/CD）
- ドメインエキスパート（MLパイプライン、データエンジニアリング、モバイル）

### スキル

ワークフロー定義とドメイン知識:
- 言語のベストプラクティス
- フレームワークパターン
- テスト戦略
- アーキテクチャガイド
- ドメイン固有の知識

### コマンド

便利なワークフローを呼び出すスラッシュコマンド:
- デプロイメントコマンド
- テストコマンド
- ドキュメントコマンド
- コード生成コマンド

### フック

便利な自動化:
- リント/フォーマットフック
- セキュリティチェック
- バリデーションフック
- 通知フック

### ルール

常に従うべきガイドライン:
- セキュリティルール
- コードスタイルルール
- テスト要件
- 命名規則

### MCP 設定

新規または改善された MCP サーバー設定:
- データベース連携
- クラウドプロバイダー MCP
- 監視ツール
- コミュニケーションツール

---

## 貢献方法

### 1. リポジトリをフォーク

```bash
git clone https://github.com/YOUR_USERNAME/everything-claude-code.git
cd everything-claude-code
```

### 2. ブランチを作成

```bash
git checkout -b add-python-reviewer
```

### 3. 貢献を追加

ファイルを適切なディレクトリに配置してください:
- `agents/` 新しいエージェント用
- `skills/` スキル用（単一の .md またはディレクトリ）
- `commands/` スラッシュコマンド用
- `rules/` ルールファイル用
- `hooks/` フック設定用
- `mcp-configs/` MCP サーバー設定用

### 4. フォーマットに従う

**エージェント**にはフロントマターが必要です:

```markdown
---
name: agent-name
description: What it does
tools: Read, Grep, Glob, Bash
model: sonnet
---

Instructions here...
```

**スキル**は明確で実行可能であるべきです:

```markdown
# Skill Name

## When to Use

...

## How It Works

...

## Examples

...
```

**コマンド**は何をするか説明してください:

```markdown
---
description: Brief description of command
---

# Command Name

Detailed instructions...
```

**フック**には説明を含めてください:

```json
{
  "matcher": "...",
  "hooks": [...],
  "description": "What this hook does"
}
```

### 5. 貢献をテスト

提出前に Claude Code で設定が動作することを確認してください。

### 6. PR を提出

```bash
git add .
git commit -m "Add Python code reviewer agent"
git push origin add-python-reviewer
```

次に、以下の内容で PR を開いてください:
- 何を追加したか
- なぜ便利なのか
- どのようにテストしたか

---

## ガイドライン

### すべきこと

- 設定は焦点を絞りモジュール化する
- 明確な説明を含める
- 提出前にテストする
- 既存のパターンに従う
- 依存関係をドキュメント化する

### すべきでないこと

- 機密データを含める（APIキー、トークン、パス）
- 過度に複雑またはニッチな設定を追加
- テストされていない設定を提出
- 重複する機能を作成
- 代替手段なしに特定の有料サービスを必要とする設定を追加

---

## ファイル命名規則

- 小文字とハイフンを使用: `python-reviewer.md`
- 説明的に: `tdd-workflow.md`（`workflow.md` ではなく）
- エージェント/スキル名とファイル名を一致させる

---

## 質問がありますか？

Issue を開くか、X で連絡してください: [@affaanmustafa](https://x.com/affaanmustafa)

---

貢献いただきありがとうございます。一緒に素晴らしいリソースを作りましょう。
