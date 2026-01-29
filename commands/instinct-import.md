---
name: instinct-import
description: Import instincts from teammates, Skill Creator, or other sources
command: /instinct-import
implementation: python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py import <file>
---

# Instinct Import Command

## 実装

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py import <file-or-url> [--dry-run] [--force] [--min-confidence 0.7]
```

以下からインスティンクトをインポート:
- チームメイトのエクスポート
- Skill Creator（リポジトリ分析）
- コミュニティコレクション
- 以前のマシンのバックアップ

## 使用方法

```
/instinct-import team-instincts.yaml
/instinct-import https://github.com/org/repo/instincts.yaml
/instinct-import --from-skill-creator acme/webapp
```

## 実行内容

1. インスティンクトファイルを取得（ローカルパスまたはURL）
2. フォーマットを解析して検証
3. 既存のインスティンクトとの重複をチェック
4. 新しいインスティンクトをマージまたは追加
5. `~/.claude/homunculus/instincts/inherited/` に保存

## インポートプロセス

```
📥 Importing instincts from: team-instincts.yaml
================================================

Found 12 instincts to import.

Analyzing conflicts...

## New Instincts (8)
These will be added:
  ✓ use-zod-validation (confidence: 0.7)
  ✓ prefer-named-exports (confidence: 0.65)
  ✓ test-async-functions (confidence: 0.8)
  ...

## Duplicate Instincts (3)
Already have similar instincts:
  ⚠️ prefer-functional-style
     Local: 0.8 confidence, 12 observations
     Import: 0.7 confidence
     → Keep local (higher confidence)

  ⚠️ test-first-workflow
     Local: 0.75 confidence
     Import: 0.9 confidence
     → Update to import (higher confidence)

## Conflicting Instincts (1)
These contradict local instincts:
  ❌ use-classes-for-services
     Conflicts with: avoid-classes
     → Skip (requires manual resolution)

---
Import 8 new, update 1, skip 3?
```

## マージ戦略

### 重複の場合
既存のインスティンクトと一致するものをインポートする場合:
- **高い信頼度が優先**: 信頼度の高い方を保持
- **証拠をマージ**: 観測回数を結合
- **タイムスタンプを更新**: 最近検証されたものとしてマーク

### 競合の場合
既存のインスティンクトと矛盾するものをインポートする場合:
- **デフォルトでスキップ**: 競合するインスティンクトはインポートしない
- **レビュー用にフラグ**: 両方に注意が必要とマーク
- **手動解決**: ユーザーがどちらを保持するか決定

## ソース追跡

インポートされたインスティンクトには以下がマークされます:
```yaml
source: "inherited"
imported_from: "team-instincts.yaml"
imported_at: "2025-01-22T10:30:00Z"
original_source: "session-observation"  # または "repo-analysis"
```

## Skill Creator統合

Skill Creatorからインポートする場合:

```
/instinct-import --from-skill-creator acme/webapp
```

これはリポジトリ分析から生成されたインスティンクトを取得します:
- ソース: `repo-analysis`
- 高い初期信頼度（0.7以上）
- ソースリポジトリにリンク

## フラグ

- `--dry-run`: インポートせずにプレビュー
- `--force`: 競合があってもインポート
- `--merge-strategy <higher|local|import>`: 重複の処理方法
- `--from-skill-creator <owner/repo>`: Skill Creator分析からインポート
- `--min-confidence <n>`: しきい値以上のインスティンクトのみをインポート

## 出力

インポート後:
```
✅ Import complete!

Added: 8 instincts
Updated: 1 instinct
Skipped: 3 instincts (2 duplicates, 1 conflict)

New instincts saved to: ~/.claude/homunculus/instincts/inherited/

Run /instinct-status to see all instincts.
```
