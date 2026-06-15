---
inclusion: manual
---

# skill-wiki-mode: Vault の方法論モード設定

Vault の整理方法論（LYT / PARA / Zettelkasten / Generic）を設定し、ingest・save・autoresearch が新しいページをどこに配置するかを制御するスキル。`.vault-meta/mode.json` にモードを記録し、`scripts/wiki-mode.py` が他スキルにルーティングパスを返す。

トリガーワード: 「モードを設定して」「LYT に切り替えて」「PARA に変更して」「Zettelkasten を使いたい」「今のモードを教えて」「wiki mode」「methodology mode」

---

## 4つのモード

### Generic（デフォルト）

**考え方**: タイプ別にフォルダで分類する。v1.x のデフォルト動作を完全に踏襲。モードを設定していない場合も Generic として扱われる。

**ファイル配置規則**:
- ソース: `wiki/sources/<slug>.md`
- エンティティ: `wiki/entities/<Name>.md`
- コンセプト: `wiki/concepts/<Name>.md`
- セッション保存: `wiki/sessions/<YYYY-MM-DD>-<topic>.md`

**向いているケース**: 整理方法論を決めたくない、または v1.x から移行して動作を変えたくない場合。

---

### LYT（Linking Your Thinking — Nick Milo 式）

**考え方**: フォルダで整理しない。整理の単位は **MOC**（Map of Content）— トピッククラスターにリンクするハブノート。フォルダをブラウズするのではなく、リンクをたどってナビゲートする。

**ファイル配置規則**:
- MOC: `wiki/mocs/<topic>-moc.md`
- アトミックノート: `wiki/notes/<note-slug>.md`（少なくとも1つの MOC からリンクされる）

**向いているケース**: 100 ページ超の中〜大規模 Vault、概念クラスターで思考するユーザー、知識グラフ重視。

---

### PARA（Tiago Forte 式）

**考え方**: **行動可能性**で整理する。トピックではなく「今これに取り組んでいるか」で分類。

**ファイル配置規則**:
- `wiki/projects/<project-name>/<note>.md` — 締め切り・成果物がある進行中プロジェクト
- `wiki/areas/<area-name>/<note>.md` — 継続的な責務（締め切りなし）
- `wiki/resources/<topic>/<note>.md` — 参照資料（トピック別）
- `wiki/archives/<year>/<note>.md` — 完了・休止したもの

**向いているケース**: タスク管理重視のユーザー、多数のプロジェクトを同時管理するナレッジワーカー、GTD 実践者。

---

### Zettelkasten（Niklas Luhmann 式）

**考え方**: アトミックノート・ユニーク ID・密な双方向リンク。フォルダなし。各ノートはひとつのアイデアだけを扱う。ノートは ID 参照で互いを見つける。

**ファイル配置規則**:
- `wiki/<YYYYMMDDHHMMSSffffff>-<slug>.md`（タイムスタンプ20桁 + スラッグ、衝突耐性あり）
- フロントマター必須: `id:`、`parent_id:`（任意）、`child_ids:`（任意）
- サブディレクトリなし: `wiki/` ルートがすべて

**向いているケース**: 研究者・学者・長期的な知識アーティファクトを構築するユーザー。最も規律が必要。

---

## モードの設定方法

### インタラクティブ（推奨）

```bash
bash bin/setup-mode.sh
```

4つのモードから選択するプロンプトが表示される。選択後に `.vault-meta/mode.json` が書き込まれ、オプションでテンプレートフォルダが作成される。

### 非インタラクティブ

```bash
bash bin/setup-mode.sh --mode lyt
bash bin/setup-mode.sh --mode para
bash bin/setup-mode.sh --mode zettelkasten
bash bin/setup-mode.sh --mode generic
```

### 現在のモードを確認

```bash
bash bin/setup-mode.sh --check
# または
python3 scripts/wiki-mode.py get
```

---

## `.vault-meta/mode.json` スキーマ

```json
{
  "schema_version": 1,
  "mode": "lyt|para|zettelkasten|generic",
  "configured_at": "ISO-8601 タイムスタンプ",
  "config": {
    "lyt": {
      "moc_folder": "wiki/mocs/",
      "notes_folder": "wiki/notes/"
    },
    "para": {
      "projects_folder": "wiki/projects/",
      "areas_folder": "wiki/areas/",
      "resources_folder": "wiki/resources/",
      "archives_folder": "wiki/archives/"
    },
    "zettelkasten": {
      "id_format": "YYYYMMDDHHMMSSffffff",
      "no_folders": true,
      "root_folder": "wiki/"
    },
    "generic": {
      "sources_folder": "wiki/sources/",
      "entities_folder": "wiki/entities/",
      "concepts_folder": "wiki/concepts/",
      "sessions_folder": "wiki/sessions/"
    }
  }
}
```

`config` ブロックには常に4モード全ての設定を含む。アクティブなモードは `mode` フィールドで指定される。これによりモード切り替え時にカスタムフォルダ設定が失われない。

---

## 他スキルへのルーティング表

他スキル（skill-wiki-ingest・skill-save）は `python3 scripts/wiki-mode.py route <type> "<slug>"` を呼び出してファイルパスを取得する。

| コンテンツタイプ | Generic | LYT | PARA | Zettelkasten |
|---------------|---------|-----|------|-------------|
| ソース ingest | `wiki/sources/<slug>.md` | `wiki/notes/<slug>.md` + トピック MOC に追加 | `wiki/resources/<topic>/<slug>.md` | `wiki/<ID>-<slug>.md` |
| エンティティ | `wiki/entities/<Name>.md` | `wiki/notes/<Name>.md` + エンティティ MOC | `wiki/resources/people/<Name>.md` | `wiki/<ID>-<name>.md` |
| コンセプト | `wiki/concepts/<Name>.md` | `wiki/notes/<Name>.md` + コンセプト MOC | `wiki/resources/concepts/<Name>.md` | `wiki/<ID>-<name>.md` |
| セッション保存（save） | `wiki/sessions/<date>-<topic>.md` | `wiki/notes/<date>-<topic>.md` + セッション MOC | `wiki/projects/inbox/<date>-<topic>.md` | `wiki/<ID>-session-<topic>.md` |

`.vault-meta/mode.json` が存在しない場合、すべてのスキルは Generic として動作する（後方互換性）。

---

## フィーチャー検出イディオム（他スキルが参照するコード）

```bash
if [ -f .vault-meta/mode.json ]; then
  MODE=$(python3 -c 'import json; print(json.load(open(".vault-meta/mode.json"))["mode"])')
else
  MODE="generic"
fi
```

または `scripts/wiki-mode.py` 経由:

```bash
MODE=$(python3 scripts/wiki-mode.py get 2>/dev/null || echo "generic")
ROUTED_PATH=$(python3 scripts/wiki-mode.py route concept "MyConceptName")
```

---

## モード切り替えの注意事項

- **既存ファイルは自動マイグレーションされない**。新しいモードは切り替え以降に作成されるページにのみ適用される
- 切り替え後に古いパスのページが孤立する可能性がある → `#skill-wiki-lint` で孤立ページを検出・確認する
- 手動マイグレーションが必要な場合は `git mv` で移動し、wikilink を更新する
