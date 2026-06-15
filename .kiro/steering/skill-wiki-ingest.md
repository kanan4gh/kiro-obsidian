---
inclusion: always
---

# skill-wiki-ingest: ソース取り込み（Ingest）

ソースを読む。wiki に書く。すべてをクロスリファレンスする。1ソースから通常 8〜15 の wiki ページが生成される。

**構文標準**: すべての Obsidian Markdown は Obsidian Flavored Markdown で記述する。ウィキリンクは `[[ノート名]]`、callout は `> [!type] タイトル`、埋め込みは `![[ファイル]]`、プロパティは YAML frontmatter。

---

## Transport（書き込み方法）

vault ファイルを変更する前に `.vault-meta/transport.json` を確認する（`bash scripts/detect-transport.sh` で自動生成）。`preferred` の transport を使う:

- **filesystem** — Claude の `Write`/`Edit` ツールで vault ルートからの絶対パスを指定（常に使える最終フォールバック）
- **mcp-obsidian** / **mcpvault** — MCP が設定されている場合は `mcp__obsidian-vault__write_note` 等を使用（オプション）

`.vault-meta/transport.json` が存在しない場合は filesystem を使う。

---

## Methodology Mode 対応

新規 wiki ページを作成する前に、Vault の methodology mode を確認する:

```bash
SRC_PATH=$(python3 scripts/wiki-mode.py route source "Karpathy 2025 LLM Wiki essay")
# generic:      wiki/sources/Karpathy-2025-LLM-Wiki-essay.md
# lyt:          wiki/notes/Karpathy-2025-LLM-Wiki-essay.md
# para:         wiki/resources/incoming/Karpathy-2025-LLM-Wiki-essay.md
# zettelkasten: wiki/20260517123456-Karpathy-2025-LLM-Wiki-essay.md

ENT_PATH=$(python3 scripts/wiki-mode.py route entity "Andrej Karpathy")
CON_PATH=$(python3 scripts/wiki-mode.py route concept "Compounding Vault Pattern")
```

`.vault-meta/mode.json` が存在しない場合は mode=generic のパスが返される（v1.7 以前の動作と同一）。

**mode 別の追加作業**:
- **LYT**: atomic note を保存後、関連 MOC（`wiki/mocs/<topic>-moc.md`）を更新してリンクを追加。MOC が存在しない場合は `_templates/moc-template.md` から作成
- **Zettelkasten**: ファイル名にタイムスタンプ ID が含まれる。frontmatter の `id:` フィールドに同じ値を設定
- **PARA**: 新規 ingest は `wiki/resources/incoming/` に格納。ユーザーがトピックを決定するまで自動分類しない

---

## 並列書き込み安全手順（wiki-lock）

**v1.7+ では wiki-lock が必須**。ページへの書き込みは必ず `wiki-lock acquire` を先行させること。

```bash
# ロック取得 — 別のライターが保持中は75（EX_TEMPFAIL）を返す
if bash scripts/wiki-lock.sh acquire wiki/concepts/Foo.md; then
  # Transport で選択した方法でページを書き込む
  bash scripts/wiki-lock.sh release wiki/concepts/Foo.md
else
  # rc=75: 別ライターが書き込み中。2秒後にリトライ1回
  sleep 2
  bash scripts/wiki-lock.sh acquire wiki/concepts/Foo.md && {
    # 書き込む
    bash scripts/wiki-lock.sh release wiki/concepts/Foo.md
  } || echo "wiki/concepts/Foo.md をスキップ（ロック保持中）。wiki/log.md に記録"
fi
```

**特性**:
- **ファイル単位**: 異なるページへの書き込みはロック競合なしに並列実行可能
- **stale 解放**: デフォルト 60 秒で自動解放。クラッシュしたライターも 60 秒以内に解放される
- **クロスプロセス解放**: `rm -f` なので PID 不問。`wiki-lock clear-stale --max-age 0` がリカバリーパス
- **auto-commit との連携**: `wiki-auto-commit.json` はロック保持中の場合 git add をスキップ（torn commit 防止）

---

## デルタトラッキング

ingest 前に `.raw/.manifest.json` を確認して再処理を避ける。

```bash
[ -f .raw/.manifest.json ] && echo "存在する" || echo "マニフェストなし"
```

**マニフェストフォーマット**（存在しない場合は作成）:
```json
{
  "sources": {
    ".raw/articles/article-slug-2026-04-08.md": {
      "hash": "abc123",
      "ingested_at": "2026-04-08",
      "pages_created": ["wiki/sources/article-slug.md", "wiki/entities/Person.md"],
      "pages_updated": ["wiki/index.md"]
    }
  }
}
```

