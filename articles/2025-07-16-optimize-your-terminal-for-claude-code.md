---
title: "個人的ターミナル最適化 for Claude Code"
emoji: "🐴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "vim"
  - "tmux"
  - "claudecode"
published: true
---

## 1. はじめに

Claude Code で並行作業を行いやすくなるための工夫ポイントを全て共有します！

本当の私個人の趣味盛り盛りの内容なので役に立つ部分だけ読む形でお願いします。

なお分量が多いので参考記事やリンクを用意していない部分もあります。

インストール方法も各自で調べてください。

## 2. 対象読者

- Claude Code を使っている方
- GitHub を使っている方
- Vim を使っている、もしくは使いたい方

## 3. ターミナル以前

### 3.1. モニタ

私は27インチ WQHD モニタを1枚使っていますが、やはりウルトラワイドモニタが良いらしいですね。

とはいえ私は出先や出社時はPC内蔵モニタを使っており、その場合も以降の内容で十分に Claude Code で並行作業はできるのでモニタが整っていない勢も安心して読み進めてください。

### 3.2. OS

最近よく使っているという理由で macOS (MacBook) を前提に話しますが、適宜読み替えても同様の環境が構築できるはずです。

一応 Ubuntu や Windows (WSL) も利用してはいるのである程度はクロスプラットフォームな内容にできていると思います。

### 3.3. ウインドウマネージャー

ターミナルと他ウインドウとの行き来を快適にできるのであればウインドウマネージャーは必須ではないです。

macOS では Windows でいう「Win+数字」に相当する一手で表示アプリを切り替えるショートカットが無いので AeroSpace を利用しています。

https://github.com/nikitabobko/AeroSpace

導入時に以下の記事がとても参考になりました。

https://zenn.dev/mozumasu/articles/mozumasu-window-costomization

### 3.4. ブラウザ

私は複数ウインドウや仮想デスクトップを使いこなすのが苦手なので Chrome のタブグループ機能で並行作業に対応することが多いです。

### 3.5. dotfiles

Claude Code による生産性をいかに上げるかが注目されている昨今、いつでもどんな PC でも快適に作業するために設定を記述しておく dotfiles の重要性が増していると個人的に話題です。

dotfiles にまだ入門できていない方は入門記事を参考にしてください。

今回述べている内容はほとんど以下の dotfiles に記述したものとなります。

https://github.com/i9wa4/dotfiles

私の設定で気になる内容がある場合は以下の DeepWiki で質問してみると的確な回答が得られます。

https://deepwiki.com/i9wa4/dotfiles

## 4. シェル

Zsh を愛用してます。

以下で Zsh のおすすめプラグインを紹介します。

### 4.1. Zinit

Zsh のプラグインマネージャーです。

https://github.com/zdharma-continuum/zinit

### 4.2. zeno.zsh (Zsh/Fish)

Zsh の操作体験を全般的に向上させるプラグインの zeno.zsh です。

https://github.com/yuki-yano/zeno.zsh

正直これが無いとターミナルを使う気になれないと思えるくらいに便利です。

最近 Fish にも対応したようなので Fish ユーザーの方もぜひ試してみてください。

作者の yuki-yano さんによる記事が参考になります。

https://zenn.dev/yano/articles/zsh_plugin_from_deno

以下で Claude Code と共に生きる上で zeno.zsh に担ってもらっている役割を紹介します。

もちろんここに記述していない他の機能も便利です。

#### 4.2.1. ghq 管理リポジトリへの移動

ghq については後述します。

zeno.zsh によって ghq + fzf の組み合わせでリポジトリへの移動がとても快適になります。

#### 4.2.2. tmux セッション名へのリポジトリ名自動反映

ghq でリポジトリ移動すると tmux セッション名が自動でリポジトリ名に変更されます。

これにより難なく複数セッションを扱えます。

## 5. ターミナル

何でもよいのですが私は Alacritty を使っています。

通知音が鳴るように設定しておくとよいです。

## 6. tmux

複数ターミナルの管理について、タブ機能をもつターミナルもありますが私は tmux で複数ターミナルを管理するのが好きです。

https://github.com/tmux/tmux/wiki/Installing

### 6.1. tmux おすすめキーバインド

```conf:tmux.conf
bind-key -n M-Left previous-window
bind-key -n M-Right next-window
bind-key -n M-Up switch-client -p
bind-key -n M-Down switch-client -n
```

こちらは

- tmux のセッション間を Alt + 上下キーで切り替える
- tmux のウインドウ間を Alt + 左右キーで切り替える

を実現するキーバインドです。

これにより Claude Code の並行作業の様子を片手でセッションやウインドウを切り替えながら確認できます。

```conf:tmux.conf
bind-key % split-window -h -c "#{pane_current_path}"
bind-key '"' split-window -v -c "#{pane_current_path}"
bind-key c new-window -c "#{pane_current_path}"
```

こちらは現在開いているディレクトリを継承して新しいペインやウインドウを作成するキーバインドです。

Claude Code と同じディレクトリに移動した状態のターミナルを即座に起動できます。

### 6.2. tmux おすすめプラグイン

https://github.com/tmux-plugins/tpm

tmux のプラグインマネージャーです。

https://github.com/kristijanhusak/tmux-simple-git-status

tmux のステータスバーに Git の状態を表示するプラグインです。

## 7. Git

### 7.1. worktree

最近注目されている `git worktree` を使いましょう。

worktree 作成作業は定型化できると思うので worktree 作成用のスクリプトを作成しておくと便利です。

