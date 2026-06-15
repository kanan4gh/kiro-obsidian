# 開発ガイドライン

## 基本方針

本プロジェクトは **AI エージェントハーネス** の開発であり、アプリケーションコードではなく Markdown・JSON・bash/Python で構成される。コーディング規約はこの性質に合わせて定義する。

- **Kiro IDE の GUI 内で完結する**: kiro-cli は使用しない
- **Kiro built-in Spec mode は使わない**: 作業スペックは `.steering/` ベースで管理する
- **claude-obsidian-main/ は変更しない**: 移植元のコピーは参照専用

---

## 開発フロー（SDD）

全ての機能開発・バグ修正は以下のフローで行う:

```
1. ドキュメント確認  : docs/ の永続ドキュメントで方針を確認
2. Issue 作成       : GitHub Issue を作成（gh コマンドまたは Web UI）
3. スペック作成     : .steering/_template/ をコピーして YYYYMMDD-xxx/ を作成
4. 要求定義         : requirements.md を記入（Issue URL を必ず記載）→ 承認
5. 設計             : design.md を記入 → 承認
6. タスク計画       : tasklist.md を記入 → 承認
7. 実装             : フィーチャーブランチで tasklist.md のタスクを順番に実装
8. PR 作成・マージ  : フィーチャーブランチ → PR → main へマージ
9. Publish          : kiro-obsidian テンプレートリポジトリへ成果物を同期・リリース
```

### Publish フロー（ステップ9の詳細）

機能がまとまった段階で `kiro-obsidian`（テンプレートリポジトリ）へ同期する。

詳細な手順は `docs/repository-structure.md` の「Publish フロー」セクションを参照。

### ブランチ戦略

- `main`: 安定版。直接コミット禁止
- `feature/[kebab-case]`: 機能実装（例: `feature/port-wiki-core`）
- `fix/[kebab-case]`: バグ修正（例: `fix/auto-commit-hook`）

### コミットメッセージ規約

**フォーマット**: `<type>(<scope>): <subject>`

| type | 用途 |
|------|------|
| `port` | claude-obsidian からの移植（本プロジェクト固有） |
| `feat` | 新規ハーネス機能 |
| `fix` | バグ修正 |
| `docs` | ドキュメント更新 |
| `test` | テスト追加・修正 |
| `chore` | 設定・ツール変更 |

**例**:
```
port(steering): wiki-ingest スキルを Kiro steering 形式に変換

skills/wiki-ingest/SKILL.md の内容を .kiro/steering/skill-wiki-ingest.md に移植。
inclusion: always として常時ロード設定。

Closes #12
```

### PR プロセス

**作成前チェック**:
- [ ] `.steering/` の tasklist.md が全タスク完了（`[x]`）になっている
- [ ] 動作確認済み（Kiro 上で当該機能を実際に試した）
- [ ] 関連 Issue を PR 本文に記載（`Closes #N`）

---

## ファイル作成規約

### Steering ファイル（`.kiro/steering/*.md`）

**必須 frontmatter**:
```markdown
---
inclusion: always   # または manual
---
```

**`always` を選ぶ基準**:
- セッションのほぼ全体で参照される（wiki-core.md, skill-wiki-ingest.md）
- コンテキストへのコストを許容できるサイズ（目安 10KB 以下）

**`manual` を選ぶ基準**:
- 特定のスキルとして呼び出される（`#skill-wiki-lint` など）
- 頻度が低い・サイズが大きい

**移植時の変換ルール**（claude-obsidian SKILL.md → Kiro steering）:

| claude-obsidian | Kiro steering |
|----------------|--------------|
| `---\nname: wiki\ndescription: ...\nallowed-tools: ...\n---` | `---\ninclusion: manual\n---` |
| Community footer セクション | 削除（Kiro では不要） |
| `skills/*/references/*.md` へのリンク | インライン展開または steering 内に記載 |

**変換例（Before / After）**:

```markdown
# Before: claude-obsidian SKILL.md の冒頭
---
name: wiki-ingest
description: Ingest a source document into the wiki
allowed-tools: Read, Write, Edit, Bash
tools: Read, Write, Edit, Bash
---
# Wiki Ingest
...（英語で記述された手順）...
> [!community]
> Built by @AgriciDaniel
```

```markdown
# After: Kiro steering ファイル（.kiro/steering/skill-wiki-ingest.md）
---
inclusion: always
---
# Wiki インジェスト手順
...（日本語に翻訳した手順）...
```

> **変換ポイント**: `tools:` / `allowed-tools:` フィールドは Kiro steering では不要（Kiro IDE がツールを管理するため削除）。`name:` / `description:` も steering frontmatter には不要。Community footer は削除。

**翻訳指針**（ターゲットユーザーは日本語圏）:

- steering ファイルの**内容はすべて日本語**で記述する（frontmatter のキーは英語のまま）
- 技術用語（wikilink・frontmatter・hot cache 等）は日本語訳せずそのまま使う
- LLM への指示文は「〜してください」「〜すること」の日本語で統一する
- wiki ページの生成言語は `wiki-core.md` に「すべての wiki ページを日本語で生成すること」と明示する

### Hooks ファイル（`.kiro/hooks/*.json`）

**必須フィールド**:
```json
{
  "name": "wiki-[用途]",
  "version": "1.0.0",
  "description": "[日本語で説明]",
  "when": { "type": "fileEdited" または "userTriggered", ... },
  "then": { "type": "askAgent", "prompt": "..." }
}
```