**ingest 前の確認手順**:
1. ハッシュを計算: `md5sum [ファイル] | cut -d' ' -f1`（Linux は `sha256sum`）
2. `.manifest.json` にそのパスと同じハッシュが存在するか確認
3. 一致する場合はスキップ。「取り込み済み（変更なし）。`force` で強制再取り込み可能。」と報告
4. 存在しないかハッシュが異なる場合は ingest を実行

**ingest 後の更新手順**:
1. `{hash, ingested_at, pages_created, pages_updated}` を `.manifest.json` に記録
2. マニフェストを書き戻す

ユーザーが "force ingest" または "再取り込み" と言った場合はデルタチェックをスキップする。

---

## URL Ingest

トリガー: ユーザーが `https://` で始まる URL を渡した場合。

手順:

1. **取得**: WebFetch ツールでページを取得する
2. **クリーニング**（オプション）: `defuddle` が利用可能な場合（`which defuddle 2>/dev/null`）、`defuddle [url]` を実行して広告・ナビゲーション・不要コンテンツを除去する（トークン 40〜60% 節約）。利用不可の場合は WebFetch の出力をそのまま使う
3. **スラッグ生成**: URL パスの末尾セグメントから生成（小文字化・スペース→ハイフン・クエリ文字列削除）
4. **保存**: `.raw/articles/[スラッグ]-[YYYY-MM-DD].md` に以下の frontmatter 付きで保存:
   ```markdown
   ---
   source_url: [url]
   fetched: [YYYY-MM-DD]
   ---
   ```
5. 「単一ソース Ingest」のステップ2から続行する

---

## 画像 Ingest

トリガー: ユーザーが画像ファイルパス（`.png`・`.jpg`・`.jpeg`・`.gif`・`.webp`・`.svg`・`.avif`）を渡した場合。

手順:

1. **読み込み**: Read ツールで画像ファイルを読み込む（Claude はネイティブに画像を処理できる）
2. **説明生成**: 画像の内容を説明する。テキストを OCR し、概念・エンティティ・図・データを識別する
3. **保存**: `.raw/images/[スラッグ]-[YYYY-MM-DD].md` に保存:
   ```markdown
   ---
   source_type: image
   original_file: [元のパス]
   fetched: YYYY-MM-DD
   ---
   # 画像: [スラッグ]

   [画像内容の完全な説明、転写テキスト、エンティティ等]
   ```
4. 画像が Vault 内になければ `_attachments/images/[スラッグ].[拡張子]` にコピーする
5. 保存した説明ファイルに対して「単一ソース Ingest」を実行する

用途: ホワイトボード写真・スクリーンショット・図・インフォグラフィック・文書スキャン

---

## 単一ソース Ingest

トリガー: ユーザーがファイルを `.raw/` に置いた、またはコンテンツを貼り付けた場合。

手順:

1. **読む**: ソースを完全に読む。流し読みしない
2. **議論**: 重要なポイントをユーザーと確認する。「強調すべき点は？粒度は？」と聞く。ユーザーが「そのまま取り込んで」と言った場合はスキップ
3. **ソースページ作成**: `python3 scripts/wiki-mode.py route source "<スラッグ>"` で得たパスに保存。frontmatter の schema は以下を参照:
   ```yaml
   ---
   type: source
   title: "[タイトル]"
   status: processed
   created: YYYY-MM-DD
   updated: YYYY-MM-DD
   tags: []
   source_file: .raw/articles/[ファイル名]
   ---
   ```
4. **エンティティページ作成・更新**: 言及されているすべての人物・組織・製品・リポジトリに対して1ページずつ作成または更新する。`python3 scripts/wiki-mode.py route entity "<名前>"` でパスを取得
5. **コンセプトページ作成・更新**: 重要なアイデア・フレームワーク・概念に対して作成または更新する。`python3 scripts/wiki-mode.py route concept "<名前>"` でパスを取得
6. **ドメインページ更新**: 関連するドメインページと `_index.md` サブインデックスを更新する
7. **`wiki/overview.md` 更新**: 全体像が変わった場合に更新する
8. **`wiki/index.md` 更新**: 新規作成したすべてのページのエントリを追加する
9. **`wiki/hot.md` 更新**: この ingest の文脈を反映して更新する
10. **`wiki/log.md` に追記**（新しいエントリを**先頭**に追加）:
    ```markdown
    ## [YYYY-MM-DD] ingest | ソースタイトル
    - ソース: `.raw/articles/ファイル名.md`
    - サマリー: [[ソースタイトル]]
    - 作成ページ: [[ページ1]]、[[ページ2]]
    - 更新ページ: [[ページ3]]、[[ページ4]]
    - 主要な洞察: 新しく得られた最も重要な知見を1文で
    ```
11. **矛盾チェック**: 新しい情報が既存ページと矛盾する場合は `> [!contradiction]` callout を両方のページに追加する

