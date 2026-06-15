---
inclusion: manual
---

# skill-wiki-lint: Wiki ヘルスチェックスキル

10〜15 回の ingest ごと、または週次で lint を実行します。何かを自動修正する前に必ず確認を取ります。lint レポートを `wiki/meta/lint-report-YYYY-MM-DD.md` に出力します。

---

## トランスポート

lint は主に読み取りを行い、最後にレポートファイルを1件書き込みます。

- **filesystem**（デフォルト・MVP）— Kiro の `Read` / `Glob` / `Grep` ツールを使用。常に利用可能
- **cli** — `obsidian-cli` がある場合は `obsidian-cli backlinks "$VAULT" "$NOTE"` でバックリンクグラフを取得可能（`Grep` より効率的）
- **mcp** — MCP サーバーがある場合は `read_multiple_notes`・`list_all_tags` を使用可能

---

## Lint チェック

以下の順に処理します:

1. **孤立ページ（Orphan）**。インバウンドのウィキリンクがない wiki ページ。存在するが何も指していない
2. **デッドリンク（Dead links）**。存在しないページへのウィキリンク参照
3. **古い主張（Stale claims）**。新しいソースが矛盾または更新したと考えられる古いページの主張
4. **欠落ページ（Missing pages）**。複数のページで言及されているが、独自のページがないコンセプトやエンティティ
5. **欠落クロスリファレンス（Missing cross-references）**。ページ内で言及されているが、リンクされていないエンティティ
6. **フロントマターの欠落（Frontmatter gaps）**。必須フィールド（type, status, created, updated, tags）が欠けているページ
7. **空セクション（Empty sections）**。内容のない見出し
8. **インデックスの古いエントリ（Stale index entries）**。名前変更または削除されたファイルを指す `wiki/index.md` のエントリ
9. **Address Validation**（DragonScale Mechanism 2、オプトイン）。下記セクション参照
10. **Semantic Tiling**（DragonScale Mechanism 3、オプトイン）。下記セクション参照

---

## Lint レポートフォーマット

`wiki/meta/lint-report-YYYY-MM-DD.md` に作成:

```markdown
---
type: meta
title: "Lint レポート YYYY-MM-DD"
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [meta, lint]
status: developing
---

# Lint レポート: YYYY-MM-DD

## サマリー
- スキャンしたページ数: N
- 見つかった問題: N
- 自動修正済み: N
- 要確認: N

## 孤立ページ
- [[ページ名]]: インバウンドリンクなし。提案: [[関連ページ]] からリンクするか削除

## デッドリンク
- [[欠落ページ]]: [[ソースページ]] で参照されているが存在しない。提案: スタブを作成またはリンクを削除

## 欠落ページ
- 「コンセプト名」: [[ページA]], [[ページB]], [[ページC]] で言及。提案: コンセプトページを作成

## フロントマターの欠落
- [[ページ名]]: 欠落フィールド: status, tags

## 古い主張
- [[ページ名]]: 主張「X」が新しいソース [[新しいソース]] と矛盾している可能性あり

## クロスリファレンスの欠落
- [[エンティティ名]] が [[ページA]] でウィキリンクなしに言及されている
```

---

## 命名規則

lint 中にこれらを強制します:

| 要素 | 規則 | 例 |
|-----|-----|---|
| ファイル名 | スペース付きタイトルケース | `Machine Learning.md` |
| フォルダ | ダッシュ付き小文字 | `wiki/data-models/` |
| タグ | 小文字、階層的 | `#domain/architecture` |
| ウィキリンク | ファイル名と完全一致 | `[[Machine Learning]]` |

ファイル名は vault 内で一意でなければならない。ウィキリンクはファイル名が一意の場合のみパスなしで機能する。

---

## 文書スタイルチェック

lint 中に、スタイルガイドに違反するページにフラグを立てます:

- 宣言的現在形でない（「X は基本的に Y をする」ではなく「X は Y をする」）
- 主張が行われているのにソース引用がない
- 不確実性が `> [!gap]` でフラグされていない
- 矛盾が `> [!contradiction]` でフラグされていない

