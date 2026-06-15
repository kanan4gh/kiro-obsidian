# プロダクト要求定義書

## プロダクト概要

### 名称
**kiro-obsidian** — Kiro + Obsidian 知識複利ハーネス

### プロダクトコンセプト
- **LLM Wiki パターンの Kiro 移植**: claude-obsidian（Claude Code 用ハーネス）を Kiro IDE 向けに完全移植し、同等の知識構築体験を実現する
- **知識が複利で増える構造**: ソースを追加するたびに wiki が充実し、過去の知識と新しい知識が自動的に接続される Obsidian Vault を構築する
- **会社環境対応**: Claude Code が使えない会社環境でも、Kiro を通じて AI 支援の知識管理を実現する

### プロダクトビジョン
会社環境で Claude Code が使えないエンジニア・ナレッジワーカーが、Kiro IDE を通じて「知識が複利で増える LLM Wiki パターン」を構築できるようにする。ソースをドロップして "ingest" と言うだけで、エンティティ・概念・出典ページが自動生成され、Obsidian Vault に蓄積される。蓄積された知識は次のセッションでも hot cache により即座に活用でき、知識が雪だるま式に増える体験を提供する。

### 目的
- claude-obsidian v1.9 相当の機能を Kiro ハーネスとして再実装する
- Kiro の制約（スキルシステムなし・フックイベント差異）を乗り越える移植パターンを確立する
- 移植成果物を **`kiro-obsidian`（専用テンプレートリポジトリ）** として分離公開し、他のユーザーが複製するだけで同じ体験を得られるようにする

### リポジトリ構成
本プロジェクトは2リポジトリ構成をとる:

| リポジトリ | 役割 |
|-----------|------|
| **claude-obsidian-for-kiro**（本リポジトリ） | 開発リポジトリ。移植作業・検証・ドキュメントを管理 |
| **kiro-obsidian**（テンプレートリポジトリ） | 公開用 GitHub Template。ユーザーが複製して使う成果物 |

---

## ターゲットユーザー（ペルソナ定義）

本プロジェクトには3つのペルソナが存在する。それぞれが関わるリポジトリと責任範囲が異なる。

### ペルソナ1: メタ管理者（第二の脳構築ハーネスのメタ管理者）

**関わるリポジトリ**: `claude-obsidian-for-kiro`（本リポジトリ）

**役割**: claude-obsidian（オリジナル）から「Kiro + Obsidian で第二の脳を構築するためのテンプレート」を生成・更新する。claude-obsidian の新バージョンリリース時に追従する。

**特性**:
- Kiro ハーネスの仕組み（steering・hooks・agents）を深く理解している
- claude-obsidian の移植パターンを確立・改善できる
- 本リポジトリの SDD フローで開発を進める

---

### ペルソナ2: ハーネス管理者（第二の脳構築ハーネスの管理者）

**関わるリポジトリ**: `kiro-obsidian`（テンプレートリポジトリを複製したもの）

**役割**: `kiro-obsidian` テンプレートをベースに自分の環境向けにハーネスをカスタマイズ・メンテナンスする。Vault の設計方針（Methodology Mode 等）を決定し、steering ファイルを必要に応じて調整する。

**特性**:
- Kiro IDE を日常的に使用している（会社環境で Claude Code が使用不可）
- Obsidian による知識管理に関心がある
- ハーネスに問題が起きたとき steering ファイルを読んで自力でデバッグできる必要がある
- **日本語圏**（→ steering ファイル・agents・hooks がすべて日本語であることが必須）

---

### ペルソナ3: ハーネス利用者（第二の脳構築ハーネスの利用者）

**関わるリポジトリ**: `kiro-obsidian`（複製済み）の Vault

**役割**: ハーネスが提供するスキル（ingest・query・lint 等）を使って日々の知識を蓄積・活用する。Kiro に自然言語で話しかけるだけで wiki が育っていく体験を得る。

**特性**:
- ハーネスの内部構造を知らなくても使える
- ソースを `.raw/` に投入して「ingest して」と言うだけで wiki が拡充される
- 「X について何を知ってる？」と聞くだけで蓄積した知識から回答が得られる
- **日本語圏**（→ wiki ページ・hot cache・回答がすべて日本語で生成されることが必須）

---

> **注**: 現実には1人の人物がペルソナ1〜3を兼ねる場合がほとんど。ペルソナを分けるのは各役割に必要な能力と体験の質を明確にするため。

---

## ユーザージャーニー

### ペルソナ1：メタ管理者のジャーニー

