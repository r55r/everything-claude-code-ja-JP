# Checkpoint Command

ワークフローにチェックポイントを作成または検証します。

## 使用方法

`/checkpoint [create|verify|list] [name]`

## チェックポイントの作成

チェックポイント作成時:

1. `/verify quick` を実行して現在の状態がクリーンであることを確認
2. チェックポイント名でgit stashまたはコミットを作成
3. `.claude/checkpoints.log` にチェックポイントを記録:

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. チェックポイント作成完了を報告

## チェックポイントの検証

チェックポイントとの比較検証時:

1. ログからチェックポイントを読み込み
2. 現在の状態をチェックポイントと比較:
   - チェックポイント以降に追加されたファイル
   - チェックポイント以降に変更されたファイル
   - 現在とその時点でのテスト合格率
   - 現在とその時点でのカバレッジ

3. 報告:
```
CHECKPOINT COMPARISON: $NAME
============================
Files changed: X
Tests: +Y passed / -Z failed
Coverage: +X% / -Y%
Build: [PASS/FAIL]
```

## チェックポイント一覧

すべてのチェックポイントを表示:
- 名前
- タイムスタンプ
- Git SHA
- ステータス（current、behind、ahead）

## ワークフロー

典型的なチェックポイントフロー:

```
[開始] --> /checkpoint create "feature-start"
   |
[実装] --> /checkpoint create "core-done"
   |
[テスト] --> /checkpoint verify "core-done"
   |
[リファクタリング] --> /checkpoint create "refactor-done"
   |
[PR] --> /checkpoint verify "feature-start"
```

## 引数

$ARGUMENTS:
- `create <name>` - 名前付きチェックポイントを作成
- `verify <name>` - 名前付きチェックポイントと比較検証
- `list` - すべてのチェックポイントを表示
- `clear` - 古いチェックポイントを削除（直近5件は保持）
