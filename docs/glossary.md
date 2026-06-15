# ユビキタス言語定義（用語集）

**更新日**: 2026-06-14

このドキュメントはプロジェクト内で統一して使用する用語を定義する。ドキュメント・会話・コードで同じ用語を使うことで認識齟齬を防ぐ。

---

## ドメイン用語（LLM Wiki パターン）

### Vault
**定義**: Obsidian が管理するフォルダ全体。wiki ページ・ソース・メタデータを含む知識の格納庫。

**本プロジェクトでの使用**: リポジトリルートが Vault。`.raw/`・`wiki/`・`.vault-meta/` を含む。

**関連用語**: wiki、hot cache

---

### LLM Wiki パターン
**定義**: Andrej Karpathy が提唱した知識管理パターン。LLM がソースを読み込んで構造化した wiki ページを生成・更新し続けることで、知識が複利で増えていく仕組み。

**関連用語**: ingest、hot cache、Vault

**英語表記**: LLM Wiki Pattern

---

### Hot Cache（ホットキャッシュ）
**定義**: `wiki/hot.md` に保存されるセッション間コンテキストキャッシュ。約 500 words。前セッションの重要情報・最近の変更・進行中のスレッドを記録する。

**説明**: LLM はセッション開始時にまず hot.md を読むことで、前回の作業文脈を即座に取得できる。index.md（全ページカタログ）を読む前の軽量な第一ステップ。

**関連用語**: Vault、wiki/index.md

---

### Ingest（インジェスト）
**定義**: ソースドキュメントを読み込み、エンティティ・コンセプト・ソースサマリーの wiki ページを生成して Vault に統合するオペレーション。

**説明**: 1 ソースから通常 8〜15 ページが生成される。index.md・log.md・hot.md も更新される。

**関連用語**: .raw/、wiki-ingest スキル

---

### Source（ソース）
**定義**: Vault に取り込む元データ。`.raw/` ディレクトリに配置する。PDF・Markdown・テキスト等。

**ルール**: `.raw/` 内のソースは LLM が変更してはならない（読み取り専用）。

**関連用語**: .raw/、ingest

---

### wiki-lock（ウィキロック）
**定義**: `scripts/wiki-lock.sh` が提供する per-file advisory ロック機構。並列 ingest 時に複数エージェントが同一ページを同時書き込みする競合を防ぐ。

**説明**: ファイルパスの SHA1 ハッシュをロックキーとして使用。デフォルト 60 秒で自動解放（stale lock 防止）。

**関連用語**: ingest、agents

---

### Methodology Mode（メソドロジーモード）
**定義**: Vault のページ整理方針。`generic`・`LYT`・`PARA`・`Zettelkasten` の 4 種類。`.vault-meta/mode.json` に記録される。

**説明**: モードによって新規ページの保存パスが変わる（例: LYT では `wiki/notes/`、PARA では `wiki/resources/incoming/`）。`scripts/wiki-mode.py route <type> "<name>"` がパスを解決する。

**関連用語**: wiki-mode スキル、wiki-mode.py

---

### Transport（トランスポート）
**定義**: LLM が Vault ファイルを読み書きする経路。優先順位：Obsidian CLI → mcp-obsidian → mcpvault → filesystem（常に利用可能）。

**説明**: `scripts/detect-transport.sh` が環境を検出して `.vault-meta/transport.json` に書き込む。本プロジェクトでは基本的に filesystem transport を使用。

**関連用語**: .vault-meta/

---

## アーキテクチャ用語（Kiro ハーネス）

### ハーネス（Harness）
**定義**: 「エンジン（LLM）を動かすために必要な、エンジン以外のすべて」。本プロジェクトでは `.kiro/`・`scripts/`・`docs/` の総体を指す。

**関連用語**: Steering、Hook、Agent

---

### Steering（ステアリング）
**定義**: Kiro IDE が LLM コンテキストに注入する Markdown ファイル群。`.kiro/steering/` に配置。

**2種類**:
- `inclusion: always` — 常時注入（セッション全体で有効）
- `inclusion: manual` — `#skill-name` 参照時のみ注入

**関連用語**: skill、Hook

---

### Skill（スキル）
**定義**: `inclusion: manual` の steering ファイルとして定義された、特定オペレーションの手順書。`#skill-wiki-ingest` のように `#` プレフィックスで参照して起動する。

**説明**: claude-obsidian では `/wiki` コマンドでロードしていたが、Kiro では `#skill-wiki` 参照に相当する。

