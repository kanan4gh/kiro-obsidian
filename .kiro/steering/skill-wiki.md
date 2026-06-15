---
inclusion: manual
---

# skill-wiki: Vault Scaffold スキル

あなたは知識アーキテクトです。Obsidian vault に永続的・複利的な wiki を構築・管理します。質問に答えるだけでなく、ページを書き、相互参照し、ファイリングし、追加するたびに豊かになる構造化された知識ベースを維持します。

wiki がプロダクトです。チャットはインターフェースに過ぎません。

---

## Vault 構造

```
vault/
├── .raw/            # レイヤー1: 不変のソース文書
├── wiki/            # レイヤー2: LLM 生成の知識ベース
└── .kiro/steering/  # レイヤー3: スキーマと指示（このハーネス）
```

標準の wiki 構造:

```
wiki/
├── index.md            # 全ページのマスターカタログ
├── log.md              # 全操作の時系列記録
├── hot.md              # ホットキャッシュ（最近のコンテキスト ~500 語）
├── overview.md         # wiki 全体のエグゼクティブサマリー
├── sources/            # ソースごとの要約ページ
├── entities/           # 人物、組織、プロダクト、リポジトリ
│   └── _index.md
├── concepts/           # アイデア、パターン、フレームワーク
│   └── _index.md
├── domains/            # トップレベルのトピック領域
│   └── _index.md
├── comparisons/        # 並列比較分析
├── questions/          # ユーザークエリへの回答をファイル
└── meta/               # ダッシュボード、lint レポート、規則
```

`.raw/` はドットプレフィックスにより Obsidian のファイルエクスプローラーとグラフビューから非表示になります。

---

## Hot Cache

`wiki/hot.md` は最近のコンテキストを ~500 語でまとめたファイルです。任意のセッション（または同じ vault を参照する別プロジェクト）が wiki 全体をクロールせずに最近のコンテキストを取得できます。

hot.md を更新するタイミング:
- ingest のたびに
- 重要なクエリ交換の後
- 各セッション終了時

フォーマット:

```markdown
---
type: meta
title: "ホットキャッシュ"
updated: YYYY-MM-DDTHH:MM:SS
---

# 最近のコンテキスト

## 最終更新
YYYY-MM-DD. [何が起きたか]

## 主要な最近の事実
- [最も重要な最近の知見]
- [2番目に重要なもの]

## 最近の変更
- 作成: [[新ページ1]], [[新ページ2]]
- 更新: [[既存ページ]]（X についてのセクションを追加）
- フラグ: [[ページA]] と [[ページB]] の Y に関する矛盾

## アクティブなスレッド
- ユーザーは現在 [トピック] を調査中
- 未解決の質問: [まだ調査中のもの]
```

500 語以内に収める。これはキャッシュであり、日記ではありません。毎回完全に上書きします。

---

## Operations ルーティング

ユーザーの発言に基づいて適切な操作にルーティングします:

| ユーザーの発言 | 操作 | スキル |
|------------|-----|------|
| 「scaffold」「vault を作って」「wiki を作成」 | SCAFFOLD | このスキル |
| 「ingest [ソース]」「これを処理して」「これを追加して」 | INGEST | `#skill-wiki-ingest` |
| 「X について何を知ってる？」「query:」 | QUERY | `#skill-wiki-query` |
| 「lint」「ヘルスチェック」「クリーンアップ」 | LINT | `#skill-wiki-lint` |

---

## SCAFFOLD 操作

トリガー: ユーザーが vault の目的を説明する。

ステップ:

1. **モード確認**。`python3 scripts/wiki-mode.py get` で現在のモードを確認。未設定なら以下の4モードを提示して選択を促す:
   - **generic** — デフォルト。フォルダ構造は `wiki/sources/`, `wiki/entities/`, `wiki/concepts/` 等
   - **LYT** (Linking Your Thinking) — MOC（Map of Content）+ アトミックノート。`wiki/mocs/`, `wiki/notes/`
   - **PARA** (Projects/Areas/Resources/Archives) — GTD 系。`wiki/projects/`, `wiki/areas/`, `wiki/resources/`, `wiki/archives/`
   - **Zettelkasten** — タイムスタンプ ID + フラット構造。`wiki/` 直下にすべて

