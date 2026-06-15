---
type: meta
title: "ダッシュボード"
updated: 2026-06-15
tags:
  - meta
  - dashboard
status: evergreen
related:
  - "[[index]]"
  - "[[log]]"
  - "[[hot]]"
---

# ダッシュボード

ナビゲーション: [[index]] | [[log]] | [[hot]]

> **注**: 以下のクエリは [Dataview プラグイン](https://github.com/blacksmithgu/obsidian-dataview) が必要です。

---

## 最近更新されたページ

```dataview
TABLE updated, status
FROM "wiki"
WHERE type != "meta"
SORT updated DESC
LIMIT 10
```

---

## ステータス別ページ数

```dataview
TABLE length(rows) AS "件数"
FROM "wiki"
WHERE type != "meta"
GROUP BY status
```

---

## 孤立ページ（被リンクなし）

```dataview
LIST
FROM "wiki"
WHERE type != "meta"
AND length(file.inlinks) = 0
```

---

## タイプ別ページ一覧

```dataview
TABLE title, status, updated
FROM "wiki"
WHERE type = "concept"
SORT updated DESC
```

---

## 最近の操作ログ

```dataview
TABLE file.mtime AS "更新日時"
FROM "wiki/log.md"
SORT file.mtime DESC
LIMIT 5
```
