# kiro-obsidian

**Kiro IDE + Obsidian で知識が複利で増える第二の脳**

ソースをドロップして「ingest して」と言うだけで、8〜15 の相互リンクされた wiki ページが自動生成されます。蓄積された知識は次のセッションでも Hot Cache で即座に活用でき、知識が雪だるま式に増える体験を提供します。

---

## 何ができるか

| 操作 | やること |
|------|---------|
| **ingest** | `.raw/` にソースを入れて「ingest して」と言う → wiki ページが 8〜15 自動生成 |
| **query** | 「[トピック] について教えて」→ wiki から引用付きで回答 |
| **lint** | 「wiki を lint して」→ 孤立ページ・デッドリンク・矛盾を検出 |
| **autoresearch** | 「[トピック] をリサーチして」→ Web 検索・ソース取得・wiki 登録を自動実行 |
| **save** | 「保存して」→ 会話の洞察を wiki ページとして永続化 |
| **wiki-mode** | `#skill-wiki-mode` → Vault の整理方法論を選択（LYT / PARA / Zettelkasten / Generic） |

---

## 前提条件

- [Kiro IDE](https://kiro.dev/) がインストールされていること
- [Obsidian](https://obsidian.md/) v1.6 以上がインストールされていること
- Git がインストールされていること（`git --version` で確認）
- Python 3.10 以上がインストールされていること（scripts/ に必要）

---

## セットアップ

### 1. テンプレートからリポジトリを作成

GitHub の「Use this template」ボタンから新しいリポジトリを作成します。

### 2. リポジトリをクローン

```bash
git clone https://github.com/[あなたのユーザー名]/[リポジトリ名]
cd [リポジトリ名]
```

### 3. Obsidian で Vault を開く

Obsidian を起動し、「フォルダとして開く」でクローンしたフォルダをルートとして開きます。

初回は以下の推奨プラグインを手動でインストールしてください:
- **Templater** — `_templates/` のテンプレートを使用するため
- **Dataview** — `wiki/meta/dashboard.md` のクエリを使用するため（任意）

### 4. Vault を初期化（任意）

```bash
bash bin/setup-vault.sh
```

Graph View の設定・カラーグループ・CSS スニペットを適用します。

### 5. Kiro IDE で開いて使い始める

Kiro IDE でこのフォルダを開きます。あとは自然言語で操作するだけです:

```
.raw/ にドキュメントを入れて、ingest してください
```

---

## 基本的な使い方

### ソースの取り込み（ingest）

1. `.raw/` フォルダに任意のファイルを置く（PDF、テキスト、Markdown など）
2. Kiro のチャットで指示:

```
report.pdf を ingest してください
```

Kiro が自動で wiki ページを作成し、`wiki/index.md`・`wiki/log.md`・`wiki/hot.md` を更新します。

### 知識への質問（query）

```
機械学習における過学習について、wiki から教えてください
```

### wiki の健全性チェック（lint）

```
wiki を lint して、孤立ページとデッドリンクを教えてください
```

---

## ディレクトリ構造

```
kiro-obsidian/
├── .kiro/
│   ├── steering/     # Kiro が参照するスキル定義（常時・手動）
│   ├── hooks/        # 自動コミットなどのイベント駆動自動化
│   └── agents/       # 並列 ingest・lint 用サブエージェント
├── wiki/             # Obsidian Vault（LLM が作成・維持）
├── .raw/             # ← ソースをここに入れる
├── _templates/       # Obsidian Templater テンプレート
├── scripts/          # bash/Python ヘルパースクリプト
├── bin/              # セットアップスクリプト
└── tests/            # スクリプト単体テスト
```

---

## スキルの呼び出し方

Kiro では `/skill-name` の代わりに `#skill-name` でスキルを参照します:

| 操作 | Kiro への指示 |
|------|-------------|
| Vault の初期設定 | 「`#skill-wiki` を参照して vault を scaffold してください」 |
| ingest（自動） | 「[ファイル名] を ingest してください」（常時ロード済み） |
| query | 「`#skill-wiki-query` を参照して [質問] を教えてください」 |
| lint | 「`#skill-wiki-lint` を参照して wiki を lint してください」 |
| autoresearch | 「`#skill-autoresearch` を参照して [トピック] をリサーチしてください」 |
| save | 「`#skill-save` を参照してこの会話を保存してください」 |
| wiki-mode 設定 | 「`#skill-wiki-mode` を参照して Vault のモードを設定してください」 |

---

## ライセンス

MIT License

元プロジェクト: [claude-obsidian](https://github.com/AI-Marketing-Hub/claude-obsidian) by AI Marketing Hub
