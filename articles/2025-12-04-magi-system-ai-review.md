---
title: "Vim x tmux によるレビュー用 MAGI システム (AI多数決レビューシステム)"
emoji: "🐴"
type: "tech"
topics:
  - "vim"
  - "tmux"
  - "claudecode"
  - "codexcli"
publication_name: "genda_jp"
published: true
published_at: 2025-12-04 07:00
---

## 1. はじめに

株式会社GENDA データエンジニア / MLOps エンジニアの uma-chan です。
この記事は GENDA Advent Calendar 2025 シリーズ3 Day 4 の記事です。

https://qiita.com/advent-calendar/2025/genda

Vim と tmux を活用して複数の AI エージェント (Claude Code、Codex CLI) による多数決レビューの仕組みを構築しました。

:::message
本記事の内容は 2025年12月時点での検証結果に基づいています。
:::

## 2. MAGI システムとは

アニメ「新世紀エヴァンゲリオン」に登場する3台のスーパーコンピュータシステムです。

- MELCHIOR-1: 科学者としての赤木ナオコ
- BALTHASAR-2: 母親としての赤木ナオコ
- CASPER-3: 女としての赤木ナオコ

それぞれ異なる人格 (観点) が独立に判断し、多数決で結論を下します。

これを AI レビューに応用し、複数のエージェントから異なる観点でレビューを受ける仕組みを構築しました。

## 3. 構成

### 3.1. 利用ツール

#### 3.1.1. vde-layout

https://github.com/yuki-yano/vde-layout

`vde-layout` コマンドを実行することで予め設定した tmux レイアウトを適用できます。
YAML で定義したレイアウトを一発で構築できるため、毎回手動でペイン分割する手間が省けます。

#### 3.1.2. vim-tmux-send-to-ai-cli

https://github.com/i9wa4/vim-tmux-send-to-ai-cli

Vim から tmux 経由で AI エージェント起動中のペインにプロンプトを送信するためのプラグインです。

送信先

- 同一ウインドウの中で最も小さい番号のペインに送信 (デフォルト)
- 同一ウインドウの特定のペインに送信
- 同一ウインドウの全てのペインに送信

送信内容

- カーソル行
- 段落
- 選択範囲
- ファイル全体
- 行範囲 (例: 10行目から20行目)

#### 3.1.3. ddc-source-slash-commands

https://github.com/i9wa4/ddc-source-slash-commands

Vim 補完プラグイン ddc.vim のソースプラグインです。
カスタムスラッシュコマンドを補完候補として表示します。

https://github.com/Shougo/ddc.vim

カスタムスラッシュコマンドに対応していない AI エージェントの場合はファイルのフルパスを送信すれば問題ありません。

#### 3.1.4. AI エージェント

- Claude Code: Anthropic 公式の CLI ツール。Claude Opus/Sonnet を利用可能
- Codex CLI: OpenAI 公式の CLI ツール。o3/o4-mini を利用可能

https://docs.anthropic.com/en/docs/claude-code

https://github.com/openai/codex

### 3.2. レイアウト構成

```text
+----------------------------+----------------------------+
|                            |                            |
|          pane 0            |          pane 1            |
|           Vim              |        Claude Code         |
|                            |                            |
+------------------+------------------+------------------+
|      pane 2      |      pane 3      |      pane 4      |
|    Codex CLI     |    Codex CLI     |   Claude Code    |
+------------------+------------------+------------------+
```

5ペイン構成 (Vim 1 + AI エージェント 4) です。

### 3.3. 操作デモ

以下は Vim から複数の AI エージェントに同じプロンプトを送信している様子です (4ペイン構成)。

@[youtube](AyE-ruGOGCc)

詳細は下記の記事を参照してください。

https://i9wa4.github.io/blog/2025-11-19-vim-tmux-orchestrating-ai-coding-agents.html

### 3.4. vde-layout 設定

実際に使用している設定ファイルです。

