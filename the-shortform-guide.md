# The Shorthand Guide to Everything Claude Code（日本語版）

![Header: Anthropic Hackathon Winner - Tips & Tricks for Claude Code](./assets/images/shortform/00-header.png)

---

**2月の実験的リリースからClaude Codeのヘビーユーザーで、[@DRodriguezFX](https://x.com/DRodriguezFX)と共に[zenith.chat](https://zenith.chat)でAnthropic x Forum Venturesハッカソンで優勝しました - すべてClaude Codeを使用して。**

10ヶ月の日常使用後の完全なセットアップをご紹介します：スキル、フック、サブエージェント、MCP、プラグイン、そして実際に機能するもの。

---

## スキルとコマンド

スキルはルールのように機能し、特定のスコープとワークフローに制限されます。特定のワークフローを実行する必要があるときのプロンプトのショートハンドです。

Opus 4.5との長いコーディングセッションの後、デッドコードや不要な.mdファイルをクリーンアップしたい場合は`/refactor-clean`を実行。テストが必要な場合は`/tdd`、`/e2e`、`/test-coverage`。スキルにはコードマップも含めることができます - Claudeがコンテキストを消費せずにコードベースを素早くナビゲートする方法です。

![Terminal showing chained commands](./assets/images/shortform/02-chaining-commands.jpeg)
*コマンドをチェーンして実行*

コマンドはスラッシュコマンドで実行されるスキルです。重複しますが、保存場所が異なります：

- **スキル**: `~/.claude/skills/` - より広範なワークフロー定義
- **コマンド**: `~/.claude/commands/` - 素早く実行可能なプロンプト

```bash
# スキル構造の例
~/.claude/skills/
  pmx-guidelines.md      # プロジェクト固有のパターン
  coding-standards.md    # 言語のベストプラクティス
  tdd-workflow/          # README.mdを含むマルチファイルスキル
  security-review/       # チェックリストベースのスキル
```

---

## フック

フックは特定のイベントで発火するトリガーベースの自動化です。スキルとは異なり、ツール呼び出しとライフサイクルイベントに制限されています。

**フックの種類：**

1. **PreToolUse** - ツール実行前（検証、リマインダー）
2. **PostToolUse** - ツール完了後（フォーマット、フィードバックループ）
3. **UserPromptSubmit** - メッセージ送信時
4. **Stop** - Claudeの応答完了時
5. **PreCompact** - コンテキスト圧縮前
6. **Notification** - 権限リクエスト

**例：長時間実行コマンド前のtmuxリマインダー**

```json
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm|pnpm|yarn|cargo|pytest)\"",
      "hooks": [
        {
          "type": "command",
          "command": "if [ -z \"$TMUX\" ]; then echo '[Hook] Consider tmux for session persistence' >&2; fi"
        }
      ]
    }
  ]
}
```

![PostToolUse hook feedback](./assets/images/shortform/03-posttooluse-hook.png)
*PostToolUseフック実行時にClaude Codeで得られるフィードバックの例*

**プロのヒント：** `hookify`プラグインを使用すると、JSONを手動で書く代わりに会話形式でフックを作成できます。`/hookify`を実行して、欲しいものを説明してください。

---

## サブエージェント

サブエージェントは、オーケストレーター（メインのClaude）が限定されたスコープでタスクを委任できるプロセスです。バックグラウンドまたはフォアグラウンドで実行でき、メインエージェントのコンテキストを解放します。

サブエージェントはスキルとうまく連携します - スキルのサブセットを実行できるサブエージェントにタスクを委任し、それらのスキルを自律的に使用できます。また、特定のツール権限でサンドボックス化することもできます。

```bash
# サブエージェント構造の例
~/.claude/agents/
  planner.md           # 機能実装の計画
  architect.md         # システム設計の決定
  tdd-guide.md         # テスト駆動開発
  code-reviewer.md     # 品質/セキュリティレビュー
  security-reviewer.md # 脆弱性分析
  build-error-resolver.md
  e2e-runner.md
  refactor-cleaner.md
```

適切なスコープのために、サブエージェントごとに許可されたツール、MCP、権限を設定します。

---

## ルールとメモリ

`.rules`フォルダには、Claudeが**常に**従うべきベストプラクティスを含む`.md`ファイルが格納されています。2つのアプローチ：

1. **単一のCLAUDE.md** - すべてを1つのファイルに（ユーザーまたはプロジェクトレベル）
2. **ルールフォルダ** - 関心事ごとにグループ化されたモジュラーな`.md`ファイル

```bash
~/.claude/rules/
  security.md      # ハードコードされた秘密なし、入力の検証
  coding-style.md  # 不変性、ファイル構成
  testing.md       # TDDワークフロー、80%カバレッジ
  git-workflow.md  # コミットフォーマット、PRプロセス
  agents.md        # サブエージェントへの委任タイミング
  performance.md   # モデル選択、コンテキスト管理
```

**ルールの例：**

- コードベースに絵文字なし
- フロントエンドで紫色の色調を避ける
- デプロイ前に必ずコードをテスト
- メガファイルよりモジュラーコードを優先
- console.logをコミットしない

---

## MCP（Model Context Protocol）

MCPはClaudeを外部サービスに直接接続します。APIの代替ではありません - APIのプロンプト駆動ラッパーであり、情報のナビゲーションにより柔軟性を持たせます。

**例：** Supabase MCPにより、Claudeはコピー＆ペーストなしで特定のデータを取得し、上流で直接SQLを実行できます。データベース、デプロイメントプラットフォームなども同様です。

![Supabase MCP listing tables](./assets/images/shortform/04-supabase-mcp.jpeg)
*Supabase MCPがパブリックスキーマ内のテーブルをリストする例*

**Claude内のChrome：** Claudeがブラウザを自律的に制御できる組み込みプラグインMCPです - クリックして動作を確認できます。

**重要：コンテキストウィンドウ管理**

MCPは厳選してください。すべてのMCPをユーザー設定に保持していますが、**未使用のものはすべて無効化**しています。`/plugins`に移動して下にスクロールするか、`/mcp`を実行してください。

![/plugins interface](./assets/images/shortform/05-plugins-interface.jpeg)
*/pluginsを使用してMCPに移動し、現在インストールされているものとそのステータスを確認*

圧縮前の200kコンテキストウィンドウは、有効なツールが多すぎると70kしかない可能性があります。パフォーマンスは大幅に低下します。

**目安：** 設定に20-30のMCPを持ち、有効は10未満/アクティブツールは80未満に抑える。

```bash
# 有効なMCPを確認
/mcp

# ~/.claude.jsonのprojects.disabledMcpServersで未使用のものを無効化
```

---

## プラグイン

プラグインは面倒な手動セットアップの代わりに、簡単にインストールできるようにツールをパッケージ化します。プラグインはスキル+MCPの組み合わせ、またはフック/ツールのバンドルになります。

**プラグインのインストール：**

```bash
# マーケットプレイスを追加
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep

# Claudeを開き、/pluginsを実行、新しいマーケットプレイスを見つけて、そこからインストール
```

![Marketplaces tab showing mgrep](./assets/images/shortform/06-marketplaces-mgrep.jpeg)
*新しくインストールされたMixedbread-Grepマーケットプレイスの表示*

**LSPプラグイン**は、エディタの外でClaude Codeを頻繁に実行する場合に特に便利です。Language Server Protocolにより、IDEを開かなくてもClaudeにリアルタイムの型チェック、定義へのジャンプ、インテリジェントな補完を提供します。

```bash
# 有効なプラグインの例
typescript-lsp@claude-plugins-official  # TypeScriptインテリジェンス
pyright-lsp@claude-plugins-official     # Python型チェック
hookify@claude-plugins-official         # 会話形式でフックを作成
mgrep@Mixedbread-Grep                   # ripgrepより優れた検索
```

MCPと同じ警告 - コンテキストウィンドウに注意。

---

## ヒントとコツ

### キーボードショートカット

- `Ctrl+U` - 行全体を削除（バックスペース連打より速い）
- `!` - クイックbashコマンドプレフィックス
- `@` - ファイルを検索
- `/` - スラッシュコマンドを開始
- `Shift+Enter` - 複数行入力
- `Tab` - 思考表示を切り替え
- `Esc Esc` - Claudeを中断/コードを復元

### 並列ワークフロー

- **フォーク** (`/fork`) - 重複しないタスクを並列で行うために会話をフォーク（キューにメッセージを詰め込む代わりに）
- **Gitワークツリー** - 競合なしで重複する並列Claudeを実行。各ワークツリーは独立したチェックアウト

```bash
git worktree add ../feature-branch feature-branch
# 各ワークツリーで別々のClaudeインスタンスを実行
```

### 長時間実行コマンド用のtmux

Claudeが実行するログ/bashプロセスをストリームして監視：

https://github.com/user-attachments/assets/shortform/07-tmux-video.mp4

```bash
tmux new -s dev
# Claudeはここでコマンドを実行、デタッチして再アタッチ可能
tmux attach -t dev
```

### mgrep > grep

`mgrep`はripgrep/grepから大幅に改善されています。プラグインマーケットプレイスからインストールし、`/mgrep`スキルを使用。ローカル検索とWeb検索の両方で動作します。

```bash
mgrep "function handleSubmit"  # ローカル検索
mgrep --web "Next.js 15 app router changes"  # Web検索
```

### その他の便利なコマンド

- `/rewind` - 以前の状態に戻る
- `/statusline` - ブランチ、コンテキスト%、todosでカスタマイズ
- `/checkpoints` - ファイルレベルのアンドゥポイント
- `/compact` - 手動でコンテキスト圧縮を実行

### GitHub Actions CI/CD

GitHub ActionsでPRのコードレビューを設定。設定すればClaudeが自動的にPRをレビューできます。

![Claude bot approving a PR](./assets/images/shortform/08-github-pr-review.jpeg)
*Claudeがバグ修正PRを承認*

### サンドボックス

リスクのある操作にはサンドボックスモードを使用 - Claudeは実際のシステムに影響を与えない制限された環境で実行されます。

---

## エディタについて

エディタの選択はClaude Codeのワークフローに大きく影響します。Claude Codeはどのターミナルからでも動作しますが、高機能なエディタと組み合わせることで、リアルタイムのファイル追跡、素早いナビゲーション、統合されたコマンド実行が可能になります。

### Zed（私の好み）

[Zed](https://zed.dev)を使用しています - Rustで書かれているので本当に速い。即座に開き、巨大なコードベースも問題なく処理し、システムリソースをほとんど消費しません。

**Zed + Claude Codeが素晴らしい組み合わせである理由：**

- **速度** - Rustベースのパフォーマンスにより、Claudeが高速にファイルを編集してもラグがない。エディタがついていける
- **エージェントパネル統合** - ZedのClaude統合により、Claudeが編集する際にリアルタイムでファイル変更を追跡。Claudeが参照するファイル間をエディタを離れずにジャンプ
- **CMD+Shift+Rコマンドパレット** - すべてのカスタムスラッシュコマンド、デバッガー、ビルドスクリプトに検索可能なUIで素早くアクセス
- **最小限のリソース使用** - 重い操作中にClaudeとRAM/CPUを奪い合わない。Opus実行時に重要
- **Vimモード** - お好みなら完全なvimキーバインディング

![Zed Editor with custom commands](./assets/images/shortform/09-zed-editor.jpeg)
*CMD+Shift+Rを使用したカスタムコマンドドロップダウン付きのZed Editor。右下に表示されているのはフォローモードの的。*

**エディタに依存しないヒント：**

1. **画面を分割** - 片側にClaude Code付きターミナル、もう片側にエディタ
2. **Ctrl + G** - ZedでClaudeが現在作業中のファイルを素早く開く
3. **自動保存** - Claudeのファイル読み取りが常に最新になるよう自動保存を有効に
4. **Git統合** - コミット前にClaudeの変更をレビューするためにエディタのgit機能を使用
5. **ファイルウォッチャー** - ほとんどのエディタは変更されたファイルを自動リロード、これが有効になっていることを確認

### VSCode / Cursor

これも実行可能な選択肢であり、Claude Codeとうまく動作します。ターミナル形式で使用でき、`\ide`を使用してエディタと自動同期し、LSP機能を有効にできます（プラグインで多少冗長になりました）。または、エディタにより統合され、マッチするUIを持つ拡張機能を選択できます。

![VS Code Claude Code Extension](./assets/images/shortform/10-vscode-extension.jpeg)
*VS Code拡張機能はClaude Codeのネイティブグラフィカルインターフェースを提供し、IDEに直接統合されます。*

---

## 私のセットアップ

### プラグイン

**インストール済み：** （通常は一度に4-5個のみ有効化）

```markdown
ralph-wiggum@claude-code-plugins       # ループ自動化
frontend-design@claude-code-plugins    # UI/UXパターン
commit-commands@claude-code-plugins    # Gitワークフロー
security-guidance@claude-code-plugins  # セキュリティチェック
pr-review-toolkit@claude-code-plugins  # PR自動化
typescript-lsp@claude-plugins-official # TSインテリジェンス
hookify@claude-plugins-official        # フック作成
code-simplifier@claude-plugins-official
feature-dev@claude-code-plugins
explanatory-output-style@claude-code-plugins
code-review@claude-code-plugins
context7@claude-plugins-official       # ライブドキュメント
pyright-lsp@claude-plugins-official    # Python型
mgrep@Mixedbread-Grep                  # より良い検索
```

### MCPサーバー

**設定済み（ユーザーレベル）：**

```json
{
  "github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"] },
  "firecrawl": { "command": "npx", "args": ["-y", "firecrawl-mcp"] },
  "supabase": {
    "command": "npx",
    "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=YOUR_REF"]
  },
  "memory": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-memory"] },
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  },
  "vercel": { "type": "http", "url": "https://mcp.vercel.com" },
  "railway": { "command": "npx", "args": ["-y", "@railway/mcp-server"] },
  "cloudflare-docs": { "type": "http", "url": "https://docs.mcp.cloudflare.com/mcp" },
  "cloudflare-workers-bindings": {
    "type": "http",
    "url": "https://bindings.mcp.cloudflare.com/mcp"
  },
  "clickhouse": { "type": "http", "url": "https://mcp.clickhouse.cloud/mcp" },
  "AbletonMCP": { "command": "uvx", "args": ["ableton-mcp"] },
  "magic": { "command": "npx", "args": ["-y", "@magicuidesign/mcp@latest"] }
}
```

これが鍵です - 14のMCPを設定していますが、プロジェクトごとに有効なのは約5-6個のみ。コンテキストウィンドウを健全に保ちます。

### 主要なフック

```json
{
  "PreToolUse": [
    { "matcher": "npm|pnpm|yarn|cargo|pytest", "hooks": ["tmuxリマインダー"] },
    { "matcher": "Write && .md file", "hooks": ["README/CLAUDE以外はブロック"] },
    { "matcher": "git push", "hooks": ["レビュー用にエディタを開く"] }
  ],
  "PostToolUse": [
    { "matcher": "Edit && .ts/.tsx/.js/.jsx", "hooks": ["prettier --write"] },
    { "matcher": "Edit && .ts/.tsx", "hooks": ["tsc --noEmit"] },
    { "matcher": "Edit", "hooks": ["grep console.log警告"] }
  ],
  "Stop": [
    { "matcher": "*", "hooks": ["変更されたファイルのconsole.logをチェック"] }
  ]
}
```

### カスタムステータスライン

ユーザー、ディレクトリ、ダーティインジケーター付きgitブランチ、残りコンテキスト%、モデル、時間、todoカウントを表示：

![Custom status line](./assets/images/shortform/11-statusline.jpeg)
*Macルートディレクトリでのステータスラインの例*

```
affoon:~ ctx:65% Opus 4.5 19:52
▌▌ plan mode on (shift+tab to cycle)
```

### ルール構造

```
~/.claude/rules/
  security.md      # 必須のセキュリティチェック
  coding-style.md  # 不変性、ファイルサイズ制限
  testing.md       # TDD、80%カバレッジ
  git-workflow.md  # 規約に沿ったコミット
  agents.md        # サブエージェント委任ルール
  patterns.md      # APIレスポンスフォーマット
  performance.md   # モデル選択（Haiku vs Sonnet vs Opus）
  hooks.md         # フックのドキュメント
```

### サブエージェント

```
~/.claude/agents/
  planner.md           # 機能を分解
  architect.md         # システム設計
  tdd-guide.md         # テストを先に書く
  code-reviewer.md     # 品質レビュー
  security-reviewer.md # 脆弱性スキャン
  build-error-resolver.md
  e2e-runner.md        # Playwrightテスト
  refactor-cleaner.md  # デッドコード削除
  doc-updater.md       # ドキュメントを同期
```

---

## 主なポイント

1. **複雑にしすぎない** - 設定はアーキテクチャではなく、ファインチューニングとして扱う
2. **コンテキストウィンドウは貴重** - 未使用のMCPとプラグインを無効化
3. **並列実行** - 会話をフォーク、gitワークツリーを使用
4. **繰り返しを自動化** - フォーマット、リント、リマインダー用のフック
5. **サブエージェントのスコープを限定** - 限定されたツール = 集中した実行

---

## 参考資料

- [プラグインリファレンス](https://code.claude.com/docs/en/plugins-reference)
- [フックのドキュメント](https://code.claude.com/docs/en/hooks)
- [チェックポイント](https://code.claude.com/docs/en/checkpointing)
- [インタラクティブモード](https://code.claude.com/docs/en/interactive-mode)
- [メモリシステム](https://code.claude.com/docs/en/memory)
- [サブエージェント](https://code.claude.com/docs/en/sub-agents)
- [MCP概要](https://code.claude.com/docs/en/mcp-overview)

---

**注意：** これは詳細の一部です。高度なパターンについては[Longform Guide](./the-longform-guide.md)を参照してください。

---

*NYCでのAnthropic x Forum Venturesハッカソンで[@DRodriguezFX](https://x.com/DRodriguezFX)と共に[zenith.chat](https://zenith.chat)を構築して優勝*
