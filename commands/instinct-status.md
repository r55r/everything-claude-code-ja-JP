---
name: instinct-status
description: Show all learned instincts with their confidence levels
command: /instinct-status
implementation: python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
---

# インスティンクトステータスコマンド

学習したすべてのインスティンクトを、ドメイン別にグループ化して信頼度スコアとともに表示します。

## 実装

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
```

## 使用方法

```
/instinct-status
/instinct-status --domain code-style
/instinct-status --low-confidence
```

## 実行内容

1. `~/.claude/homunculus/instincts/personal/` からすべてのインスティンクトファイルを読み込む
2. `~/.claude/homunculus/instincts/inherited/` から継承されたインスティンクトを読み込む
3. ドメイン別にグループ化し、信頼度バーとともに表示する

## 出力フォーマット

```
📊 インスティンクトステータス
==================

## コードスタイル (4 インスティンクト)

### prefer-functional-style
トリガー: 新しい関数を書くとき
アクション: クラスより関数型パターンを使用
信頼度: ████████░░ 80%
ソース: session-observation | 最終更新: 2025-01-22

### use-path-aliases
トリガー: モジュールをインポートするとき
アクション: 相対インポートの代わりに @/ パスエイリアスを使用
信頼度: ██████░░░░ 60%
ソース: repo-analysis (github.com/acme/webapp)

## テスト (2 インスティンクト)

### test-first-workflow
トリガー: 新機能を追加するとき
アクション: 実装の前にテストを書く
信頼度: █████████░ 90%
ソース: session-observation

## ワークフロー (3 インスティンクト)

### grep-before-edit
トリガー: コードを修正するとき
アクション: Grepで検索し、Readで確認してからEditする
信頼度: ███████░░░ 70%
ソース: session-observation

---
合計: 9 インスティンクト (4 個人、5 継承)
オブザーバー: 実行中 (最終分析: 5分前)
```

## フラグ

- `--domain <name>`: ドメインでフィルター (code-style, testing, git など)
- `--low-confidence`: 信頼度 < 0.5 のインスティンクトのみ表示
- `--high-confidence`: 信頼度 >= 0.7 のインスティンクトのみ表示
- `--source <type>`: ソースでフィルター (session-observation, repo-analysis, inherited)
- `--json`: プログラムで使用するためにJSONとして出力