#### 初回セットアップ
```
1. platform-harness-engineering の kiro-template を参考に
   claude-obsidian-for-kiro（本リポジトリ）を作成
2. claude-obsidian-main を参照元として .kiro/ を移植（日本語化を含む）
3. 動作検証（Kiro 上で scaffold → ingest → query → lint）
4. kiro-obsidian テンプレートリポジトリを生成・公開
```

#### 継続運用

**パターンA：claude-obsidian がバージョンアップしたとき**
```
1. claude-obsidian-main を新バージョンで上書き更新
2. git diff で変更点を確認
3. .kiro/ 側の対応ファイルに差分を反映（翻訳も更新）
4. 動作検証 → kiro-obsidian に publish
```

**パターンB：ハーネス管理者から issue・PR が届いたとき**
```
1. issue の内容を確認
   - kiro-obsidian 固有のバグ → .kiro/ を修正して publish
   - Kiro 仕様変更の影響 → hooks/agents を修正
   - 翻訳の問題 → steering を修正
2. 修正内容を動作検証
3. kiro-obsidian に publish・リリース
4. issue クローズ
```

**パターンC：Kiro 自体がアップデートしたとき**
```
1. hooks の fileEdited / userTriggered の挙動変化を確認
2. agents の frontmatter 仕様変更を確認
3. 修正が必要なら対応 → publish
```

**⚠️ 未解決事項**：
- claude-obsidian の更新追従は手動 diff か自動化か
- kiro-obsidian への publish は手動 rsync か CI か

---

### ペルソナ2：ハーネス管理者のジャーニー

#### 初回セットアップ
```
1. kiro-obsidian を GitHub Template から複製（自分のリポジトリとして）
2. Kiro IDE でリポジトリを開く
3. bash bin/setup-vault.sh を実行（Obsidian 設定を初期化）
4. Kiro に「vault を設定したい」と伝えて scaffold
5. Vault の目的と Methodology Mode を決定
6. 必要に応じて steering を自己カスタマイズ
```

#### 継続運用

**日常（ハーネス利用者として）**
```
→ ペルソナ3のジャーニーをそのまま実施
```

**問題対応**
```
1. 挙動がおかしいとき
   → .kiro/steering/ の日本語 steering を読んでデバッグ
   → .kiro/hooks/ の JSON を確認
2. 自己解決できないとき
   → kiro-obsidian に issue を上げる（メタ管理者へのフィードバック）
3. 自分で修正できたとき
   → 自リポジトリで修正
   → 共通改善なら kiro-obsidian に PR を送る
```

**kiro-obsidian の更新取り込み**
```
1. kiro-obsidian の新バージョンリリースを確認
2. 変更内容（リリースノート）を確認
3. 自分のカスタマイズと競合しないか確認
4. 競合なし → rsync 等で取り込み
5. 競合あり → 手動マージ
```

**⚠️ 未解決事項**：
- 自リポジトリのカスタマイズと kiro-obsidian 更新のマージ方法
- kiro-obsidian を git upstream として管理するか、手動 rsync にとどめるか
- カスタマイズ可能な範囲の指針（steering 全面書き換えは追従困難）

---

### ペルソナ3：ハーネス利用者のジャーニー

#### 初回セットアップ
```
ハーネス管理者がセットアップ済みの Vault を Kiro + Obsidian で開く
（自身がハーネス管理者を兼ねる場合はペルソナ2の初回完了後から継続）
```

#### 日常利用

**知識の取り込み（ingest）**
```
1. 読んだ記事・PDF・メモを .raw/ に投入
2. Kiro に「[ファイル名] を ingest して」と伝える
3. wiki ページが 8〜15 件自動生成される
4. index.md / log.md / hot.md が自動更新される
5. git auto-commit が走る（手動操作不要）
```

**知識の活用（query）**
```
1. 「X について何を知ってる？」と Kiro に聞く
2. hot.md → index.md → 個別ページの順に読んで回答が返る
3. wikilink 引用付きで出典が明確
```

**wiki の維持（lint）**
```
1. 10〜15 件 ingest したら「lint the wiki」を実行
2. 孤立ページ・デッドリンク・矛盾を検出
3. Kiro が修正提案を出す
```

**セッション間の継続性**
```
1. 次のセッションを開始
2. wiki-core.md の常時ロードにより hot.md が自動参照される
3. 前回の文脈から即座に再開できる
```

#### 長期的な体験
```
Month 1:  wiki が数十ページに成長。基本的な質問に答えられる
Month 3:  クロスリファレンスが充実。矛盾の自動検出が価値を出す
Month 6+: 知識が大量蓄積 → wiki-fold / wiki-retrieve が必要になる（P2）
```