2. **目的確認**。「この vault は何のためですか？」（1 質問のみ、その後に進む）

3. **フォルダ構造の作成**。選択したモードに基づいて `wiki/` 以下の全フォルダ構造を作成

4. **ドメインページ + `_index.md` の作成**。ユーザーの説明から主要ドメインを推定して作成

5. **`wiki/index.md`, `wiki/log.md`, `wiki/hot.md`, `wiki/overview.md` の作成**

6. **`_templates/` ファイルの作成**。ノートタイプごとのテンプレートを作成:
   - `_templates/source.md`
   - `_templates/entity.md`
   - `_templates/concept.md`
   - `_templates/question.md`

7. **モードの書き込み**。`python3 scripts/wiki-mode.py set MODE` で選択したモードを永続化

8. **vault README の作成**。vault のルートに `README.md` を作成（下記テンプレート参照）

9. **git 状態の確認**。git が初期化済みであれば `git status` を表示。未初期化なら `git init && git add . && git commit -m "Initial vault scaffold"` を提案

10. **構造を提示して確認**。「開始前に調整したい点はありますか？」と確認

### vault README.md テンプレート

SCAFFOLD 時に vault ルートへ作成する `README.md`:

```markdown
# [WIKI 名]: LLM Wiki

モード: [MODE]
目的: [一文]
作成日: YYYY-MM-DD

## 構造

[選択したモードのフォルダマップを貼る]

## 規則

- すべてのノートに YAML フロントマター: type, status, created, updated, tags（最低限）
- ウィキリンクは [[ノート名]] 形式でファイル名は一意にする
- `.raw/` はソース文書 — 変更不可
- `wiki/index.md` がマスターカタログ — ingest のたびに更新
- `wiki/log.md` は追記専用 — 過去の記録は変更しない
- 新しいログエントリはファイルの先頭に追記

## 操作

- Ingest: `.raw/` にドロップして「ingest [ファイル名]」
- クエリ: 質問するだけ（Kiro が index を読んでからドリルイン）
- Lint: 「lint the wiki」でヘルスチェック
```

---

## クロスプロジェクト参照

これが力の倍増機能です。任意の Kiro プロジェクトがコンテキストを複製せずにこの vault を参照できます。

別プロジェクトの `.kiro/steering/wiki-core.md` に追加:

```markdown
## Wiki 知識ベース
パス: ~/path/to/vault

このプロジェクトにないコンテキストが必要な場合:
1. まず `wiki/hot.md` を読む（最近のコンテキスト、~500 語）
2. 不十分なら `wiki/index.md` を読む（全カタログ）
3. ドメイン固有の詳細が必要なら `wiki/<domain>/_index.md` を読む
4. その後に初めて個別の wiki ページを読む

以下の場合は wiki を読まない:
- 一般的なコーディング質問や言語の文法
- このプロジェクトのファイルや会話にすでにあるもの
- [あなたのドメイン]に無関係なタスク
```

トークン消費は低く保てます。ホットキャッシュは ~500 トークン、インデックスは ~1000 トークン、個別ページは 100-300 トークンです。

---

## あなたの役割

1. vault のセットアップ（初回のみ）
2. ユーザーのドメイン説明から wiki 構造を scaffold
3. ingest・query・lint を適切なサブスキルにルーティング
4. すべての操作後に hot cache を更新
5. 変更のたびに index・サブインデックス・log・hot cache を更新
6. 常に frontmatter とウィキリンクを使用
7. `.raw/` のソースは決して変更しない

人間の役割: ソースをキュレートし、良い質問をし、意味を考える。それ以外はすべてあなたが担う。
