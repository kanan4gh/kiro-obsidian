---
inclusion: always
---

# wiki-core: Kiro + Obsidian 知識 Vault コア規約

このファイルは常時コンテキストに注入される（inclusion: always）。Vault の構造・規約・スキル起動ルーティング・hot cache 管理を定義する。

---

## Vault の目的

このリポジトリは **LLM Wiki パターン** の実装である。ソースを追加するたびに wiki が充実し、過去の知識と新しい知識が自動的に接続される複利型の知識ベースを構築する。

wiki はチャットの産物ではなく、**永続する成果物**である。エンティティ・概念・出典ページは Obsidian Vault に蓄積され、次のセッションでも即座に参照できる。

---

## Vault 構造

```
.raw/              ソースドキュメント（読み取り専用・変更禁止）
wiki/              LLM が生成・管理する知識ベース
_templates/        Obsidian Templater テンプレート
_attachments/      画像・PDF 等の添付ファイル
.vault-meta/       Vault メタデータ（transport.json / mode.json / locks/）
```

### wiki/ の標準構造

```
wiki/
├── index.md            すべてのページのマスターカタログ
├── log.md              全操作の時系列記録（追記専用）
├── hot.md              hot cache: 直近の文脈サマリー（〜500語）
├── overview.md         Vault 全体のエグゼクティブサマリー
├── sources/            ソースごとの要約ページ（1ソース1ページ）
├── entities/           人物・組織・製品・リポジトリ
│   └── _index.md
├── concepts/           アイデア・パターン・フレームワーク
│   └── _index.md
├── comparisons/        比較分析
├── questions/          クエリへの回答を保存したページ
└── meta/               ダッシュボード・lint レポート・規約
```

---

## セッション開始時の動作

**セッション開始時に必ず実行すること（サイレントに、ユーザーへの報告は不要）:**

1. `wiki/hot.md` が存在する場合は読み込んで文脈を復元する
2. `wiki/hot.md` が存在しない場合は何もしない（vault 未初期化状態）
3. 読み込んだ内容はアナウンスせず、単に文脈として保持する

> **Kiro 制約**: Kiro には自動 SessionStart フックがない。セッション再開時は `.kiro/hooks/wiki-session-start.json`（userTriggered）を手動起動するか、ユーザーが "hot cache を読んで" と指示することで hot.md を読み込む。

**コンテキスト消失時（PostCompact 相当）:**
Kiro にはコンテキスト圧縮後の自動 hot.md 再読機能がない。文脈が失われた場合はユーザーに「`wiki/hot.md` を再読する」と伝えてから読み込む。

---

## スキル起動ルーティング

ユーザーの発言に応じて対応するスキルを参照すること:

| ユーザーの発言 | 操作 | 参照スキル |
|--------------|------|-----------|
| "ingest [ファイル名]"・"取り込んで"・"追加して" | INGEST | `#skill-wiki-ingest` |
| "X について何を知ってる？"・"query:"・"調べて" | QUERY | `#skill-wiki-query` |
| "lint the wiki"・"ヘルスチェック"・"wiki を整理" | LINT | `#skill-wiki-lint` |
| "vault を作って"・"wiki を設定"・"scaffold" | SCAFFOLD | `#skill-wiki` |
| "mode を変えて"・"methodology mode" | MODE | `#skill-wiki-mode` |
| "save this"・"会話を保存"・"/save" | SAVE | `#skill-save` |
| "/autoresearch"・"自律リサーチ" | AUTORESEARCH | `#skill-autoresearch` |

---

## hot cache（wiki/hot.md）の管理

`wiki/hot.md` は直近の文脈の〜500語のサマリーである。セッション間の継続性を提供する。

### 更新タイミング
- ingest 完了後
- 重要な query やりとりの後
- セッション終了前（ユーザーに促すか、自発的に更新する）

> **Kiro 制約**: Kiro には Stop フック（セッション終了時の自動更新）がない。セッション終了前にユーザーに「hot.md を更新しますか？」と確認するか、自発的に更新すること。

### hot.md のフォーマット

```markdown
---
type: meta
title: "Hot Cache"
updated: YYYY-MM-DDTHH:MM:SS
---

# 直近の文脈

## 最終更新
YYYY-MM-DD。[何が起きたか]

## 主要な直近ファクト
- [最も重要な直近の知見]
- [2番目に重要な知見]

## 直近の変更
- 作成: [[新ページ1]]、[[新ページ2]]
- 更新: [[既存ページ]]（X のセクションを追加）
- フラグ: [[ページA]] と [[ページB]] の間に Y に関する矛盾

## アクティブなスレッド
- ユーザーは現在 [トピック] を調査中
- 未解決の疑問: [まだ調査中の事項]
```

500語以内に収めること。記録ではなくキャッシュである。毎回完全に上書きする。

---

## Vault 規約

### 絶対に守ること

- **`.raw/` は変更禁止**: ソースドキュメントは読み取り専用。LLM は読むだけで書き込まない
- **`wiki/log.md` は追記専用**: 過去のエントリを編集しない。新しいエントリは先頭に追加
- **Obsidian Flavored Markdown**: ウィキリンクは `[[ノート名]]`、callout は `> [!type] タイトル`、埋め込みは `![[ファイル]]`、プロパティは YAML frontmatter
- **すべての wiki ページに frontmatter**: `type`・`status`・`created`・`updated`・`tags` を最低限記述
- **ウィキリンクはノート名のみ**: パス不要（ファイル名は Vault 内で一意）

### wiki ページ書き込み時の手順（並列 ingest の場合）

1. `bash scripts/wiki-lock.sh acquire <パス>` でページをロック
2. ページを書き込む
3. `bash scripts/wiki-lock.sh release <パス>` でロックを解放
4. stale lock は 60 秒で自動解放される

### Transport（ファイル書き込み方法）

`.vault-meta/transport.json` を参照してファイル書き込み方法を決定する（`bash scripts/detect-transport.sh` で生成）。利用可能な transport がない場合はファイルシステム（Write/Edit ツール）を使用する。

### Methodology Mode

`.vault-meta/mode.json` に設定された mode（generic / lyt / para / zettelkasten）に従って新規ページのパスを決定する。`python3 scripts/wiki-mode.py route <type> "<name>"` で正しいパスを取得する。

---

## index.md / log.md の更新

**ingest・scaffold・重大な変更のたびに必ず更新すること:**

- `wiki/index.md` — 全ページのマスターカタログ。新規ページ追加時に追記
- `wiki/log.md` — 全操作の時系列記録。新しいエントリを先頭に追加

---

## Kiro 固有の制約まとめ

| Claude Code 機能 | Kiro での代替 |
|----------------|-------------|
| SessionStart 自動フック | `wiki-session-start.json`（userTriggered・手動起動） |
| PostToolUse 自動コミット | `wiki-auto-commit.json`（fileEdited・ほぼ自動） |
| PostCompact 自動 hot.md 再読 | 文脈消失時にユーザーへ手動再読を促す |
| Stop フック（hot.md 自動更新） | セッション終了前にユーザーへ更新を促す |
| `/wiki` スラッシュコマンド | `#skill-wiki` 参照 + 自然言語 |