### 7.2. gitignore

リポジトリ内の作業ディレクトリ (`tmp/` など) を gitignore して Claude Code の作業場にしておくことをおすすめします。

指示用のドキュメントやログを格納して Claude Code に読ませるといった使い方ができます。

### 7.3. ghq

リポジトリの管理は ghq に任せたほうがよいです。

https://github.com/x-motemen/ghq

## 8. エディタ

私は Vim が好きで使っています。

以下で私が Claude Code のために利用している Vim プラグインを紹介します。

### 8.1. vim-tmux-send-to-ai-cli

https://github.com/i9wa4/vim-tmux-send-to-ai-cli

拙作プラグインです。

Vim のバッファや複数行にわたる選択範囲などを自在に tmux の Claude Code ペインに送信できます。

現時点では Claude Code と Gemini CLI に対応しています。

このプラグインを使うと基本的に AI への入力にまつわる問題を Vim での文章作成における問題に帰着できます。

つまり何が言いたいかというと AI の入力欄の機能に依存せずに AI への入力を作成できるということです。

Vim に帰着させることで改行しづらい問題や Gemini CLI にカスタムスラッシュコマンドがない問題などを全て解決できます。

### 8.2. ddc.vim

https://github.com/Shougo/ddc.vim

補完機能を強化するために ddc.vim を使っています。

### 8.3. skkeleton

https://github.com/vim-skk/skkeleton

Vim で日本語入力をしづらいという問題を解決するために SKK を利用できる skkeleton を使っています。

私は ddc.vim の source として利用しています。

### 8.4. ddc-source-file

https://github.com/LumaKernel/ddc-source-file

ddc.vim の source です。ファイル名やパスを補完してくれます。

一応 Vim の補完機能で

> CTRL-X CTRL-F

https://vim-jp.org/vimdoc-ja/usr_24.html#24.3

を使用すると同等の補完ができますが、より快適な ddc.vim の補完機能がよいと思います。

## 9. Claude Code 設定

### 9.1. `$CLAUDE_CONFIG_DIR`

必ず設定しましょう。

私は以下のように設定しています。

```sh:~/.zshenv
export CLAUDE_CONFIG_DIR="${XDG_CONFIG_HOME}"/claude
```

### 9.2. CLAUDE.md

私は CLAUDE.md は作成していません。

CLAUDE.md を作成せず Claude Code に最適化しすぎないことで Gemini CLI などの他の AI ツールに移行しやすくしようと考えています。

もう1つの理由としては再読み込みが必要になる場面では CLAUDE.md よりカスタムスラッシュコマンドの再実行の方が楽だと感じているからです。

### 9.3. カスタムスラッシュコマンド

ここが AI とやりとりをする上での肝になります。

私の dotfiles では以下のカスタムスラッシュコマンドを用意しています。

```sh
.
└── commands
    ├── commit.md
    ├── CONTRIBUTING.md
    ├── issue-to-pr.md
    ├── learn-bigquery.md
    ├── learn-databricks.md
    ├── learn-dbt.md
    └── review.md
```

issue-to-pr.md については、羃等性をもたせ Anthropic のベストプラクティスに沿った形で Pull Request 作成に誘導できるコマンドに仕上げている最中で、ノウハウが蓄積されてきたら別途記事にしようと思います。

### 9.4. settings.json

あまり凝ったことはしていません。

```json:settings.json
{
    "permissions": {
        "allow": [
        ],
        "deny": [
            "Bash(rm:*)",
            "WebFetch(domain:github.com)"
        ]
    },
    "env": {
        "BASH_DEFAULT_TIMEOUT_MS": 300000,
        "BASH_MAX_TIMEOUT_MS": 300000
    },
    "includeCoAuthoredBy": false
}
```

基本的に

```sh
claude --dangerously-skip-permissions
```

で起動するので permissions に関しては allow ではなく deny を充実させていく方針で設定しています。

Claude Code はよくファイル削除を試みるので `Bash(rm:*)` は必須だと思います。

### 9.5. .claude.json

通知音を鳴らすための設定を行います。

```sh
claude config set --global preferredNotifChannel terminal_bell
```

このコマンドを実行すると以下のような設定が `.claude.json` に追加されます。

```json:.claude.json
  "preferredNotifChannel": "terminal_bell",
```

これで足りなければ Hooks 設定の作り込みで対応していきましょう。

通知音設定については勉強中です。

## 10. Dev Container

Dev Container で Claude Code を利用することで安全に

```sh
claude --dangerously-skip-permissions
```

ができます。

https://docs.anthropic.com/en/docs/claude-code/devcontainer

Anthropic 公式が示す `.devcontainer` がそのまま使えます。

以下のようにグローバルな Claude Code 設定をマウントするなどして利便性向上を図りつつ使っていきましょう。

```json:.devcontainer/devcontainer.json
    "mounts": [
        "source=claude-code-bashhistory-${devcontainerId},target=/commandhistory,type=volume",
        "source=claude-code-config-${devcontainerId},target=/home/node/.claude,type=volume",
        "source=${localEnv:HOME}/ghq/github.com/i9wa4/dotfiles/dot.config/claude,target=/home/node/.claude,type=bind,consistency=cached"
    ],
```

https://github.com/devcontainers/cli

Dev Container を CLI で利用する際は Dev Container CLI を利用しましょう。

## 11. おわりに

各項目それだけでも記事になるくらいの内容で消化が大変かもしれませんが参考になれば幸いです。
