# Git ワークフロー

## コミットメッセージ形式

```
<type>: <description>

<optional body>
```

タイプ: feat, fix, refactor, docs, test, chore, perf, ci

注: ~/.claude/settings.json でアトリビューションはグローバルに無効化されています。

## プルリクエストワークフロー

PR 作成時:
1. 完全なコミット履歴を分析（最新のコミットだけでなく）
2. `git diff [base-branch]...HEAD` で全ての変更を確認
3. 包括的な PR サマリーを作成
4. TODO を含むテスト計画を記載
5. 新しいブランチの場合は `-u` フラグでプッシュ

## 機能実装ワークフロー

1. **まず計画**
   - **planner** エージェントで実装計画を作成
   - 依存関係とリスクを特定
   - フェーズに分割

2. **TDD アプローチ**
   - **tdd-guide** エージェントを使用
   - まずテストを書く（RED）
   - テストをパスするように実装（GREEN）
   - リファクタリング（IMPROVE）
   - 80%以上のカバレッジを確認

3. **コードレビュー**
   - コード作成直後に **code-reviewer** エージェントを使用
   - CRITICAL と HIGH の問題に対処
   - 可能な場合は MEDIUM の問題も修正

4. **コミット＆プッシュ**
   - 詳細なコミットメッセージ
   - Conventional Commits 形式に従う
