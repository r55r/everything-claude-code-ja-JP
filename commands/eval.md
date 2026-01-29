# Eval Command

評価駆動開発ワークフローを管理します。

## 使用方法

`/eval [define|check|report|list] [feature-name]`

## 評価の定義

`/eval define feature-name`

新しい評価定義を作成:

1. テンプレートで `.claude/evals/feature-name.md` を作成:

```markdown
## EVAL: feature-name
Created: $(date)

### Capability Evals
- [ ] [機能1の説明]
- [ ] [機能2の説明]

### Regression Evals
- [ ] [既存の動作1が引き続き動作する]
- [ ] [既存の動作2が引き続き動作する]

### Success Criteria
- capability evalsのpass@3 > 90%
- regression evalsのpass^3 = 100%
```

2. 具体的な基準を記入するようユーザーに促す

## 評価のチェック

`/eval check feature-name`

機能の評価を実行:

1. `.claude/evals/feature-name.md` から評価定義を読み込み
2. 各capability evalに対して:
   - 基準を検証しようと試みる
   - PASS/FAILを記録
   - `.claude/evals/feature-name.log` に試行を記録
3. 各regression evalに対して:
   - 関連するテストを実行
   - ベースラインと比較
   - PASS/FAILを記録
4. 現在のステータスを報告:

```
EVAL CHECK: feature-name
========================
Capability: X/Y passing
Regression: X/Y passing
Status: IN PROGRESS / READY
```

## 評価レポート

`/eval report feature-name`

包括的な評価レポートを生成:

```
EVAL REPORT: feature-name
=========================
Generated: $(date)

CAPABILITY EVALS
----------------
[eval-1]: PASS (pass@1)
[eval-2]: PASS (pass@2) - required retry
[eval-3]: FAIL - see notes

REGRESSION EVALS
----------------
[test-1]: PASS
[test-2]: PASS
[test-3]: PASS

METRICS
-------
Capability pass@1: 67%
Capability pass@3: 100%
Regression pass^3: 100%

NOTES
-----
[問題、エッジケース、または観察事項]

RECOMMENDATION
--------------
[SHIP / NEEDS WORK / BLOCKED]
```

## 評価一覧

`/eval list`

すべての評価定義を表示:

```
EVAL DEFINITIONS
================
feature-auth      [3/5 passing] IN PROGRESS
feature-search    [5/5 passing] READY
feature-export    [0/4 passing] NOT STARTED
```

## 引数

$ARGUMENTS:
- `define <name>` - 新しい評価定義を作成
- `check <name>` - 評価を実行してチェック
- `report <name>` - 完全なレポートを生成
- `list` - すべての評価を表示
- `clean` - 古い評価ログを削除（直近10回の実行を保持）
