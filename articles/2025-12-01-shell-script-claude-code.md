---
title: "シェルスクリプト × Claude Code: 日常スクリプトへの導入実践"
emoji: "🐴"
type: "tech"
topics:
  - "claudecode"
  - "shellscript"
  - "github"
publication_name: "genda_jp"
published: true
published_at: 2025-12-01 07:00
---

## 1. はじめに

株式会社GENDA データエンジニア / MLOps エンジニアの uma-chan です。
この記事は GENDA Advent Calendar 2025 シリーズ1 1日目の記事です。

https://qiita.com/advent-calendar/2025/genda

最近は AI コーディングエージェントをターミナルから利用する機会が増えてきました。

- Claude Code
- Codex CLI
- Gemini CLI
- GitHub Copilot CLI

など様々なツールが登場しています。

これらのツールは対話的に使うだけでなく、シェルスクリプトに組み込んで自動化に活用することもできます。
本記事では私が日常的に使っているシェルスクリプトに Claude Code を導入した実践例を紹介します。

:::message
本記事の内容は 2025年12月時点での検証結果に基づいています。
:::

## 2. 対象読者

- シェルスクリプトに抵抗がない方
- AI CLI ツールの活用に興味がある方

## 3. AI CLI ツールの非対話モード

各 AI CLI ツールには、プロンプトを渡して結果を受け取る非対話モードが用意されています。

### 3.1. 各ツールのコマンド

| ツール             | コマンド例                |
| ---                | ---                       |
| Claude Code        | `claude -p "プロンプト"`  |
| Gemini CLI         | `gemini "プロンプト"`     |
| GitHub Copilot CLI | `copilot -p "プロンプト"` |
| Codex CLI          | `codex exec "プロンプト"` |

### 3.2. シェルスクリプトでの利用における重要な違い

実際に各ツールを試してみたところ、シェルスクリプトで `$()` を使って結果を変数に取り込む用途では大きな違いがありました。

```bash
# 理想的な動作
result=$(ai-tool "Hello と返してください")
echo "$result"  # => Hello
```

Claude Code だけが余計な出力なしで純粋な結果のみを返します。

他のツールはツール実行情報などが含まれてしまいシェルスクリプトで取り扱う際に `grep` や `sed` でゴミを除去する手間が発生します。

というわけでシェルスクリプトへの組み込みには Claude Code が最適です。

## 4. 題材スクリプトの紹介

私が日常的に使っている `issue-worktree-create` というスクリプトを題材にします。

以下は執筆時点のバージョンのスクリプトです。

https://github.com/i9wa4/dotfiles/blob/f92d338fb8e21ba915df1ee077d53eafafa1a118/bin/issue-worktree-create

このスクリプトは GitHub Issue に対応する Git Worktree を作成するものです。

### 4.1. 解決したかった課題

初期はスクリプト化した都合で `issue-123` のような単純なブランチ名にしていました。

しばらく運用していると管理しやすくするためにブランチ名をちゃんと生成したいと思うようになりました。

### 4.2. 改善後のスクリプトの概要

このスクリプトは以下の処理を行います。

1. 引数で Issue 番号を受け取る
2. `gh` コマンドで Issue 情報を取得
3. Claude Code でブランチ名のスラッグを生成
4. `git worktree` で作業ディレクトリを作成

## 5. Claude Code の組み込み方法

### 5.1. 基本形

Claude Code の非対話モードは `-p` または `--print` オプションで利用できます。

```bash
result=$(claude -p "プロンプト")
```

### 5.2. 実際のコード

issue-worktree-create での実装例です。

```bash
branch_slug=$(claude --print "以下のIssue情報から、Gitブランチ名として適切な英語の短いスラッグ(kebab-case)を生成してください。
- 最大5単語程度
- 小文字とハイフンのみ使用
- 特殊文字は除去
- 動詞から始めることを推奨

出力は英語のスラッグのみを返してください(説明不要)。

---
Issueタイトル: ${issue_title}
---
Issue本文:
${issue_body}
---
Issueコメント:
${issue_comments}
---" 2>/dev/null | tr -d '\n' || echo "")
```

### 5.3. プロンプト設計のポイント

#### 5.3.1. 出力形式を明確に指定する

```text
出力は英語のスラッグのみを返してください(説明不要)。
```

AI は親切に説明を付けたがりますが、シェルスクリプトでは純粋な結果だけが必要です。「説明不要」と明示することで余計な出力を防ぎます。

#### 5.3.2. 制約条件を箇条書きで示す

```text
- 最大5単語程度
- 小文字とハイフンのみ使用
- 特殊文字は除去
- 動詞から始めることを推奨
```

期待する出力の形式を具体的に示すことで、安定した結果が得られます。

#### 5.3.3. 入力情報を構造化して渡す

```text
---
Issueタイトル: ${issue_title}
---
Issue本文:
${issue_body}
---
```

AI が理解しやすいように、入力情報をセクションで区切って渡します。

### 5.4. エラーハンドリング

```bash
... 2>/dev/null | tr -d '\n' || echo ""
```

- `2>/dev/null`: 標準エラー出力を破棄
- `tr -d '\n'`: 改行を除去
- `|| echo ""`: エラー時は空文字を返す

AI の呼び出しが失敗しても、スクリプト全体が止まらないようにしています。空文字が返った場合は後続の処理でデフォルトのブランチ名 (`issue-123`) を使用します。

## 6. 応用アイデア

Claude Code の非対話モードはAIに安定した出力を期待できる場面で活用できます。

### 6.1. コミットメッセージの生成

```bash
commit_msg=$(GIT_PAGER=cat git diff --staged --no-color | claude -p "以下の差分から適切なコミットメッセージを1行で生成してください。Conventional Commits 形式で。出力はメッセージのみ。")
git commit -m "$commit_msg"
```

### 6.2. ファイル名のリネーム

```bash
new_name=$(claude -p "以下のファイル名を英語の kebab-case に変換してください。出力はファイル名のみ。: ${old_name}")
mv "${old_name}" "${new_name}"
```

### 6.3. ログの要約

```bash
summary=$(tail -100 /var/log/app.log | claude -p "以下のログから重要なイベントを3行で要約してください。")
echo "$summary"
```

## 7. おわりに

せっかく定額課金しているので応用範囲を広げて日常の様々な作業に活用していきたいですね。
