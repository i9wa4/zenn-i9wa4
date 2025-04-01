---
title: "Aider+ChatGPTのAIエージェントを使ってみる"
emoji: "🐴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "aider"
  - "chatgpt"
published: false
publication_name: "genda_jp"
---

## 1. はじめに

こんにちは。株式会社GENDAの MLOps エンジニア/データエンジニアの uma-chan です。

チーム内に展開しやすい形でAIエージェントを導入してみようと思い Aider + ChatGPT を試してみました。

## 2. Aider とは

Aider は手元のコーディング環境 (ターミナル) にAIエージェントを組み込むことができるツールです。

https://aider.chat/

https://github.com/Aider-AI/aider

## 3. 導入手順

### 3.1. Aider のインストール

https://aider.chat/docs/install.html

```bash
python -m pip install aider-install
aider-install
```

### 3.2. OpenAI API キーの取得

以下から API キーを取得します

https://platform.openai.com/api-keys

取得できたら `~/.zshenv` 等に以下を追加します。

```bash
# Mac/Linux
export OPENAI_API_KEY=<key>
```

### 3.3. OpenAI 向け設定

https://aider.chat/docs/llms/openai.html

```bash
python -m pip install -U aider-chat

# OpenAI API キーの読み込み (~/.zshenv の場合は初回のみ)
. ~/.zshenv

# 利用できるモデルの確認
aider --list-models openai/

# gpt-4 モデルを利用する
aider --model openai/gpt-4
```

## 4. 利用手順
