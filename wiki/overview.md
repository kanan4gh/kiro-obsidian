---
type: overview
title: "Wiki 概要"
created: 2026-06-15
updated: 2026-06-15
tags:
  - meta
  - overview
status: developing
related:
  - "[[index]]"
  - "[[hot]]"
  - "[[log]]"
  - "[[meta/dashboard]]"
  - "[[LLM Wikiパターン]]"
sources:
---

# Wiki 概要

ナビゲーション: [[index]] | [[hot]] | [[log]] | [[meta/dashboard]]

---

## 目的

これは **kiro-obsidian** のデモ Vault です。[[LLM Wikiパターン]] を実演します。Kiro IDE と Obsidian を使って、ソースを追加するたびに知識が複利で増える永続的な第二の脳を構築するシステムです。

`#skill-wiki` を参照して scaffold を実行し、この Vault を自分のドメインに合わせてカスタマイズしてください。

---

## 初期シードコンテンツ

**コンセプト（seeded）**:
- [[LLM Wikiパターン]] — 永続的・複利的な知識ベースのコアアーキテクチャ
- [[Hot Cache]] — セッションコンテキストのメカニズム

**エンティティ・ソース**:
- （ingest 後に追加されます）

---

## 現在の状態

- 取り込み済みソース: 0
- Wiki ページ数: 2
- 最終活動: 初期セットアップ（2026-06-15）

---

## キーとなるコンセプト

**知識は複利で増える。** RAG と異なり、wiki は合成を事前にコンパイルする。クロスリファレンスはすでに存在し、矛盾はフラグが立てられる。新しいソースを ingest するたびに、孤立したチャンクを追加するのではなく既存ページが充実する。

**Hot Cache は力の乗数器。** 〜500語のファイルが最近のコンテキストを保持する。新しいセッションは最小限のトークンコストで全コンテキストを持った状態で始まる。

**Obsidian は IDE、Kiro は AI 。** Graph View は接続関係を可視化する。人間はソースをキュレートし質問を投げかける。Kiro がすべての wiki ページを作成・維持する。
