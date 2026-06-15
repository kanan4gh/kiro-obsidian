# 技術仕様書

## テクノロジースタック

### 言語・ランタイム

| 技術 | バージョン | 用途 |
|------|-----------|------|
| Python | 3.10+ | scripts/ の検索・ロック・モードルーティングスクリプト |
| bash | 4.0+ (zsh 可) | セットアップスクリプト・wiki-lock・detect-transport |
| Markdown | — | steering / agents / Vault ページ・テンプレート |
| JSON | — | hooks 定義・Vault メタデータ・Obsidian 設定 |

### 外部ツール・依存関係

| 技術 | 必須/任意 | 用途 | 選定理由 |
|------|---------|------|----------|
| Kiro IDE | 必須 | AI コーディングアシスタント。steering/hooks/agents を実行 | 本プロジェクトの移植先 |
| Obsidian v1.9.10+ | 必須 | Vault ビューア・ナビゲーション | LLM Wiki パターンの表示基盤 |
| Git | 必須 | Vault の自動コミット・バージョン管理 | auto-commit hook の実行基盤 |
| Python 標準ライブラリ | 必須 | scripts/ の実行（外部パッケージ不要） | 依存最小化 |
| ollama | 任意（P2） | wiki-retrieve のコサイン再ランク用ローカル埋め込み | プライバシー保護のためローカル実行 |
| Anthropic API | 任意（P2） | wiki-retrieve の文脈付与プレフィックス生成 | Contextual Retrieval 研究に基づく精度向上 |

---

## アーキテクチャパターン

### 2リポジトリ構成

```
claude-obsidian-for-kiro/（開発リポジトリ）
├── .kiro/                  ← 移植・検証作業場
├── claude-obsidian-main/   ← 移植元参照（変更しない）
├── docs/, .steering/       ← 開発記録
└── [publish スクリプト]
        │ gh release / rsync 等で同期
        ▼
kiro-obsidian/（テンプレートリポジトリ・GitHub Template）
├── .kiro/          ← ユーザーが使う
├── scripts/        ← ユーザーが使う
├── wiki/           ← ユーザーが使う（seeded）
├── _templates/     ← ユーザーが使う
├── .obsidian/      ← ユーザーが使う
├── .raw/           ← ユーザーが使う（空）
└── bin/            ← ユーザーが使う
```

**publish の基本方針**: 開発リポジトリで動作確認が完了した `.kiro/`・`scripts/` 等を、不要ファイル（`docs/`・`.steering/`・`.claude/`・`claude-obsidian-main/`）を除いてテンプレートリポジトリに同期する。

---

## ハーネスアーキテクチャ（3層構造）

本プロジェクトはアプリケーションコードではなく **AI エージェントハーネス** である。LLM（Kiro の AI）を動かすための「エンジン以外のすべて」を提供する。

```
┌─────────────────────────────────────────────┐
│  コンテキスト層（.kiro/steering/）            │
│  LLM が「何をすべきか」を常時・手動で注入     │
├─────────────────────────────────────────────┤
│  自動化層（.kiro/hooks/ + .kiro/agents/）    │
│  イベント駆動の自動化とサブエージェント並列化 │
├─────────────────────────────────────────────┤
│  ツール層（scripts/ + git）                  │
│  LLM が呼び出すスクリプト・ファイルシステム   │
└─────────────────────────────────────────────┘
```

#### コンテキスト層
- **責務**: LLM に操作手順・Vault 規約・スキル定義を提供する
- `inclusion: always` → 常時 LLM コンテキストに注入（wiki-core.md, skill-wiki-ingest.md）
- `inclusion: manual` → `#skill-name` 参照時のみ注入（他のスキルファイル）
- **禁止**: ビジネスロジックのハードコード。手順の定義に留める

#### 自動化層
- **責務**: ファイル変更・ユーザートリガーに応じて自動化アクションを実行する
- hooks: JSON 定義（`fileEdited` / `userTriggered`）でシェルコマンドまたは LLM プロンプトを発火
- agents: Markdown 定義のサブエージェント。並列 ingest・lint に使用
- **禁止**: hooks からの直接的な wiki ページ操作（hooks → scripts/git のみ）