**解決済み・要検討**：
- セッション開始時の hot.md 読み込み: `wiki-session-start.json`（userTriggered hook）で手動トリガーにより暫定実装。詳細は functional-design.md 参照
- hot.md が肥大化したときの対処: wiki-fold（P2）で対応予定

---

### ジャーニー間の関係

```
メタ管理者（claude-obsidian-for-kiro）
  │
  │ claude-obsidian 更新追従
  │ ハーネス管理者からの issue/PR 取り込み
  │ Kiro 仕様変更対応
  ↓
kiro-obsidian（テンプレートリポジトリ）
  │                    ↑
  │ 複製               │ issue / PR
  ↓                    │
ハーネス管理者の自リポジトリ ──┘
  │
  │ 兼任（多くの場合）
  ↓
ハーネス利用者（日々の ingest / query / lint）
```

---

## 成功指標（KPI）

### 機能完成度（開発リポジトリ）
- claude-obsidian のコアスキル（wiki/wiki-ingest/wiki-query/wiki-lint）が Kiro 上で動作すること
- Kiro steering の `inclusion: manual` によるオンデマンドスキル起動が機能すること
- `.kiro/hooks/` によるファイル変更後の自動コミットが動作すること

### 品質（開発リポジトリ）
- 移植後の vault scaffold → ingest → query → lint の一連フローが動作確認済みであること
- scripts/（wiki-lock.sh, wiki-mode.py 等）が Kiro 環境で正常動作すること

### テンプレートリポジトリ（kiro-obsidian）
- GitHub Template Repository として公開されていること
- ユーザーが複製した直後に Kiro で開いて動作すること（開発用ファイルを含まないクリーンな状態）

### 定量的 KPI（要検討）
> 現時点では定量目標を設定できる計測手段がないため未定。ハーネス管理者・利用者からフィードバックを収集した後に設定する。
> 候補指標: ingest 所要時間（ソース1件あたり）・1ヶ月後の wiki ページ数・scaffold からの初回 ingest 完了率

---

## 機能要件

### コア機能（MVP・P0）

#### 1. Vault セットアップ（wiki scaffold）

**ユーザーストーリー**:
Kiro ユーザーとして、新しい知識管理 vault を素早く構築するために、`#skill-wiki` を参照して scaffold を実行したい。

**受け入れ条件**:
- [ ] `.kiro/steering/skill-wiki.md`（inclusion: manual）が存在し、scaffold 手順を定義している
- [ ] "vault を作って" / "wiki を設定して" 等の自然言語で scaffold が開始できる
- [ ] wiki/ の標準フォルダ構造（sources/, entities/, concepts/, meta/）が作成される
- [ ] wiki/index.md, wiki/log.md, wiki/hot.md, wiki/overview.md が初期化される
- [ ] .obsidian/snippets/vault-colors.css が作成される

**優先度**: P0

#### 2. ソース取り込み（ingest）

**ユーザーストーリー**:
Kiro ユーザーとして、.raw/ にドロップしたソースを wiki に統合するために、"ingest [ファイル名]" と言って wiki ページを自動生成したい。

**受け入れ条件**:
- [ ] `.kiro/steering/skill-wiki-ingest.md`（inclusion: always または manual）が存在する
- [ ] 1ソースから 8〜15 の wiki ページ（source/entity/concept）が生成される
- [ ] wiki/index.md, wiki/log.md, wiki/hot.md が ingest 後に更新される
- [ ] 矛盾がある場合 `[!contradiction]` callout が生成される
- [ ] バッチ ingest 時は `.kiro/agents/wiki-ingest.md` による並列処理が動作する

**優先度**: P0

#### 3. 知識クエリ（query）

**ユーザーストーリー**:
Kiro ユーザーとして、蓄積した wiki から回答を得るために、自然言語で質問して引用付きの回答を受け取りたい。

**受け入れ条件**:
- [ ] `.kiro/steering/skill-wiki-query.md`（inclusion: manual）が存在する
- [ ] hot.md → index.md → 個別ページ の順に読んでトークンコストを最小化する
- [ ] 回答に wiki ページの wikilink 引用が含まれる

**優先度**: P0

#### 4. Wiki ヘルスチェック（lint）

**ユーザーストーリー**:
Kiro ユーザーとして、wiki の健全性を維持するために、"lint the wiki" と言って孤立ページ・デッドリンク・矛盾を検出したい。

**受け入れ条件**:
- [ ] `.kiro/steering/skill-wiki-lint.md`（inclusion: manual）が存在する
- [ ] 8 カテゴリ（孤立ページ・デッドリンク・stale claims 等）のチェックが実行される
- [ ] `.kiro/agents/wiki-lint.md` による並列エージェントが動作する

