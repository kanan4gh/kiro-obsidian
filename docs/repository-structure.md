# リポジトリ構造定義書

## リポジトリ構成概要

本プロジェクトは2リポジトリ構成をとる。

```
claude-obsidian-for-kiro/（開発リポジトリ）  →  publish  →  kiro-obsidian/（テンプレートリポジトリ）
```

---

## 開発リポジトリ（claude-obsidian-for-kiro）の構造

```
claude-obsidian-for-kiro/
├── .kiro/                        # Kiro IDE 設定（ハーネスのコア）
│   ├── steering/                 # LLM コンテキスト注入ファイル
│   ├── hooks/                    # イベント駆動自動化
│   └── agents/                   # サブエージェント定義
├── .steering/                    # 作業単位のステアリングファイル（SDD）
├── .claude/                      # Claude Code 設定（このリポジトリ自体の開発用）
├── claude-obsidian-main/         # 移植元（参照専用・変更しない）
├── docs/                         # 永続ドキュメント
│   └── ideas/                    # ブレインストーミングメモ
├── scripts/                      # bash/Python ヘルパースクリプト
├── agents/                       # claude-obsidian 原本エージェント（参照用）
├── skills/                       # claude-obsidian 原本スキル（参照用）
├── wiki/                         # Obsidian Vault（seeded サンプル）
├── _templates/                   # Obsidian Templater テンプレート
├── .raw/                         # ソースドキュメント投入先
├── .vault-meta/                  # Vault メタデータ（transport/mode/lock）
├── .obsidian/                    # Obsidian 設定
├── tests/                        # スクリプト単体テスト
└── bin/                          # セットアップスクリプト
```

---

## ディレクトリ詳細

### `.kiro/` — Kiro ハーネスのコア

本プロジェクトの成果物。Claude Code 用ハーネスを Kiro 向けに移植したファイル群。

#### `.kiro/steering/`

**役割**: LLM（Kiro）に注入するコンテキスト・スキル定義

**配置ファイル**:

| ファイル | inclusion | 内容 |
|---------|-----------|------|
| `wiki-core.md` | always | Vault 構造・規約・hot cache・ルーティング指示 |
| `skill-wiki-ingest.md` | always | ingest 手順（頻繁呼び出しのため常時ロード） |
| `skill-wiki.md` | manual | Vault scaffold 手順 |
| `skill-wiki-query.md` | manual | クエリ手順 |
| `skill-wiki-lint.md` | manual | lint 手順 |
| `skill-wiki-mode.md` | manual | Methodology mode 切り替え |
| `skill-autoresearch.md` | manual | 自律リサーチループ |
| `skill-save.md` | manual | 会話保存 |
| `skill-think.md` | manual | 10 原則思考ループ |

**命名規則**:
- 常時ロード: `wiki-core.md`（`wiki-` プレフィックス）
- 手動スキル: `skill-[スキル名].md`（`skill-` プレフィックス）

**frontmatter フォーマット**:
```markdown
---
inclusion: always   # または manual
---
```

#### `.kiro/hooks/`

**役割**: イベント駆動の自動化定義（JSON）

**配置ファイル**:

| ファイル | イベント | 内容 |
|---------|---------|------|
| `wiki-auto-commit.json` | fileEdited | wiki/ 変更後の git auto-commit |
| `wiki-ingest.json` | userTriggered | ingest フロー起動 |
| `wiki-session-start.json` | userTriggered | セッション開始時 hot.md 読み込み |

**JSON フォーマット**:
```json
{
  "name": "[hook名]",
  "version": "1.0.0",
  "description": "[説明]",
  "when": { "type": "fileEdited", "patterns": ["wiki/**"] },
  "then": { "type": "askAgent", "prompt": "[指示]" }
}
```

#### `.kiro/agents/`

**役割**: 並列実行サブエージェント定義（Markdown）

**配置ファイル**:
- `wiki-ingest.md` — バッチ ingest 並列エージェント
- `wiki-lint.md` — lint チェックエージェント

**frontmatter フォーマット**:
```markdown
---
name: wiki-ingest
description: >
  [説明]
model: [モデル名]
maxTurns: 30
tools: Read, Write, Edit, Bash
---
```

---

### `.steering/` — SDD 作業単位ファイル

**役割**: 各開発作業の要求・設計・タスクリストを管理（SDD フロー）

**構造**:
```
.steering/
├── _template/               # テンプレート（直接編集しない）
│   ├── requirements.md
│   ├── design.md
│   └── tasklist.md
└── YYYYMMDD-[task-name]/    # 作業単位
    ├── requirements.md
    ├── design.md
    └── tasklist.md
```

**命名規則**: `20260613-port-wiki-core` 形式（日付 + 英小文字ハイフン区切り）

---

### `docs/` — 永続ドキュメント

**役割**: プロジェクト全体の「何を作るか・どう作るか」を定義

