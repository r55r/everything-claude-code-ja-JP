---
name: continuous-learning-v2
description: Instinct-based learning system that observes sessions via hooks, creates atomic instincts with confidence scoring, and evolves them into skills/commands/agents.
version: 2.0.0
---

# Continuous Learning v2 - 本能ベースアーキテクチャ

Claude Code セッションを、原子的な「本能」- 信頼度スコアリングを持つ小さな学習済み行動 - を通じて再利用可能な知識に変換する高度な学習システムです。

## v2 の新機能

| 機能 | v1 | v2 |
|---------|----|----|
| 観察 | Stop hook（セッション終了時） | PreToolUse/PostToolUse（100%信頼性） |
| 分析 | メインコンテキスト | バックグラウンドエージェント（Haiku） |
| 粒度 | 完全なスキル | 原子的な「本能」 |
| 信頼度 | なし | 0.3〜0.9 重み付け |
| 進化 | スキルへ直接 | 本能 → クラスタ → スキル/コマンド/エージェント |
| 共有 | なし | 本能のエクスポート/インポート |

## 本能モデル

本能は小さな学習済み行動です:

```yaml
---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.7
domain: "code-style"
source: "session-observation"
---

# Prefer Functional Style

## Action
Use functional patterns over classes when appropriate.

## Evidence
- Observed 5 instances of functional pattern preference
- User corrected class-based approach to functional on 2025-01-15
```

**プロパティ:**
- **原子的** - 1つのトリガー、1つのアクション
- **信頼度重み付け** - 0.3 = 暫定的、0.9 = ほぼ確実
- **ドメインタグ付け** - code-style、testing、git、debugging、workflow など
- **エビデンス裏付け** - 作成の元となった観察を追跡

## 仕組み

```
セッションアクティビティ
      │
      │ Hooks がプロンプト + ツール使用をキャプチャ（100%信頼性）
      ▼
┌─────────────────────────────────────────┐
│         observations.jsonl              │
│   （プロンプト、ツール呼び出し、結果）    │
└─────────────────────────────────────────┘
      │
      │ Observer エージェントが読み取り（バックグラウンド、Haiku）
      ▼
┌─────────────────────────────────────────┐
│          パターン検出                    │
│   • ユーザー修正 → 本能                  │
│   • エラー解決 → 本能                    │
│   • 繰り返しワークフロー → 本能          │
└─────────────────────────────────────────┘
      │
      │ 作成/更新
      ▼
┌─────────────────────────────────────────┐
│         instincts/personal/             │
│   • prefer-functional.md (0.7)          │
│   • always-test-first.md (0.9)          │
│   • use-zod-validation.md (0.6)         │
└─────────────────────────────────────────┘
      │
      │ /evolve がクラスタリング
      ▼
┌─────────────────────────────────────────┐
│              evolved/                   │
│   • commands/new-feature.md             │
│   • skills/testing-workflow.md          │
│   • agents/refactor-specialist.md       │
└─────────────────────────────────────────┘
```

## クイックスタート

### 1. 観察 Hooks を有効化

`~/.claude/settings.json` に追加:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh pre"
      }]
    }],
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh post"
      }]
    }]
  }
}
```

### 2. ディレクトリ構造を初期化

```bash
mkdir -p ~/.claude/homunculus/{instincts/{personal,inherited},evolved/{agents,skills,commands}}
touch ~/.claude/homunculus/observations.jsonl
```

### 3. Observer エージェントを実行（オプション）

Observer はバックグラウンドで観察を分析できます:

```bash
# バックグラウンド Observer を開始
~/.claude/skills/continuous-learning-v2/agents/start-observer.sh
```

## コマンド

| コマンド | 説明 |
|---------|-------------|
| `/instinct-status` | 学習済み本能と信頼度を表示 |
| `/evolve` | 関連する本能をスキル/コマンドにクラスタリング |
| `/instinct-export` | 共有用に本能をエクスポート |
| `/instinct-import <file>` | 他者から本能をインポート |

## 設定

`config.json` を編集:

```json
{
  "version": "2.0",
  "observation": {
    "enabled": true,
    "store_path": "~/.claude/homunculus/observations.jsonl",
    "max_file_size_mb": 10,
    "archive_after_days": 7
  },
  "instincts": {
    "personal_path": "~/.claude/homunculus/instincts/personal/",
    "inherited_path": "~/.claude/homunculus/instincts/inherited/",
    "min_confidence": 0.3,
    "auto_approve_threshold": 0.7,
    "confidence_decay_rate": 0.05
  },
  "observer": {
    "enabled": true,
    "model": "haiku",
    "run_interval_minutes": 5,
    "patterns_to_detect": [
      "user_corrections",
      "error_resolutions",
      "repeated_workflows",
      "tool_preferences"
    ]
  },
  "evolution": {
    "cluster_threshold": 3,
    "evolved_path": "~/.claude/homunculus/evolved/"
  }
}
```

## ファイル構造

```
~/.claude/homunculus/
├── identity.json           # プロファイル、技術レベル
├── observations.jsonl      # 現在のセッション観察
├── observations.archive/   # 処理済み観察
├── instincts/
│   ├── personal/           # 自動学習された本能
│   └── inherited/          # 他者からインポート
└── evolved/
    ├── agents/             # 生成された専門エージェント
    ├── skills/             # 生成されたスキル
    └── commands/           # 生成されたコマンド
```

## Skill Creator との連携

[Skill Creator GitHub App](https://skill-creator.app) を使用すると、**両方**が生成されます:
- 従来の SKILL.md ファイル（後方互換性のため）
- 本能コレクション（v2 学習システム用）

リポジトリ分析からの本能は `source: "repo-analysis"` を持ち、ソースリポジトリの URL を含みます。

## 信頼度スコアリング

信頼度は時間とともに進化します:

| スコア | 意味 | 動作 |
|-------|---------|----------|
| 0.3 | 暫定的 | 提案されるが強制されない |
| 0.5 | 中程度 | 関連する場合に適用 |
| 0.7 | 強い | 適用が自動承認 |
| 0.9 | ほぼ確実 | コア動作 |

**信頼度が上がる**場合:
- パターンが繰り返し観察される
- ユーザーが提案された動作を修正しない
- 他のソースからの類似した本能が一致する

**信頼度が下がる**場合:
- ユーザーが明示的に動作を修正する
- パターンが長期間観察されない
- 矛盾するエビデンスが現れる

## 観察に Hooks vs Skills を使う理由

> 「v1 は観察にスキルを使用していました。スキルは確率的で、Claude の判断に基づいて約50〜80%の確率で発火します。」

Hooks は**100%の確率**で、決定論的に発火します。これは以下を意味します:
- すべてのツール呼び出しが観察される
- パターンを見逃さない
- 学習が包括的

## 後方互換性

v2 は v1 と完全に互換性があります:
- 既存の `~/.claude/skills/learned/` スキルは引き続き動作
- Stop hook も引き続き実行（ただし v2 にもフィード）
- 段階的な移行パス: 両方を並行して実行可能

## プライバシー

- 観察はマシン上に**ローカル**で保存
- エクスポートできるのは**本能**（パターン）のみ
- 実際のコードや会話内容は共有されない
- エクスポート内容はあなたが制御

## 関連

- [Skill Creator](https://skill-creator.app) - リポジトリ履歴から本能を生成
- [Homunculus](https://github.com/humanplane/homunculus) - v2 アーキテクチャのインスピレーション
- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 継続学習セクション

---

*本能ベース学習: 1つの観察から、Claude にあなたのパターンを教える。*