---

## Dataview ダッシュボード

`wiki/meta/dashboard.md` を作成または更新します:

````markdown
---
type: meta
title: "ダッシュボード"
updated: YYYY-MM-DD
---
# Wiki ダッシュボード

## 最近のアクティビティ
```dataview
TABLE type, status, updated FROM "wiki" SORT updated DESC LIMIT 15
```

## シードページ（要開発）
```dataview
LIST FROM "wiki" WHERE status = "seed" SORT updated ASC
```

## ソースのないエンティティ
```dataview
LIST FROM "wiki/entities" WHERE !sources OR length(sources) = 0
```

## 未解決の質問
```dataview
LIST FROM "wiki/questions" WHERE answer_quality = "draft" SORT created DESC
```
````

---

## Canvas マップ

ビジュアルドメインマップとして `wiki/meta/overview.canvas` を作成または更新します:

```json
{
  "nodes": [
    {
      "id": "1",
      "type": "file",
      "file": "wiki/overview.md",
      "x": 0, "y": 0,
      "width": 300, "height": 140,
      "color": "1"
    }
  ],
  "edges": []
}
```

ドメインページごとに1ノードを追加します。重要なクロスリファレンスがあるドメインを接続します。色は CSS スキームにマップされます: 1=青, 2=紫, 3=黄, 4=オレンジ, 5=緑, 6=赤

---

## Address Validation（DragonScale Mechanism 2）

**オプトイン機能。** 以下で DragonScale を検出します:

```bash
if [ -x ./scripts/allocate-address.sh ] && [ -f ./.vault-meta/address-counter.txt ]; then
  DRAGONSCALE_ADDRESSES=1
else
  DRAGONSCALE_ADDRESSES=0
fi
```

`DRAGONSCALE_ADDRESSES=0` の場合、このセクション全体をスキップします。

`DRAGONSCALE_ADDRESSES=1` の場合、以下を実行します。

**ロールアウト基準日**: 2026-04-23（Phase 2 出荷日）。より新しく DragonScale を採用した vault は、対象ページの最初の `created:` 日付を基準日として使用します。選択した基準日を `.vault-meta/legacy-pages.txt` の先頭にコメント行として記録: `# rollout: YYYY-MM-DD`

### 分類ルール（ページごとに適用）

| 分類 | 基準 |
|-----|-----|
| **Meta/fold/除外** | `wiki/folds/` 以下のファイル、または `_index.md`, `index.md`, `log.md`, `hot.md`, `overview.md`, `dashboard.md` 等のファイル名。Address 不要 |
| **ロールアウト後（address 必須）** | `type` が meta/fold でなく、フロントマター `created:` が 2026-04-23 以降で、legacy manifest にパスがない |
| **レガシー（バックフィル対象）** | `type` が meta/fold でなく、`created:` が 2026-04-23 より前、またはパスが legacy manifest にある。バックフィルまで address 不要 |

**Legacy baseline manifest**: `.vault-meta/legacy-pages.txt`（オプション）。1行1パス。`created:` メタデータが間違っているページを grandfather するために使用。

### バリデーションチェック（順番に実行）

1. **フォーマットチェック**: `address:` が設定されているページは以下のいずれかに一致すること:
   - `^c-[0-9]{6}$`（ロールアウト後の作成アドレス）
   - `^l-[0-9]{6}$`（レガシーバックフィルアドレス）

2. **一意性チェック**: 同じ address 値を持つ2つのページがないこと。両方のパスを報告

3. **カウンター整合性**: `./scripts/allocate-address.sh --peek` が次のカウンター値を返す。観測されたすべての `c-NNNNNN` は `NNNNNN < peek_value` を満たすこと。違反 = カウンタードリフト

4. **ロールアウト後の強制**: 「ロールアウト後（address 必須）」に分類されるページで `address:` フィールドがないものは lint **エラー**（情報提供ではなくエラー）

5. **レガシー識別**: address がないレガシーページは情報提供。lint レポートで「バックフィル待ち」としてリスト（総数付き）

