---
name: evolve
description: Cluster related instincts into skills, commands, or agents
command: /evolve
implementation: python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve
---

# Evolve Command

## 実装

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve [--generate]
```

インスティンクトを分析し、関連するものを上位レベルの構造にクラスタリングします:
- **Commands**: インスティンクトがユーザーから呼び出されるアクションを記述する場合
- **Skills**: インスティンクトが自動トリガーされる動作を記述する場合
- **Agents**: インスティンクトが複雑なマルチステッププロセスを記述する場合

## 使用方法

```
/evolve                    # すべてのインスティンクトを分析し、進化を提案
/evolve --domain testing   # testingドメインのインスティンクトのみを進化
/evolve --dry-run          # 作成せずに何が作成されるか表示
/evolve --threshold 5      # クラスタリングに5つ以上の関連インスティンクトを必要とする
```

## 進化ルール

### → Command（ユーザー呼び出し）
インスティンクトがユーザーが明示的に要求するアクションを記述する場合:
- 「ユーザーが...を要求したとき」に関する複数のインスティンクト
- 「新しいXを作成するとき」のようなトリガーを持つインスティンクト
- 繰り返し可能なシーケンスに従うインスティンクト

例:
- `new-table-step1`: 「データベーステーブルを追加するとき、マイグレーションを作成」
- `new-table-step2`: 「データベーステーブルを追加するとき、スキーマを更新」
- `new-table-step3`: 「データベーステーブルを追加するとき、型を再生成」

→ 作成されるもの: `/new-table` コマンド

### → Skill（自動トリガー）
インスティンクトが自動的に発生すべき動作を記述する場合:
- パターンマッチングトリガー
- エラーハンドリング応答
- コードスタイルの強制

例:
- `prefer-functional`: 「関数を書くとき、関数型スタイルを優先」
- `use-immutable`: 「状態を変更するとき、イミュータブルパターンを使用」
- `avoid-classes`: 「モジュールを設計するとき、クラスベースの設計を避ける」

→ 作成されるもの: `functional-patterns` スキル

### → Agent（深さ/分離が必要）
インスティンクトが分離が有益な複雑なマルチステッププロセスを記述する場合:
- デバッグワークフロー
- リファクタリングシーケンス
- リサーチタスク

例:
- `debug-step1`: 「デバッグするとき、まずログを確認」
- `debug-step2`: 「デバッグするとき、失敗しているコンポーネントを分離」
- `debug-step3`: 「デバッグするとき、最小限の再現を作成」
- `debug-step4`: 「デバッグするとき、テストで修正を検証」

→ 作成されるもの: `debugger` エージェント

## 実行内容

1. `~/.claude/homunculus/instincts/` からすべてのインスティンクトを読み込み
2. インスティンクトを以下でグループ化:
   - ドメインの類似性
   - トリガーパターンの重複
   - アクションシーケンスの関係
3. 3つ以上の関連インスティンクトの各クラスターに対して:
   - 進化タイプを決定（command/skill/agent）
   - 適切なファイルを生成
   - `~/.claude/homunculus/evolved/{commands,skills,agents}/` に保存
4. 進化した構造をソースインスティンクトにリンク

## 出力フォーマット

```
🧬 Evolve Analysis
==================

Found 3 clusters ready for evolution:

## Cluster 1: Database Migration Workflow
Instincts: new-table-migration, update-schema, regenerate-types
Type: Command
Confidence: 85% (based on 12 observations)

Would create: /new-table command
Files:
  - ~/.claude/homunculus/evolved/commands/new-table.md

## Cluster 2: Functional Code Style
Instincts: prefer-functional, use-immutable, avoid-classes, pure-functions
Type: Skill
Confidence: 78% (based on 8 observations)

Would create: functional-patterns skill
Files:
  - ~/.claude/homunculus/evolved/skills/functional-patterns.md

## Cluster 3: Debugging Process
Instincts: debug-check-logs, debug-isolate, debug-reproduce, debug-verify
Type: Agent
Confidence: 72% (based on 6 observations)

Would create: debugger agent
Files:
  - ~/.claude/homunculus/evolved/agents/debugger.md

---
Run `/evolve --execute` to create these files.
```

## フラグ

- `--execute`: 実際に進化した構造を作成（デフォルトはプレビュー）
- `--dry-run`: 作成せずにプレビュー
- `--domain <name>`: 指定されたドメインのインスティンクトのみを進化
- `--threshold <n>`: クラスターを形成するために必要な最小インスティンクト数（デフォルト: 3）
- `--type <command|skill|agent>`: 指定されたタイプのみを作成

## 生成されるファイルフォーマット

### Command
```markdown
---
name: new-table
description: Create a new database table with migration, schema update, and type generation
command: /new-table
evolved_from:
  - new-table-migration
  - update-schema
  - regenerate-types
---

# New Table Command

[クラスタリングされたインスティンクトに基づいて生成されたコンテンツ]

## Steps
1. ...
2. ...
```

### Skill
```markdown
---
name: functional-patterns
description: Enforce functional programming patterns
evolved_from:
  - prefer-functional
  - use-immutable
  - avoid-classes
---

# Functional Patterns Skill

[クラスタリングされたインスティンクトに基づいて生成されたコンテンツ]
```

### Agent
```markdown
---
name: debugger
description: Systematic debugging agent
model: sonnet
evolved_from:
  - debug-check-logs
  - debug-isolate
  - debug-reproduce
---

# Debugger Agent

[クラスタリングされたインスティンクトに基づいて生成されたコンテンツ]
```
