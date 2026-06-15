---
inclusion: manual
---

# skill-save: 会話を wiki ページとして保存する

セッション中の重要な洞察・回答・意思決定を、構造化された wiki ページとして永続化するスキル。会話内容を分析してノートタイプを自動判定し、frontmatter 付きのページを作成し、index・log・hot.md を更新する。

トリガーワード: 「保存して」「wiki に残して」「この回答を保存」「メモしておいて」「ファイリングして」「このセッションを保存」「wiki に追加して」

wiki は複利で増える。積極的に保存すること。

---

## ノートタイプの判定

会話の内容から最適なタイプを選ぶ。ユーザーがタイプを指定した場合はそれを優先する。

| タイプ | 保存先（Generic モード） | 使うケース |
|--------|------------------------|-----------|
| synthesis | `wiki/questions/` | 複数ステップの分析・比較・特定の問いへの回答 |
| concept | `wiki/concepts/` | アイデア・パターン・フレームワークの説明・定義 |
| source | `wiki/sources/` | セッションで議論した外部資料のサマリー |
| decision | `wiki/meta/` | アーキテクチャ・プロジェクト・戦略的な意思決定 |
| session | `wiki/meta/` | セッション全体のサマリー（議論したことをすべて記録） |

迷ったときは `synthesis` を使う。

---

## Transport

ファイルシステムへの直接書き込みを使用する（`Write` ツール）。パスは常に vault ルートからの相対パスで指定する（例: `wiki/questions/my-analysis.md`）。

---

## Concurrency（wiki-lock 連携）

複数のエージェントが同時に書き込む可能性があるため、ページ作成前に wiki-lock を取得する:

```bash
NOTE_PATH="wiki/questions/<slug>.md"
bash scripts/wiki-lock.sh acquire "$NOTE_PATH" || {
  echo "スキップ: $NOTE_PATH は別のライターがロック中"; exit 0
}
# … Write ツールでページを作成 …
bash scripts/wiki-lock.sh release "$NOTE_PATH"
```

複数ファイルを保存する場合（ページ + index + log 更新など）は、パスをアルファベット順にソートしてロックを取得することでデッドロックを防ぐ。

---

## モード連携（wiki-mode.py）

`.vault-meta/mode.json` が存在する場合、ページの保存先をモードに合わせてルーティングする:

```bash
# セッション保存のパスを取得
ROUTED_PATH=$(python3 scripts/wiki-mode.py route session "<topic-summary>")
```

`.vault-meta/mode.json` が存在しない場合は Generic モードとして動作する（後方互換性）。

モードごとのセッション保存先:

| モード | 保存先 |
|--------|--------|
| generic | `wiki/sessions/<YYYY-MM-DD>-<topic>.md` |
| lyt | `wiki/notes/<YYYY-MM-DD>-<topic>.md` + セッション MOC に追加 |
| para | `wiki/projects/inbox/<YYYY-MM-DD>-<topic>.md` |
| zettelkasten | `wiki/<YYYYMMDDHHMMSSffffff>-session-<topic>.md` |

---

## 保存ワークフロー（10ステップ）

### Step 1: 会話をスキャンする

会話全体を読み、保存する価値のある内容を特定する。そのまま書き起こすのではなく、最も価値の高い洞察・決定・合成を抽出する。

### Step 2: ノート名を決める

まだ名前がついていない場合: 「何というノート名にしますか？」と確認する。短く・説明的に。

### Step 3: ノートタイプを決める

上記の判定表を参照してタイプを選ぶ。

### Step 4: モードルーティングでパスを決める

```bash
python3 scripts/wiki-mode.py route <type> "<slug>"
```

モードが generic またはファイルが存在しない場合はデフォルトパスを使用。

### Step 5: wiki-lock を取得する

```bash
bash scripts/wiki-lock.sh acquire "<note-path>"
```

### Step 6: ページを作成する

`Write` ツールで `<note-path>` にページを作成する。frontmatter テンプレート（下記参照）を使用する。

- 同じパスのファイルがすでに存在する場合は、上書きする前にユーザーに確認する
- 宣言的・現在形で記述する（「ユーザーが質問した」ではなく、内容そのものを書く）

### Step 7: 関連リンクを収集する

会話で言及された wiki ページを特定し、frontmatter の `related:` に追加する。

### Step 8: `wiki/index.md` を更新する

該当セクションの先頭に新しいエントリを追加する:

```markdown
- [[Note Title]] — [1行の説明]（status: developing）
```

### Step 9: `wiki/log.md` に追記する

先頭に新しいエントリを追加する（追記専用・編集禁止）:

```markdown
## [YYYY-MM-DD] save | Note Title
- Type: [ノートタイプ]
- Location: wiki/[folder]/Note Title.md
- From: [セッションの簡単な説明]
```

### Step 10: `wiki/hot.md` を更新する

新しいページの追加を反映する。「最近作成・更新されたページ」セクションを更新する。

完了後: wiki-lock を解放し、「[[Note Title]] を wiki/[folder]/ に保存しました」と報告する。

---

## frontmatter テンプレート

```yaml
---
type: <synthesis|concept|source|decision|session>
title: "ノートのタイトル"
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags:
  - <関連タグ>
status: developing
related:
  - "[[言及された wiki ページ]]"
sources:
  - "[[.raw/source-if-applicable.md]]"
---
```

`question` タイプの場合は追加:
```yaml
question: "元の問い（そのまま）"
answer_quality: solid
```

`decision` タイプの場合は追加:
```yaml
decision_date: YYYY-MM-DD
status: active
```

---

## 文体ガイド

- **宣言的・現在形**で書く。知識そのものを書く。会話の記録を書かない
- NG: 「ユーザーが X について質問し、Kiro が説明した...」
- OK: 「X は Y によって機能する。重要な洞察は Z である。」
- 将来のセッションでこのページを初見で読んでも理解できる内容にする
- 言及したコンセプト・エンティティ・wiki ページにはすべて wikilink を付ける
- 該当する場合はソースを引用する: `（ソース: [[ページ名]]）`

---

## 保存すべき内容・スキップすべき内容

**保存する**:
- 自明でない洞察・合成
- 根拠を伴う意思決定
- 多くの労力を要した分析
- 再参照される可能性の高い比較・リサーチ結果

**スキップする**:
- 機械的な Q&A（自明な答えの検索質問）
- すでに別の場所に文書化されたセットアップ手順
- 永続的な洞察がない一時的なデバッグセッション
- すでに wiki にある内容（重複を作らない → 既存ページを更新する）