**配置ファイル**:
- `product-requirements.md` — プロダクト要求定義書
- `functional-design.md` — 機能設計書
- `architecture.md` — 技術仕様書
- `repository-structure.md` — リポジトリ構造定義書（本ドキュメント）
- `development-guidelines.md` — 開発ガイドライン
- `glossary.md` — ユビキタス言語定義

**`docs/ideas/`**:
- 壁打ち・技術調査メモを自由形式で格納
- 構造化は最小限

---

### `scripts/` — ヘルパースクリプト

**役割**: LLM が Bash 経由で呼び出す bash/Python スクリプト群（claude-obsidian から変更なしで移植）

**配置ファイル**:

| ファイル | 言語 | 用途 |
|---------|------|------|
| `wiki-lock.sh` | bash | per-file advisory ロック |
| `wiki-mode.py` | Python | Methodology mode ページパスルーター |
| `detect-transport.sh` | bash | Vault transport 自動検出 |
| `bm25-index.py` | Python | BM25 インデックス（P2） |
| `retrieve.py` | Python | ハイブリッド検索（P2） |
| `rerank.py` | Python | コサイン再ランク（P2） |
| `contextual-prefix.py` | Python | 文脈付与プレフィックス（P2） |
| `allocate-address.sh` | bash | DragonScale アドレス割り当て（P2） |
| `boundary-score.py` | Python | タイリング境界スコア（P2） |
| `tiling-check.py` | Python | セマンティックタイリング検証（P2） |

**変更方針**: claude-obsidian 原本のスクリプトをそのまま使用。Kiro 固有の変更は不要。

> **要検討**: P2 スクリプト（bm25-index.py, retrieve.py, rerank.py 等）をテンプレートリポジトリに含めるかどうか。含めると初期状態のリポジトリが重くなるが、P2 機能への導線になる。含めない場合は wiki-retrieve 実装時に別途追加する方針とする。

---

### `wiki/` — Obsidian Vault（seeded サンプル）

**役割**: claude-obsidian の seeded vault コンテンツ。新規ユーザー向けの初期状態デモ。

```
wiki/
├── index.md              # マスターカタログ
├── log.md                # 追記専用オペレーションログ
├── hot.md                # Hot Cache（~500 words）
├── overview.md           # エグゼクティブサマリー
├── sources/              # ソースサマリーページ
├── entities/             # 人物・組織・製品
│   └── _index.md
├── concepts/             # 概念・パターン
│   └── _index.md
├── domains/              # トップレベルトピック
│   └── _index.md
├── comparisons/          # 比較分析
├── questions/            # クエリ回答
└── meta/                 # ダッシュボード
```

---

### `claude-obsidian-main/` — 移植元（参照専用）

**役割**: 移植元の claude-obsidian リポジトリのコピー。実装時の参照に使用。

**運用ルール**:
- このディレクトリのファイルは**変更しない**
- 移植する際は `.kiro/` 配下に新規ファイルとして作成する
- 差分確認・参照のみに使用

---

### `tests/` — スクリプト単体テスト

**役割**: `scripts/` の動作確認テスト（claude-obsidian から流用）

```
tests/
├── test_wiki_lock.sh         # wiki-lock 並列安全性
├── test_wiki_mode.py         # mode ルーティング
├── test_bm25_index.py        # BM25 インデックス（P2）
├── test_retrieve.py          # 検索パイプライン（P2）
├── test_tiling_check.py      # セマンティックタイリング（P2）
├── test_boundary_score.py    # 境界スコア（P2）
├── test_contextual_prefix.py # 文脈付与プレフィックス（P2）
├── test_concurrent_write.sh  # 並列書き込み（P2）
└── test_allocate_address.sh  # アドレス割り当て（P2）
```

---

### `bin/` — セットアップスクリプト

**役割**: Vault 初期セットアップ用スクリプト（claude-obsidian から流用）

```
bin/
├── setup-vault.sh         # Obsidian 設定（graph.json/app.json/snippets）
├── setup-retrieve.sh      # ハイブリッド検索パイプライン構築（P2）
├── setup-mode.sh          # Methodology mode 設定（P2）
├── setup-dragonscale.sh   # DragonScale Memory 有効化（P2）
└── setup-multi-agent.sh   # マルチエージェント設定
```

---

---

## テンプレートリポジトリ（kiro-obsidian）の構造

ユーザーが GitHub Template から複製した直後の状態。開発用ファイルを含まないクリーンな構成。