6. **address-map 整合性**（`.raw/.manifest.json`）: `address_map` の各ページパスは存在し、フロントマターの `address` がマッピングと一致すること。不一致はエラー

### Lint 姿勢サマリー

- 不正なフォーマットの address を持つページ: **エラー**
- 衝突する address を持つページ: **エラー**
- **ロールアウト後** で address なし: **エラー**
- **レガシー** で address なし: **情報提供**（想定内）
- address なしの Meta/fold ページ: **無視**
- カウンタードリフト（観測カウンター >= peek）: **エラー**
- address-map の不一致: **エラー**

lint は観察のみ。lint 中に欠落した address を自動割り当てしない。割り当ては `#skill-wiki-ingest` の責任のみ。

### レポートの出力セクション

```markdown
## Address Validation

- カウンター状態: `$(./scripts/allocate-address.sh --peek)`
- 観測された最高の c- アドレス: c-XXXXXX
- チェックされたロールアウト後ページ: N（X 通過、Y エラー）
- バックフィル待ちのレガシーページ: M

### エラー
- [[ページ名]]: 不正な address フォーマット `{value}`。`c-NNNNNN` または `l-NNNNNN` が期待される
- [[ページA]] と [[ページB]] が address `c-000042` を共有している
- [[ロールアウト後ページ]]: address なし。2026-04-25 作成（ロールアウト後）で address 必須。wiki-ingest を実行するか、`./scripts/allocate-address.sh` を手動実行してフロントマターに追加
- [[ページ名]] の address は `c-000100` だがカウンター peek は `50`。カウンタードリフト; `./scripts/allocate-address.sh --rebuild` を実行
- `.raw/.manifest.json` が `wiki/foo.md` -> `c-000010` にマップしているが、ページフロントマターは `c-000012`。不一致を解決

### バックフィル待ち（情報提供）
- M 件のレガシーページに address なし。正規のレガシーセットは `.vault-meta/legacy-pages.txt` を参照、または `created:` < 2026-04-23 でフィルター
```

---

## Semantic Tiling（DragonScale Mechanism 3）

**オプトイン機能。** 以下で検出します:

```bash
if [ -x ./scripts/tiling-check.py ] && command -v python3 >/dev/null 2>&1; then
  ./scripts/tiling-check.py --peek > /tmp/tiling-peek.json 2>/dev/null
  PEEK_EXIT=$?
  case $PEEK_EXIT in
    0)  TILING_READY=1 ;;
    10) TILING_READY=0 ; echo "tiling スキップ: ollama に接続できません（exit 10）" ;;
    11) TILING_READY=0 ; echo "tiling スキップ: 'ollama pull nomic-embed-text' を実行してください（exit 11）" ;;
    *)  TILING_READY=0 ; echo "tiling エラー: tiling-check.py --peek が exit $PEEK_EXIT で終了" ;;
  esac
else
  TILING_READY=0
  echo "tiling スキップ: scripts/tiling-check.py または python3 が利用できません（MVP 未実装）"
fi
```

**注意**: `scripts/tiling-check.py` は MVP では未実装です。`TILING_READY=0` になります。

`TILING_READY=1` の場合:

```bash
./scripts/tiling-check.py --report wiki/meta/tiling-report-YYYY-MM-DD.md
```

exit コード 0/2/3/4/10/11 をすべて個別に表示。「不明」にまとめない。

レポートへの埋め込み:

```markdown
## Semantic Tiling
詳細なペアリストは [[tiling-report-YYYY-MM-DD]] を参照。
- エラー（>=0.90）: N ペア
- 要確認（0.80-0.90）: M ペア
- キャリブレーション済み: true|false
```

---

## 自動修正の前に

常に lint レポートを先に表示します。「自動修正しますか、それとも1つずつ確認しますか？」と確認します。

自動修正して安全なもの:
- プレースホルダー値で欠落したフロントマターフィールドを追加
- 欠落したエンティティのスタブページを作成
- リンクされていない言及にウィキリンクを追加

修正前に要確認のもの:
- 孤立ページの削除（意図的に孤立している可能性がある）
- 矛盾の解消（人間の判断が必要）
- 重複ページのマージ