```yaml
# ~/.config/vde/layout.yml
presets:
  review:
    name: review
    description: review
    windowMode: current-window
    layout:
      type: vertical
      ratio: [3, 1]
      panes:
        - type: horizontal
          ratio: [1, 1]
          panes:
            - name: editor
              command: vim -c "execute 'edit' '.i9wa4/temp.md'->expand()"
              focus: true
            - name: claude
              command: claude --dangerously-skip-permissions
        - type: horizontal
          ratio: [1, 1, 1]
          panes:
            - name: codex
              command: codex --yolo
            - name: codex
              command: codex --yolo
            - name: claude
              command: claude --dangerously-skip-permissions
```

ポイント

- `windowMode: current-window` で現在のウインドウにレイアウトを適用
- 上段は Vim と Claude Code を 1:1 で配置
- 下段は Codex CLI x 2 と Claude Code を 1:1:1 で配置
- Vim にフォーカスが当たった状態で起動

## 4. カスタムスラッシュコマンド

MAGI システムの核となる2つのカスタムスラッシュコマンドです。

### 4.1. /my-review

各ペインの AI エージェントが独立してレビューを実施するコマンドです。

```markdown
# review

このファイルを読んだら直ちに GitHub Pull Request レビューを実施する

## 1. 前提情報

- この Worktree は Pull Request の対象ブランチである
- このディレクトリ名もしくはブランチ名から Pull Request 番号を取得できる
- もし Pull Request 番号が不明な場合は分岐元との差分を元にレビューを実施する

## 2. 差分の取得方法

- YOU MUST: 差分は `git diff main...HEAD` で取得する (ドット3つ)

## 3. レビュー方法

- YOU MUST: あらゆる指摘を詳細に抽出する
- YOU MUST: 重要度順に指摘を並べる

## 4. レビュアーとしてのペルソナ・役割について

- YOU MUST: tmux ペイン番号 N ごとに異なるレビュアー役割を担う
    - N % 3 == 0: セキュリティ専門家。堅実派。
    - N % 3 == 1: パフォーマンス専門家。せっかち。
    - N % 3 == 2: コード品質専門家。完璧主義者。

## 5. レビュー結果の Markdown ファイル出力

- YOU MUST: レビュー結果を以下に保存する
    - `.i9wa4/YYYYMMDD-pN-review.md` (N: tmux ペイン番号)
```

ポイント

- 出力ファイル名に tmux ペイン番号を含めることで各エージェントが同時に実行しても競合しない
- ペイン番号に応じてペルソナを切り替えることで、同じモデルでも異なる観点からレビューを受けられる (とはいえ同じモデルではあるので限界はある)

### 4.2. /summarize-reviews

全ペインのレビュー結果を集約するコマンドです。

```markdown
# summarize-reviews

このファイルを読んだらレビュー結果を集約して

## 1. 手順

1. `.i9wa4/` ディレクトリの君以外のペインのレビュー結果を読み込む
1. すべてのレビュー結果を `.i9wa4/YYYYMMDD-pX-review.md` に集約し
   重要度順に並び替える
1. ファイル冒頭に指摘タイトル・重要度・指摘者の対応表を追加する
```

## 5. ワークフロー

### 5.1. レイアウト起動

```bash
vde-layout review
```

### 5.2. 全ペインへ /my-review を一斉送信

vim-tmux-send-to-ai-cli の機能で、`/my-review` を全 AI エージェントに一斉送信します。

各エージェントは独立して PR の差分を取得し、レビューを実施します。

### 5.3. レビュー結果の収集

各エージェントからのレビュー結果は以下の形式で出力されます。

`.i9wa4/YYYYMMDD-pN-review.md`

出力例

- `.i9wa4/20251201-p1-review.md` (Claude Code)
- `.i9wa4/20251201-p2-review.md` (Codex CLI)
- `.i9wa4/20251201-p3-review.md` (Codex CLI)
- `.i9wa4/20251201-p4-review.md` (Claude Code)