```
kiro-obsidian/（GitHub Template Repository）
├── .kiro/
│   ├── steering/
│   │   ├── wiki-core.md              # inclusion: always
│   │   ├── skill-wiki-ingest.md      # inclusion: always
│   │   ├── skill-wiki.md             # inclusion: manual
│   │   ├── skill-wiki-query.md       # inclusion: manual
│   │   ├── skill-wiki-lint.md        # inclusion: manual
│   │   ├── skill-wiki-mode.md        # inclusion: manual
│   │   ├── skill-autoresearch.md     # inclusion: manual
│   │   ├── skill-save.md             # inclusion: manual
│   │   └── skill-think.md            # inclusion: manual
│   ├── hooks/
│   │   ├── wiki-auto-commit.json
│   │   ├── wiki-ingest.json
│   │   └── wiki-session-start.json
│   └── agents/
│       ├── wiki-ingest.md
│       └── wiki-lint.md
├── scripts/                          # bash/Python スクリプト群
├── wiki/                             # seeded Vault コンテンツ
├── _templates/                       # Obsidian Templater テンプレート
├── .obsidian/                        # Obsidian 設定（graph.json 等）
├── .raw/                             # ソース投入先（空）
├── .vault-meta/                      # Vault メタデータ（空）
├── bin/                              # setup-vault.sh 等
├── tests/                            # スクリプト単体テスト
├── CLAUDE.md                         # テンプレート利用者向け使い方ガイド
└── README.md
```

**開発リポジトリに含まれ、テンプレートに含めないもの**:
- `docs/` — 設計ドキュメント（開発者向け）
- `.steering/` — SDD 作業ファイル
- `.claude/` — Claude Code 設定
- `claude-obsidian-main/` — 移植元参照

### Publish フロー

```bash
# 開発リポジトリでの作業完了後
# .kiro/ 等の成果物を kiro-obsidian リポジトリへ同期

rsync -av --delete \
  --exclude 'docs/' \
  --exclude '.steering/' \
  --exclude '.claude/' \
  --exclude 'claude-obsidian-main/' \
  --exclude '.git/' \
  ./ ../kiro-obsidian/

cd ../kiro-obsidian
git add -A && git commit -m "sync: v[version] from claude-obsidian-for-kiro"
gh release create v[version]
```

---

## ファイル配置規則

### 移植ファイルの配置

| 移植元（claude-obsidian-main/） | 移植先 | 変更 |
|-------------------------------|--------|------|
| `skills/wiki/SKILL.md` | `.kiro/steering/skill-wiki.md` | frontmatter 変換 |
| `skills/wiki-ingest/SKILL.md` | `.kiro/steering/skill-wiki-ingest.md` | frontmatter 変換 |
| `skills/wiki-query/SKILL.md` | `.kiro/steering/skill-wiki-query.md` | frontmatter 変換 |
| `skills/wiki-lint/SKILL.md` | `.kiro/steering/skill-wiki-lint.md` | frontmatter 変換 |
| `agents/wiki-ingest.md` | `.kiro/agents/wiki-ingest.md` | frontmatter 変換 |
| `agents/wiki-lint.md` | `.kiro/agents/wiki-lint.md` | frontmatter 変換 |
| `hooks/hooks.json` | `.kiro/hooks/*.json`（分割） | JSON フォーマット変換 |
| `scripts/` | `scripts/` | **変更なし** |
| `_templates/` | `_templates/` | **変更なし** |
| `wiki/` | `wiki/` | **変更なし** |
| `.obsidian/` | `.obsidian/` | **変更なし** |

### wiki-core.md（新規作成）

claude-obsidian には直接の対応ファイルなし。以下の内容を統合して新規作成:
- `skills/wiki/SKILL.md` の Architecture・Hot Cache セクション
- `hooks/hooks.json` の SessionStart プロンプト
- Vault 全体の規約・命名規則

---

## 命名規則

### `.kiro/steering/` ファイル

| 種別 | パターン | 例 |
|-----|---------|-----|
| 常時ロード（コア） | `wiki-[用途].md` | `wiki-core.md` |
| 手動スキル | `skill-[スキル名].md` | `skill-wiki-ingest.md` |

### `.kiro/hooks/` ファイル

- パターン: `wiki-[用途].json`
- 例: `wiki-auto-commit.json`, `wiki-ingest.json`

### `.steering/` ディレクトリ

- パターン: `YYYYMMDD-[kebab-case-task-name]/`
- 例: `20260613-port-wiki-core/`

### `wiki/` ページ

- ソース: `wiki/sources/[slug].md`（PascalCase または kebab-case）
- エンティティ: `wiki/entities/[Name].md`（PascalCase）
- コンセプト: `wiki/concepts/[Name].md`（PascalCase）

---

## 除外設定（.gitignore）

```
# Vault メタデータ（環境固有）
.vault-meta/locks/
.vault-meta/transport.json

# Methodology mode（opt-in）
# .vault-meta/mode.json  # git add -f で明示的にコミット可

# Obsidian ローカル設定
.obsidian/workspace.json
.obsidian/workspace-mobile.json

# macOS
.DS_Store

# Python
__pycache__/
*.pyc
*.pyo

# BM25 インデックス（再構築可能）
.vault-meta/bm25-index.db
```