**優先度**: P0

#### 5. Hot Cache 管理

**ユーザーストーリー**:
Kiro ユーザーとして、前のセッションの文脈を引き継ぐために、セッション開始時に wiki/hot.md が自動的に読まれてほしい。

**受け入れ条件**:
- [ ] `.kiro/steering/wiki-core.md`（inclusion: always）に「セッション開始時に hot.md を読む」指示が含まれる
- [ ] hot.md が存在する場合、セッション開始時にサイレントに読み込まれる
- [ ] ingest / query 後に hot.md が更新される

**優先度**: P0

#### 6. 自動コミット Hook

**ユーザーストーリー**:
Kiro ユーザーとして、wiki の変更を手動で git commit せずに済むよう、wiki/ ファイル変更後に自動コミットが走ってほしい。

**受け入れ条件**:
- [ ] `.kiro/hooks/wiki-auto-commit.json`（fileEdited, patterns: ["wiki/**", ".raw/**"]）が存在する
- [ ] wiki-lock が保持中の場合はコミットを延期する（`scripts/wiki-lock.sh` 連携）
- [ ] コミットメッセージが "wiki: auto-commit YYYY-MM-DD HH:MM" 形式になっている

**優先度**: P0

### 拡張機能（P1）

#### 7. Methodology Mode 対応（wiki-mode）
- generic / LYT / PARA / Zettelkasten の 4 モード切り替え
- `.kiro/steering/skill-wiki-mode.md`（inclusion: manual）
- `scripts/wiki-mode.py` によるページパスルーティング

#### 8. 自律リサーチ（autoresearch）
- `.kiro/steering/skill-autoresearch.md`（inclusion: manual）
- Web 検索 → 合成 → wiki 取り込みの 3 ラウンドループ

#### 9. 会話保存（save）
- `.kiro/steering/skill-save.md`（inclusion: manual）
- セッションの会話を wiki ノートとして保存

### 将来機能（P2）

#### 10. ハイブリッド検索（wiki-retrieve）
- BM25 + コサイン再ランクの 3 層検索パイプライン
- `scripts/bm25-index.py`, `scripts/retrieve.py`, `scripts/rerank.py`

#### 11. think フレームワーク
- 10 原則思考ループ（OBSERVE-THINK-CONNECT-CREATE-GROW）
- `.kiro/steering/skill-think.md`（inclusion: manual）

#### 12. DragonScale Memory（opt-in）
- ログロールアップ・決定論的アドレス・セマンティックタイリング
- `scripts/allocate-address.sh`, `scripts/boundary-score.py`

---

## 非機能要件

### 言語
- **ハーネス部分**（`.kiro/steering/`・`.kiro/hooks/`・`.kiro/agents/`）はすべて**日本語**で記述すること
  - ユーザーが運用上の問題が起きたときに内容を自力で読んで理解できることを優先する
  - claude-obsidian 原本（英語）からの移植時は、フォーマット変換と同時に**翻訳**を行う
- **Wiki 部分**（`wiki/`・`_templates/`・seeded コンテンツ）はすべて**日本語**で記述すること
  - LLM が生成する wiki ページも日本語で出力されるよう steering に明示的に指示する
- **テンプレートの README・CLAUDE.md** は日本語で記述すること

### 移植忠実性
- claude-obsidian v1.9.2 の コアスキル（wiki/ingest/query/lint）と同等の動作を実現すること
- Kiro 固有の制約により実現不可な機能（PostCompact フック等）は明示的に文書化すること

### Kiro 互換性
- `.kiro/` ディレクトリ構造が Kiro IDE の仕様に準拠すること
- steering ファイルの frontmatter（`inclusion: always/manual`）が正しく機能すること
- hooks の `fileEdited` / `userTriggered` イベントが意図通りに発火すること

### スクリプト互換性
- `scripts/` の bash/Python スクリプトが macOS/Linux 環境で動作すること
- Python 3.10+、bash 4.0+ を前提とする

### 保守性
- Kiro の仕様変更に対して各コンポーネントが独立して更新できること
- claude-obsidian の将来バージョンアップを取り込む際の差分が最小になる構造にすること

---

## スコープ外

- Obsidian プラグイン（community plugins）のインストール自動化（ユーザーが手動で行う）
- Obsidian Sync や iCloud 等のクロスデバイス同期設定
- claude-obsidian の canvas スキル（Obsidian Canvas は Kiro 環境では優先度低）
- Kiro CLI (`kiro-cli`) を使った操作（Kiro IDE GUI 内での完結を前提とする）
- DragonScale Memory の完全実装（P2 opt-in として後回し）