**関連用語**: Steering、claude-obsidian スキル

**英語表記**: Skill

---

### Hook（フック）
**定義**: `.kiro/hooks/` に配置された JSON ファイルで定義される自動化ルール。イベント発火時に LLM プロンプトまたはシェルコマンドを実行する。

**2種類のイベント**:
- `fileEdited` — ファイル変更後に発火
- `userTriggered` — ユーザーが手動で起動

**関連用語**: auto-commit、Steering

---

### Agent（エージェント）
**定義**: `.kiro/agents/` に配置された Markdown ファイルで定義されるサブエージェント。並列 ingest・lint 等の重処理を担当する。

**関連用語**: wiki-ingest agent、wiki-lint agent

---

### auto-commit（オートコミット）
**定義**: `wiki-auto-commit.json` hook が wiki/ ファイル変更後に自動的に実行する `git commit`。コミットメッセージは `"wiki: auto-commit YYYY-MM-DD HH:MM"` 形式。

**説明**: wiki-lock が保持中の場合はコミットを延期する（ingest 途中の torn commit を防ぐ）。`.vault-meta/auto-commit.disabled` が存在する場合は無効化される。

**関連用語**: Hook、wiki-lock

---

## 技術用語

### claude-obsidian
**定義**: 本プロジェクトの移植元。Claude Code（Anthropic の AI コーディングアシスタント）用の LLM Wiki パターン実装。v1.9.2 を移植対象とする。

**関連用語**: Kiro、Claude Code

---

### Kiro
**定義**: AWS の AI コーディングアシスタント。本プロジェクトの移植先。Steering・Hook・Agent の仕組みを持つ。

**関連用語**: Steering、Hook、Agent、Claude Code

---

### Claude Code
**定義**: Anthropic の AI コーディングアシスタント。claude-obsidian の実行基盤。本プロジェクトでは会社環境で利用不可のため Kiro に移植する。

**関連用語**: claude-obsidian、Kiro

---

### SDD（スペック駆動開発）
**正式名称**: Spec-Driven Development

**定義**: 実装前に requirements.md → design.md → tasklist.md の順でスペックを作成し、ユーザー承認を得てから実装に入る開発プロセス。

**本プロジェクトでの使用**: `.steering/YYYYMMDD-[task-name]/` ディレクトリに各スペックファイルを配置して管理する。

**関連用語**: .steering/

---

## 略語・頭字語

### BM25
**正式名称**: Best Match 25

**定義**: テキスト検索のランキング関数。TF-IDF の改良版。`scripts/bm25-index.py` で wiki ページの疎なインデックスを構築する（P2 機能）。

**関連用語**: wiki-retrieve、hybrid retrieval

---

### LYT
**正式名称**: Linking Your Thinking

**定義**: Nick Milo が提唱する Obsidian 整理方法論。MOC（Map of Content）をナビゲーションの基本単位とし、フォルダより wikilink による接続を重視する。

**関連用語**: Methodology Mode、MOC

---

### PARA
**正式名称**: Projects, Areas, Resources, Archives

**定義**: Tiago Forte が提唱するデジタル情報整理方法論。情報を「行動可能性」で分類する（Projects → Areas → Resources → Archives）。

**関連用語**: Methodology Mode

---

### MOC
**正式名称**: Map of Content

**定義**: LYT モードにおける「インデックスページ」。特定トピックの関連ページへのリンクを集約した概要ページ。`wiki/mocs/<topic>-moc.md` に配置。

**関連用語**: LYT、Methodology Mode

---

## ファイル・ディレクトリ用語

### `.raw/`
**定義**: ソースドキュメントの投入先ディレクトリ。Obsidian のファイルエクスプローラーでは非表示（ドットプレフィックス）。LLM は読み取るが変更しない。

### `.vault-meta/`
**定義**: Vault のメタデータ格納ディレクトリ。`transport.json`・`mode.json`・ロックファイル・`hook.log` を含む。

### `wiki/hot.md`
**定義**: Hot Cache ファイル。~500 words のセッション間コンテキストキャッシュ。最終更新日・重要な最近の事実・最近の変更・進行中スレッドを記録する。

### `wiki/index.md`
**定義**: Vault 全ページのマスターカタログ。ingest のたびに更新される。LLM は hot.md で足りない場合に index.md を読む（second tier）。

### `wiki/log.md`
**定義**: 追記専用のオペレーションログ。過去エントリは編集しない。新しいエントリはファイルの先頭に追加する。
