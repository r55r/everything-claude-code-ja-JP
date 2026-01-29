---
name: continuous-learning
description: Automatically extract reusable patterns from Claude Code sessions and save them as learned skills for future use.
---

# 継続的学習スキル

Claude Codeセッション終了時に自動的に評価を行い、学習済みスキルとして保存できる再利用可能なパターンを抽出します。

## 仕組み

このスキルは各セッション終了時に **Stopフック** として実行されます:

1. **セッション評価**: セッションが十分なメッセージ数（デフォルト: 10以上）を持っているか確認
2. **パターン検出**: セッションから抽出可能なパターンを特定
3. **スキル抽出**: 有用なパターンを `~/.claude/skills/learned/` に保存

## 設定

`config.json` を編集してカスタマイズ:

```json
{
  "min_session_length": 10,
  "extraction_threshold": "medium",
  "auto_approve": false,
  "learned_skills_path": "~/.claude/skills/learned/",
  "patterns_to_detect": [
    "error_resolution",
    "user_corrections",
    "workarounds",
    "debugging_techniques",
    "project_specific"
  ],
  "ignore_patterns": [
    "simple_typos",
    "one_time_fixes",
    "external_api_issues"
  ]
}
```

## パターンタイプ

| パターン | 説明 |
|---------|-------------|
| `error_resolution` | 特定のエラーがどのように解決されたか |
| `user_corrections` | ユーザーの修正からのパターン |
| `workarounds` | フレームワーク/ライブラリの癖への解決策 |
| `debugging_techniques` | 効果的なデバッグアプローチ |
| `project_specific` | プロジェクト固有の規約 |

## フックのセットアップ

`~/.claude/settings.json` に追加:

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
      }]
    }]
  }
}
```

## なぜStopフックか？

- **軽量**: セッション終了時に一度だけ実行
- **ノンブロッキング**: 各メッセージにレイテンシーを追加しない
- **完全なコンテキスト**: 完全なセッショントランスクリプトにアクセス可能

## 関連

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 継続的学習のセクション
- `/learn` コマンド - セッション中の手動パターン抽出

---

## 比較メモ（リサーチ: 2025年1月）

### vs Homunculus (github.com/humanplane/homunculus)

Homunculus v2はより洗練されたアプローチを採用:

| 機能 | 私たちのアプローチ | Homunculus v2 |
|---------|--------------|---------------|
| 観察 | Stopフック（セッション終了時） | PreToolUse/PostToolUseフック（100%信頼性） |
| 分析 | メインコンテキスト | バックグラウンドエージェント（Haiku） |
| 粒度 | 完全なスキル | アトミックな「instincts」 |
| 信頼度 | なし | 0.3-0.9の重み付け |
| 進化 | 直接スキルへ | Instincts → クラスター → スキル/コマンド/エージェント |
| 共有 | なし | instinctsのエクスポート/インポート |

**homunculusからの重要な洞察:**
> "v1はスキルに観察を依存していました。スキルは確率的で、約50-80%の確率で発火します。v2は観察にフック（100%信頼性）を使用し、学習された行動のアトミックな単位としてinstinctsを使用します。"

### 潜在的なv2の機能強化

1. **Instinctベースの学習** - 信頼度スコアリング付きの小さくアトミックな行動
2. **バックグラウンドオブザーバー** - 並行して分析するHaikuエージェント
3. **信頼度の減衰** - 矛盾する場合はinstinctsの信頼度が低下
4. **ドメインタグ付け** - code-style、testing、git、debuggingなど
5. **進化パス** - 関連するinstinctsをスキル/コマンドにクラスタリング

詳細は: `/Users/affoon/Documents/tasks/12-continuous-learning-v2.md` の完全な仕様を参照。