#### ツール層
- **責務**: LLM が bash/Python 経由で呼び出すユーティリティを提供する
- `scripts/wiki-lock.sh`: per-file advisory ロック
- `scripts/wiki-mode.py`: Methodology mode ページパスルーター
- `git`: Vault の自動バージョン管理
- **禁止**: ツール層から LLM への直接呼び出し（一方向依存）

---

## Kiro 固有アーキテクチャ設計

### Steering ファイルの inclusion 設計方針

```
常時ロード（inclusion: always）
├── wiki-core.md          〜2KB  Vault 規約・hot cache・ルーティング指示
└── skill-wiki-ingest.md  〜8KB  ingest は最頻操作のため常時展開

手動ロード（inclusion: manual）
├── skill-wiki.md         〜6KB  scaffold（初回のみ）
├── skill-wiki-query.md   〜3KB  query（都度参照）
├── skill-wiki-lint.md    〜4KB  lint（定期）
├── skill-wiki-mode.md    〜4KB  mode 切り替え（稀）
├── skill-autoresearch.md 〜6KB  自律リサーチ（都度）
├── skill-save.md         〜3KB  会話保存（都度）
└── skill-think.md        〜5KB  思考フレームワーク（都度）
```

**設計根拠**: Kiro のコンテキストウィンドウを節約するため、頻度の低いスキルは手動ロードに限定する。ingest は「ソース投入 → すぐ実行」のフローが最も多いため always に昇格。

### Hooks 設計

```json
// fileEdited hook のパターン（wiki-auto-commit.json）
{
  "when": { "type": "fileEdited", "patterns": ["wiki/**", ".raw/**", ".vault-meta/**"] },
  "then": { "type": "askAgent", "prompt": "<bash スクリプト実行指示>" }
}

// userTriggered hook のパターン（wiki-ingest.json）
{
  "when": { "type": "userTriggered" },
  "then": { "type": "askAgent", "prompt": "#skill-wiki-ingest を参照して ingest を開始..." }
}
```

**Claude Code との差分対応**:

| Claude Code hook | Kiro 対応 | 備考 |
|-----------------|-----------|------|
| SessionStart | `wiki-core.md`（always）に hot.md 読み込み指示 | 自動発火なし。常時コンテキストで代替 |
| PostToolUse(Write/Edit) | `wiki-auto-commit.json`（fileEdited） | ほぼ同等 |
| PostCompact | 対応なし | wiki-core.md に「文脈が失われた場合 hot.md を再読」と記載 |
| Stop | 対応なし | ユーザーに「セッション終了前に hot.md 更新を」と wiki-core.md で指示 |

---

## データ永続化戦略

### ストレージ設計

| データ種別 | ストレージ | フォーマット | 備考 |
|----------|----------|------------|------|
| wiki ページ | Vault ファイルシステム | Markdown（YAML frontmatter） | Obsidian が直接開ける形式 |
| ソースドキュメント | `.raw/` | 任意（PDF/MD/TXT 等） | 変更禁止・読み取り専用 |
| Vault メタデータ | `.vault-meta/` | JSON | transport.json / mode.json |
| per-file ロック | `.vault-meta/locks/` | lock ファイル（sha1 ハッシュ名） | 60秒で自動解放 |
| git 履歴 | `.git/` | git object store | auto-commit で自動管理 |

### Vault バックアップ

- **主バックアップ**: git auto-commit（wiki/ 変更のたびに自動コミット）
- **Obsidian Git**: コミュニティプラグインで 15 分ごとの追加バックアップ（任意）
- **手動バックアップ**: ユーザーが `git push` でリモートに同期

---

## セキュリティ設計

### データ保護

- **ローカルファースト**: wiki データは全てローカルファイルシステムに保存。外部サービスへの送信はオプトイン
- **web egress 制御**: autoresearch / wiki-retrieve の外部通信は `--allow-egress` フラグが必要
- **入力サニタイズ**: `wiki-mode.py` の `safe_name()` でパストラバーサル・制御文字をフィルタ（claude-obsidian v1.8.2+ 実装を流用）

