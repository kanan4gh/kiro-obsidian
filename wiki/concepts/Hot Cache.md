---
type: concept
title: "Hot Cache"
complexity: basic
domain: knowledge-management
aliases:
  - "hot.md"
  - "セッションキャッシュ"
  - "コンテキストキャッシュ"
created: 2026-06-15
updated: 2026-06-15
tags:
  - concept
  - knowledge-management
  - context
status: mature
related:
  - "[[LLM Wikiパターン]]"
  - "[[index]]"
  - "[[hot]]"
  - "[[concepts/_index]]"
sources:
---

# Hot Cache

wiki Vault の最新コンテキストを〜500語でまとめたファイル。`wiki/hot.md` に格納される。セッション終了後と、重要な ingest やクエリのたびに更新される。

Hot Cache が存在する理由は1つ: 「どこで止まったか？」という問いに答えるため。新しいセッションは最初に `hot.md` を読む。答えがあれば、残りの wiki を読み込む必要はない。

---

## 格納する内容

- 最近 ingest またはディスカッションされた内容
- 最近の重要な事実とテイクアウェイ
- 最近作成・更新されたページ
- 進行中のスレッドと未解決の問い
- 現在ユーザーが注目していること

---

## フォーマット

```markdown
---
type: meta
title: "Hot Cache"
updated: YYYY-MM-DDTHH:MM:SS
---

# 最近のコンテキスト

## 最終更新
YYYY-MM-DD — [何があったか]

## 最近の重要な事実
- [最も重要な最近のテイクアウェイ]
- [2番目]

## 最近作成・更新されたページ
- [[ページ名]] — [概要]

## 進行中のスレッド
- [オープンな問いや未完了の作業]
```

---

## 更新のタイミング

- **ingest 後**: 新しいソースから重要な事実を Hot Cache に追加する
- **query 後**: 重要な発見や回答のサマリーを追加する
- **セッション終了前**: 「hot cache を更新して」と指示する
- **手動**: いつでも更新できる

---

## なぜ機能するのか

新しいセッションのたびにすべての wiki ページを読み込む必要がない。Hot Cache の〜500語で、セッションのほとんどのコンテキストニーズを満たせる。コンテキストウィンドウを節約しながら、前のセッションの続きをシームレスに開始できる。

---
