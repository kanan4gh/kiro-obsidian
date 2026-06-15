# kiro-obsidian ハーネス運用ガイド

このファイルは **ハーネス管理者**（kiro-obsidian を自分の環境向けにカスタマイズ・運用する人）向けの運用ガイドです。

---

## ハーネス構成（3層構造）

```
┌─────────────────────────────────────────────┐
│  コンテキスト層（.kiro/steering/）            │
│  Kiro が「何をすべきか」を常時・手動で注入     │
├─────────────────────────────────────────────┤
│  自動化層（.kiro/hooks/ + .kiro/agents/）    │
│  イベント駆動の自動化とサブエージェント並列化 │
├─────────────────────────────────────────────┤
│  ツール層（scripts/ + git）                  │
│  Kiro が呼び出すスクリプト・ファイルシステム  │
└─────────────────────────────────────────────┘
```

### `.kiro/steering/` — コンテキスト層

| ファイル | inclusion | 役割 |
|---------|-----------|------|
| `wiki-core.md` | always | Vault 規約・hot cache・操作ルーティング |
| `skill-wiki-ingest.md` | always | ingest 手順（最頻操作のため常時ロード） |
| `skill-wiki.md` | manual | Vault scaffold 手順 |
| `skill-wiki-query.md` | manual | クエリ手順 |
| `skill-wiki-lint.md` | manual | lint 手順 |
| `skill-wiki-mode.md` | manual | Vault 整理方法論の選択・設定（LYT / PARA / Zettelkasten / Generic） |
| `skill-save.md` | manual | 会話の洞察を wiki ページとして保存 |
| `skill-autoresearch.md` | manual | Web 検索・ソース取得・wiki 登録の自律リサーチループ |

`inclusion: always` のファイルはセッション開始時に自動的に Kiro のコンテキストに注入されます。
`inclusion: manual` のファイルは `#skill-name` でチャット内から参照したときのみ注入されます。

### `.kiro/hooks/` — 自動化層

| ファイル | トリガー | 動作 |
|---------|---------|------|
| `wiki-auto-commit.json` | wiki/ または .raw/ のファイル編集後 | 自動 git コミット |
| `wiki-session-start.json` | ユーザートリガー | セッション開始時の hot.md 読み込み |

### `.kiro/agents/` — サブエージェント

| ファイル | 役割 |
|---------|------|
| `wiki-ingest.md` | バッチ ingest の並列処理 |
| `wiki-lint.md` | lint チェックの並列処理 |

---

## セッション開始時の流れ

1. Kiro が `wiki-core.md`（always）と `skill-wiki-ingest.md`（always）を自動読み込み
2. `wiki/hot.md` が存在する場合、Kiro は自動的にその内容を把握した状態でセッションを開始
3. 前のセッションの続きから即座に作業できる

---

## 基本的な wiki 操作

### ingest

```
[ファイル名] を ingest してください
```

または複数ファイル:

```
.raw/ のすべての未処理ファイルを ingest してください
```

### query

```
[トピック] について wiki から教えてください
```

詳細な検索が必要な場合:

```
#skill-wiki-query を参照して、[質問] に詳しく答えてください
```

### lint

```
#skill-wiki-lint を参照して wiki を lint してください
```

### hot cache 更新

```
hot cache を更新してください
```

---

## カスタマイズ

### Vault のドメインを変更する

`wiki/overview.md` の「目的」セクションを自分のドメインに合わせて書き換えます:

```markdown
## 目的

これは [あなたのプロジェクト名] の知識 Vault です。
[ドメインの説明] に関するソースを取り込み、...
```

### wiki-core.md をカスタマイズする

`.kiro/steering/wiki-core.md` にカスタム規約を追加できます:

- 特定のタグ規約
- wiki ページの優先分類カテゴリ
- ドメイン固有の命名規則

### 自動コミットを無効化する

auto-commit を一時的に無効化するには:

```bash
touch .vault-meta/auto-commit.disabled
```

再び有効化:

```bash
rm .vault-meta/auto-commit.disabled
```

---

## トラブルシューティング

### auto-commit が動かない

`.kiro/hooks/wiki-auto-commit.json` を確認し、Kiro の Hooks 設定が有効になっているか確認してください。

また、`scripts/wiki-lock.sh` がロックを保持中の場合はコミットが延期されます:

```bash
bash scripts/wiki-lock.sh list
bash scripts/wiki-lock.sh clear-stale --max-age 3600
```

### ingest 後に index.md が更新されない

Kiro のコンテキストに `skill-wiki-ingest.md` が読み込まれているか確認してください。
新しいセッションを開始するか、`wiki-core.md` を確認してください。

### wiki ページが日本語で生成されない

`wiki-core.md` の「言語規約」セクションに「すべての wiki ページは日本語で記述する」という指示が含まれているか確認してください。

---

## スクリプトのテスト

```bash
bash tests/test_wiki_lock.sh
python3 tests/test_wiki_mode.py
```

---

## バージョン情報

このハーネスは [claude-obsidian](https://github.com/AI-Marketing-Hub/claude-obsidian) v1.9 相当の P0 コア機能を Kiro IDE 向けに移植したものです。

| 機能 | ステータス |
|------|---------|
| wiki scaffold | ✅ P0 |
| ingest | ✅ P0 |
| query | ✅ P0 |
| lint | ✅ P0 |
| Hot Cache | ✅ P0 |
| auto-commit | ✅ P0 |
| Methodology Mode | ✅ P1 |
| save | ✅ P1 |
| autoresearch | ✅ P1 |
| ハイブリッド検索 | 🔜 P2 |
| think フレームワーク | 🔜 P2 |
| DragonScale Memory | 🔜 P2 |