### 5.4. 結果の統合

いずれかのペインに `/summarize-reviews` を送信すると、全レビュー結果を集約した `.i9wa4/YYYYMMDD-pX-review.md` が生成されます。

集約ファイルには指摘タイトル・重要度・指摘者の対応表が含まれます。
複数エージェントが同じ点を指摘した場合は優先度高として扱えます。

## 6. メリット

### 6.1. 多角的なレビュー

異なるモデル (Claude Opus、o3) から異なる観点でレビューを受けられます。

### 6.2. 並列実行による時間短縮

順次実行と比較して、レビュー時間を大幅に短縮できます。

### 6.3. ターミナル完結

IDE を使わず、Vim と tmux だけで完結します。

## 7. コンフリクト回避の仕組み

複数の AI エージェントを並列実行する際、最大の課題はファイル競合です。
この MAGI システムでは以下の仕組みでコンフリクトを回避しています。

### 7.1. レビュータスクの特性を活用

レビューは「読み取り専用」のタスクです。
各エージェントは以下の操作のみを行います。

- `git diff` でコード差分を読み取る
- コードファイルを読み取る
- レビュー結果を出力する

ソースコードを編集しないため、そもそもコンフリクトが発生しにくい構造になっています。

### 7.2. tmux ペイン番号による出力ファイルの分離

各エージェントの出力ファイル名に tmux ペイン番号を含めることで、ファイル競合を回避します。

```text
.i9wa4/YYYYMMDD-pN-review.md
                 ^
                 tmux pane number
```

各エージェントは自分のペイン番号を以下のコマンドで取得します。

```bash
tmux display-message -p -t "${TMUX_PANE}" '#{pane_index}'
```

これにより各ペインの出力先が分離されます。

- pane 1 の Claude Code -> `.i9wa4/20251201-p1-review.md`
- pane 2 の Codex CLI -> `.i9wa4/20251201-p2-review.md`
- pane 3 の Codex CLI -> `.i9wa4/20251201-p3-review.md`
- pane 4 の Claude Code -> `.i9wa4/20251201-p4-review.md`

同じファイルに書き込むことがないため、競合は発生しません。

### 7.3. global gitignore ディレクトリの活用

出力先の `.i9wa4/` ディレクトリは global gitignore に登録されています。

- Git 管理対象外のため、レビュー結果ファイルが誤ってコミットされない
- 各開発者が独自の作業ディレクトリとして利用可能
- プロジェクトの `.gitignore` を汚さない

参考: @mizchi さんのポスト

@[tweet](https://twitter.com/mizchi/status/1914543131888066561)

### 7.4. コード編集タスクへの応用

レビュー以外のコード編集タスクで並列実行する場合は、Cursor 2.0 と同様に git worktree を活用する方法が有効です。

```bash
git worktree add ../project-agent1 -b feature/agent1
git worktree add ../project-agent2 -b feature/agent2
git worktree add ../project-agent3 -b feature/agent3
```

各エージェントが独立したワーキングディレクトリで作業することで、ファイル編集の競合を回避できます。

## 8. 注意点

### 8.1. コスト

複数エージェントを並列実行するため、トークン使用量が増加します。

### 8.2. タスクの選定

並列実行に適したタスク

- コードレビュー (読み取り専用)
- ドキュメント生成 (出力先を分離)
- テスト作成 (ファイル名を分離)

並列実行に不向きなタスク

- 同一ファイルの編集
- 依存関係のある連続タスク

### 8.3. セキュリティ

`--dangerously-skip-permissions` や `--yolo` オプションはすべてのツール実行を自動承認します。
信頼できる環境でのみ使用してください。

## 9. おわりに

複数の AI エージェントを並列実行し、MAGI システムのようなレビュー体制を構築する方法を紹介しました。

ターミナルでの AI 活用がさらに広がることを期待しています。