### アクセス制御

- `.raw/` は LLM が変更しないようスキル定義に明示（「.raw/ を変更するな」と毎スキルに記載）
- `.vault-meta/auto-commit.disabled` ファイルが存在する場合 auto-commit を無効化（ユーザーによる明示的制御）

### MCP 接続（オプション）

Obsidian Local REST API を使った MCP 接続はオプション（`kiro-template/.kiro/settings/mcp.json` で設定）。未設定の場合はファイルシステム transport にフォールバック。

---

## パフォーマンス設計

### コンテキスト使用量の最適化

| 操作 | トークン消費 | 設計根拠 |
|------|-----------|---------|
| 通常会話（wiki なし） | +2KB（wiki-core.md） | always steering のベースコスト |
| query | +2KB（hot.md）〜+5KB（index.md + ページ） | hot → index → page の段階的読み込み |
| ingest | +8KB（skill-wiki-ingest.md 常時） | 頻度最大のためコスト許容 |
| lint | +4KB（#skill-wiki-lint 手動） | 定期実行のため手動許容 |

### wiki-lock パフォーマンス

- **並列 ingest**: 異なるページへの書き込みはロック競合なしに並列実行
- **同一ページ競合**: 2秒待ちリトライ1回のみ。失敗時はスキップ（ログ記録）
- **stale lock**: 60秒で自動解放（SessionStart 時にも `wiki-lock clear-stale --max-age 3600` を実行）

---

## スケーラビリティ設計

### Vault サイズへの対応

| Vault サイズ | 想定ページ数 | 対応方針 |
|------------|-----------|---------|
| 小規模 | 〜100 ページ | hot.md + index.md 直読みで十分 |
| 中規模 | 100〜500 ページ | domain _index.md で分割ナビゲーション |
| 大規模（P2） | 500+ ページ | wiki-retrieve（BM25 + cosine rerank）による検索に移行 |

### wiki-fold（DragonScale・P2）

`wiki/log.md` が大規模になった場合、`scripts/wiki-fold` でロールアップして古いエントリをアーカイブする。

---

## 技術的制約

### 環境要件

| 要件 | 値 |
|-----|---|
| OS | macOS / Linux（bash 4.0+） |
| Python | 3.10+ |
| Obsidian | v1.9.10+（Bases 使用時）/ v1.6+ （Dataview fallback） |
| Git | 任意バージョン |
| Kiro IDE | 最新版 |

### Kiro 固有の制約

1. **スキル起動**: `/wiki` のようなスラッシュコマンドなし → `#skill-name` + 自然言語で代替
2. **PostCompact なし**: コンテキスト圧縮後の自動 hot.md 再読不可 → wiki-core.md の常時指示で代替
3. **SessionStop なし**: セッション終了フック不可 → ユーザー手動または wiki-core.md の指示で代替
4. **Kiro built-in Spec mode**: 使用しない（SDD は `.steering/` ベースで管理）
5. **kiro-cli**: 使用しない（Kiro IDE GUI 内で完結）

### Claude Code ハーネスとの対応関係

本プロジェクトは claude-obsidian v1.9.2 からの移植。ターゲットユーザーは日本語圏のため、**ハーネス部分・wiki 部分ともに日本語で提供する**。

| claude-obsidian | kiro-obsidian | 変更内容 |
|----------------|--------------|---------|
| `skills/*/SKILL.md`（英語） | `.kiro/steering/skill-*.md` | **日本語翻訳** + Kiro frontmatter 変換 |
| `hooks/hooks.json`（英語） | `.kiro/hooks/*.json` | **日本語翻訳** + JSON フォーマット変換・分割 |
| `agents/*.md`（英語） | `.kiro/agents/*.md` | **日本語翻訳** + frontmatter フォーマット変換 |
| `wiki/`（英語 seeded） | `wiki/` | **日本語版に差し替え**（seeded コンテンツを翻訳） |
| `_templates/`（英語） | `_templates/` | **日本語翻訳** |
| `scripts/` | `scripts/` | **変更なし**（コードはそのまま） |
| `.obsidian/` | `.obsidian/` | **変更なし** |