**fileEdited の patterns**:
- Glob パターンで記述（例: `"wiki/**"`, `".raw/**"`）
- 過度に広いパターンは避ける（全ファイル監視はパフォーマンス問題を起こす）

**prompt の書き方**:
- `#skill-name` でスキル参照を明示する
- 日本語で記述してよい
- claude-obsidian の hooks.json の `"prompt"` フィールドを基に変換

### Agents ファイル（`.kiro/agents/*.md`）

**必須 frontmatter**:
```markdown
---
name: wiki-[用途]
description: >
  [説明。使用場面・例を含める]
model: [モデル名]
maxTurns: 30
tools: Read, Write, Edit, Bash
---
```

**claude-obsidian agents/*.md からの変換**:
- `model: sonnet` → Kiro 対応モデル名に変換（検証して記載）
- `tools:` フィールドはカンマ区切り形式に統一

### スクリプトファイル（`scripts/`）

**原則: 変更なし**。claude-obsidian 原本をそのまま使用。

変更が必要な場合のみ以下の規約に従う:

**bash スクリプト**:
- `#!/usr/bin/env bash` で始める
- `set -euo pipefail` を先頭に記述
- 関数名は `snake_case`

**Python スクリプト**:
- `#!/usr/bin/env python3`
- 型ヒントを使用（Python 3.10+）
- 外部パッケージは最小限（標準ライブラリを優先）

### Wiki ページ（`wiki/**/*.md`）

**必須 frontmatter**:
```yaml
---
type: source | entity | concept | meta | session
title: "ページタイトル"
status: draft | active | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [タグ1, タグ2]
---
```

**Wikilink 規約**:
- `[[Note Name]]` 形式（ファイル名はユニーク、パス不要）
- 矛盾は `> [!contradiction] Title` callout で記録
- `.raw/` のソースは変更しない

---

## テスト方針

### 動作確認（主テスト）

Kiro IDE 上での実際の動作確認を主テストとする。各スキルのポート完了後に以下を確認:

| スキル | 確認内容 |
|-------|---------|
| wiki scaffold | フォルダ構造・index.md・hot.md が生成される |
| wiki-ingest | 1ソースから 8〜15 ページが生成、index/log/hot が更新される |
| wiki-query | wikilink 引用付きの回答が返る |
| wiki-lint | 8 カテゴリのレポートが出力される |
| auto-commit | wiki/ 編集後に git log に "wiki: auto-commit" が記録される |

### スクリプト単体テスト

`tests/` の bash/Python テストスイートを実行:

```bash
# wiki-lock の並列安全性
bash tests/test_wiki_lock.sh

# Methodology mode ルーティング
python3 tests/test_wiki_mode.py

# 並列書き込みテスト
bash tests/test_concurrent_write.sh
```

**P2 テスト**（BM25・検索パイプラインは P2 実装後に実施）

---

## 開発環境セットアップ

### 必要ツール

| ツール | バージョン | 用途 |
|--------|-----------|------|
| Kiro IDE | 要検討（バージョン固定方法を調査中） | AI コーディングアシスタント |
| Obsidian | v1.9.10+ | Vault ビューア |
| Git | 任意 | バージョン管理 |
| Python | 3.10+ | scripts/ 実行 |
| bash | 4.0+ | scripts/ 実行 |

### セットアップ手順

```bash
# 1. リポジトリのクローン（または GitHub Template から複製）
git clone https://github.com/[user]/claude-obsidian-for-kiro
cd claude-obsidian-for-kiro

# 2. Obsidian Vault の初期設定
bash bin/setup-vault.sh

# 3. Kiro IDE でフォルダを開く
# File → Open Folder → claude-obsidian-for-kiro/

# 4. 動作確認
bash tests/test_wiki_lock.sh
python3 tests/test_wiki_mode.py
```

### Kiro 操作ガイド

**スキル起動**:
- scaffold: チャットに「vault を設定したい」または「`#skill-wiki` で scaffold して」
- ingest: チャットに「`[ファイル名]` を ingest して」（skill-wiki-ingest.md は always ロード）
- query: チャットに「X について何を知ってる？」または「`#skill-wiki-query` で質問：X」
- lint: チャットに「lint the wiki」または「`#skill-wiki-lint` でヘルスチェック」

**Hook 起動**:
- `wiki-ingest.json`（userTriggered）: Kiro のフックメニューから "wiki-ingest" を選択
- `wiki-auto-commit.json`（fileEdited）: wiki/ のファイル編集後に自動発火

---

## PR・リリースフロー

### PR 作成

```bash
# フィーチャーブランチから PR を作成
gh pr create --title "port: wiki-core steering ファイルを実装" \
  --body "$(cat <<'EOF'
## 概要
wiki-core.md（inclusion: always）を実装。Vault 構造・hot cache・ルーティング指示を定義。

## 変更内容
- `.kiro/steering/wiki-core.md` を新規作成
- Vault 規約・hot cache フォーマット・スキルルーティング指示を記載

## 動作確認
- [ ] Kiro 上で wiki-core.md が常時ロードされることを確認
- [ ] hot.md の読み込み指示が機能することを確認

Closes #[Issue番号]
EOF
)"
```

### リリース

```bash
# 関連 Issue をクローズ
gh issue close [番号]

# GitHub Release を作成
gh release create v[バージョン] --title "v[バージョン]" --notes "[リリースノート]"
```
