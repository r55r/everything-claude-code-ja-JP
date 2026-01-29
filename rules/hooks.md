# フックシステム

## フックの種類

- **PreToolUse**: ツール実行前（バリデーション、パラメータ修正）
- **PostToolUse**: ツール実行後（自動フォーマット、チェック）
- **Stop**: セッション終了時（最終確認）

## 現在のフック（~/.claude/settings.json に設定）

### PreToolUse
- **tmux リマインダー**: 長時間実行コマンド（npm, pnpm, yarn, cargo など）に tmux を提案
- **git push レビュー**: プッシュ前に Zed でレビューを開く
- **ドキュメントブロッカー**: 不要な .md/.txt ファイルの作成をブロック

### PostToolUse
- **PR 作成**: PR URL と GitHub Actions ステータスをログ出力
- **Prettier**: 編集後の JS/TS ファイルを自動フォーマット
- **TypeScript チェック**: .ts/.tsx ファイル編集後に tsc を実行
- **console.log 警告**: 編集ファイル内の console.log を警告

### Stop
- **console.log 監査**: セッション終了前に変更ファイルの console.log をチェック

## 自動承認パーミッション

慎重に使用:
- 信頼できる、明確に定義された計画に対して有効化
- 探索的な作業では無効化
- dangerously-skip-permissions フラグは決して使用しない
- 代わりに `~/.claude.json` で `allowedTools` を設定

## TodoWrite ベストプラクティス

TodoWrite ツールの使用目的:
- 複数ステップのタスク進捗を追跡
- 指示の理解を確認
- リアルタイムでの軌道修正を可能に
- 詳細な実装ステップを表示

Todo リストで明らかになること:
- 順序が間違っているステップ
- 欠落している項目
- 不要な追加項目
- 粒度の問題
- 要件の誤解