---

## バッチ Ingest

トリガー: ユーザーが複数ファイルを渡した、または "すべて取り込んで" と言った場合。

手順:

1. 処理するファイルをリストアップし、開始前にユーザーに確認する
2. **エージェントを並列起動**: `.kiro/agents/wiki-ingest.md` を参照してサブエージェントを起動する（1エージェント = 1ソース）。各エージェントは wiki-lock を取得してページを書き込む
3. **全エージェント完了後**: ソース間のクロスリファレンスパスを実行する
4. `wiki/index.md`・`wiki/hot.md`・`wiki/log.md` をまとめて1回更新する（ソースごとではなく）
5. 報告: 「N 件のソースを処理しました。X ページを作成、Y ページを更新。主な接続: ...」

バッチ ingest はインタラクティブ性が低い。30件以上の場合は 10 件ごとにユーザーに進捗報告する。

---

## コンテキストウィンドウ節約

ingest 中は以下のルールに従うこと:

- `wiki/hot.md` を最初に読む。関連する文脈がそこにあれば、ページ全体を再読しない
- `wiki/index.md` を読んで既存ページを確認してから新規作成する
- 1回の ingest で読む既存ページは 3〜5 件まで。10件以上必要な場合は読みすぎている
- 外科的な編集には PATCH を使う。1フィールド更新のためにファイル全体を再読しない
- wiki ページは短く保つ。最大 100〜300 行。それを超えたらページを分割する

---

## 矛盾検出

新しい情報が既存の wiki ページと矛盾する場合:

**既存ページに追加**:
```markdown
> [!contradiction] [[新ソース]] との矛盾
> [[既存ページ]] では X と主張している。[[新ソース]] では Y と述べている。
> 解決が必要。日付・文脈・一次資料を確認すること。
```

**新ソースサマリーに追加**:
```markdown
> [!contradiction] [[既存ページ]] との矛盾
> このソースでは Y と述べているが、既存の wiki では X としている。詳細は [[既存ページ]] を参照。
```

古い主張を黙って上書きしないこと。フラグを立ててユーザーに判断させる。

> **注**: `[!contradiction]` callout は `.obsidian/snippets/vault-colors.css`（`#skill-wiki` の scaffold で自動作成）に定義されたカスタム callout。スニペットがない場合は標準 callout スタイルにフォールバックする。

---

## 禁止事項

- **`.raw/` 内のファイルは変更禁止**: ユーザーが置いたファイルは読み取り専用。`.raw/.manifest.json` のみ wiki-ingest が更新してよい
- 重複ページを作らない。作成前に必ず index と検索で確認する
- log エントリをスキップしない。すべての ingest を記録する
- hot cache の更新をスキップしない。次のセッションを高速にするために必須

---

## Address Assignment（DragonScale Mechanism 2・opt-in）

**opt-in 機能**。`scripts/allocate-address.sh` が存在し、かつ `.vault-meta/` が存在する場合のみ実行する。

**機能検出（ingest 開始時に実行）**:

```bash
if [ -x ./scripts/allocate-address.sh ] && [ -d ./.vault-meta ]; then
  DRAGONSCALE_ADDRESSES=1
else
  DRAGONSCALE_ADDRESSES=0
fi
```

`DRAGONSCALE_ADDRESSES=0` の場合: `address:` frontmatter フィールドなしでページを作成する。

`DRAGONSCALE_ADDRESSES=1` の場合: 以下の手順に従う。

**新規非メタページにはアドレスを付与**:

```yaml
address: c-000042
```

フォーマット: `c-<6桁カウンター>`。ゼロパディング。

**アドレス割り当て手順（新規ページごと）**:

1. `./scripts/allocate-address.sh` を呼び出してアドレスを取得する
2. ページの frontmatter に `address: c-XXXXXX` を含める
3. `.raw/.manifest.json` の `address_map` にパスとアドレスのマッピングを記録する

**`address_map` のフォーマット**（`.raw/.manifest.json` 内）:
```json
{
  "sources": { "...": "..." },
  "address_map": {
    "wiki/concepts/Example.md": "c-000042",
    "wiki/entities/Another.md": "c-000043"
  }
}
```

**アドレス付与の除外対象**:
- メタファイル: `_index.md`・`index.md`・`log.md`・`hot.md`・`overview.md`・`dashboard.md`
- `wiki/folds/` 配下のファイル
- `created: < 2026-04-23` のレガシーページ

**冪等性ルール**:
- 書き込もうとしているページにすでに `address:` フィールドがある場合は再利用する
- `address_map` にそのパスのマッピングがある場合は再利用する

**並列 ingest のサブエージェントは `allocate-address.sh` を呼び出し禁止**。アドレスはオーケストレーターが全サブエージェント完了後に一括で付与する。
