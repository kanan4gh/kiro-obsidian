---
type: meta
title: "はじめかた"
updated: 2026-06-15
tags:
  - meta
  - onboarding
status: evergreen
related:
  - "[[index]]"
  - "[[overview]]"
  - "[[LLM Wikiパターン]]"
---

# kiro-obsidian のはじめかた

ようこそ。この Vault は知識が複利で増える第二の脳です。Kiro IDE と Obsidian を組み合わせ、ソースを追加するたびに 8〜15 の相互リンクされた wiki ページが自動生成されます。

---

## 3ステップ クイックスタート

### 1. ソースを用意する

`.raw/` フォルダに任意のドキュメントを入れます:
- PDF、Markdown ファイル、テキスト、記事
- または Kiro に URL を渡してフェッチしてもらう

### 2. ingest する

Kiro のチャットで自然言語で指示します:

```
ingest [ファイル名]
```

Kiro がソースを読み込み、`wiki/` 以下に 8〜15 の wiki ページを作成し、`wiki/index.md`・`wiki/log.md`・`wiki/hot.md` を更新します。

### 3. 質問する

```
[トピック]について何を知っている？
```

Kiro が Hot Cache を読み、インデックスをスキャンし、関連ページを掘り下げて、wiki ページを引用した合成回答を返します。

---

## Hot Cache の仕組み

`wiki/hot.md` は最近の Vault コンテキストを〜500語でまとめたファイルです。Kiro のセッション開始時に `wiki-core.md`（always steering）によって自動的に読まれます。

毎回のセッションで前の続きから始められます。手動で更新することもできます:

```
hot cache を更新して
```

---

## 最初の ingest — ウォークスルー

1. `.raw/` にファイルを作成（テキスト、PDF、記事など）
2. Kiro IDE でこの Vault フォルダを開く
3. チャットで入力: `ingest [ファイル名]`
4. wiki が成長するのを見る — Kiro が作成したページを報告します
5. `wiki/index.md` を開く — 新しいページがリストされています
6. Obsidian の Graph View を開く — 接続されたノードのクラスターが現れます

3〜5 回 ingest すると、グラフが本物の知識ネットワークのように見えてきます。クロスリファレンスが自動的に生成されます。

---

## 主なコマンド（Kiro への自然言語指示）

| 指示の例 | Kiro が行うこと |
|---------|----------------|
| `[ファイル名] を ingest して` | ソースから 8〜15 の wiki ページを作成 |
| `[トピック] について教えて` | wiki を検索して引用付きで回答 |
| `wiki を lint して` | 孤立ページ・デッドリンク・矛盾を検出 |
| `hot cache を更新して` | セッションコンテキストを更新 |

スキルの詳細は Kiro の steering ファイルを参照:
- `#skill-wiki` — Vault scaffold
- `#skill-wiki-ingest` — ingest の詳細手順（常時ロード済み）
- `#skill-wiki-query` — クエリの詳細手順
- `#skill-wiki-lint` — lint の詳細手順

---

## Vault を探索する

- **[[index]]** — 全 wiki ページのマスターカタログ
- **[[overview]]** — Vault コンテンツの概要
- **[[LLM Wikiパターン]]** — この Vault が基づくパターン
- **[[meta/dashboard]]** — ライブ Dataview クエリ（Dataview プラグインが必要）
