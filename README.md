# RAG_met — RAGベース気象データAIチャットボット

## 概要

RAG（Retrieval-Augmented Generation：検索拡張生成）技術を使って、
気象・業務データに基づいた回答を生成するAIチャットボット。

Claude API（Anthropic）をLLMとして使用し、気象教科書・気象庁Webデータを
ベクトルDBに格納して検索・回答生成を行う。

- **プロジェクトディレクトリ**: `~/projects/RAG_met/`
- **LLM**: Claude API（Anthropic）またはローカルLLM（Bonsai / Ollama / Qwen3 等）
- **ベクトルDB**: ChromaDB（ローカル）
- **Webインターフェース**: `http://127.0.0.1:8081`

---

## 技術スタック

| 項目 | 採用技術 |
|------|---------|
| LLM | Claude API（Haiku / Sonnet 切替可） |
| RAGフレームワーク | LangChain |
| ベクトルDB | ChromaDB（ローカル） |
| 埋め込みモデル | intfloat/multilingual-e5-small（ローカル・日本語対応） |
| Webサーバー | Python標準ライブラリ（http.server） |
| 言語 | Python 3.x |

---

## ディレクトリ構成

```
RAG_met/
├── README.md               # このファイル
├── CLAUDE.md               # Claude向け指示
├── plan.md                 # 実装計画
├── requirements.txt        # 依存パッケージ
├── .env                    # 環境変数（APIキー等・コミット禁止）
├── .env.example            # 環境変数テンプレート
├── data/                   # 元データ
│   ├── *.pdf               # 気象教科書PDF
│   └── jma_yougo/          # 気象庁予報用語（スクレイピング取得）
├── db/                     # ChromaDBベクトルデータ（コミット禁止）
└── src/
    ├── ingest.py           # PDFデータ取り込み・ベクトル化
    ├── ingest_text.py      # テキストデータ追加取り込み
    ├── scrape_jma.py       # 気象庁HPスクレイピング
    ├── query.py            # 検索・回答生成（RAGコア）
    ├── app.py              # CLIインターフェース
    └── web_app.py          # Webインターフェース
```

---

## セットアップ手順

```bash
cd ~/projects/RAG_met

# 1. 仮想環境の作成と有効化
python3 -m venv venv
source venv/bin/activate

# 2. 依存パッケージのインストール
pip install -r requirements.txt
pip install langchain-text-splitters

# 3. 環境変数の設定
cp .env.example .env
# .env を編集して ANTHROPIC_API_KEY を設定

# 4. データの取り込み（PDF）
python src/ingest.py

# 5. 気象庁予報用語の取得・取り込み
python src/scrape_jma.py
python src/ingest_text.py
```

---

## 登録データ一覧

| ファイル名 | 正式タイトル |
|-----------|-------------|
| all.pdf | 2026季節予報作業と気象情報の利活用に関する近年の取り組み |
| all-2.pdf | 2013季節予報作業指針 |
| all-3.pdf | 2019 2週間気温予報とその活用 |
| textbook_synop_basic_revision_20250331.pdf | 総観気象学（基礎編） |
| textbook_synop_advance_20210831.pdf | 総観気象学（応用編） |
| textbook_synop_theory_20220318.pdf | 総観気象学（理論編） |
| textbook_meso_v2.1.pdf | 中小規模気象学 |
| kaisetsu_jikkyo.pdf | 実況資料の解説 |
| kaisetsu_yosou.pdf | 予想資料の解説 |
| warning_riskmap.pdf | 気象警報とキキクル |
| yougo_senmon.pdf | 気象専門用語集 |
| jma_yougo/*.txt | 気象庁予報用語集（Web取得・20カテゴリ） |

---

## 起動方法

### Claude API を使う場合

```bash
source venv/bin/activate
python src/web_app.py
# → ブラウザで http://127.0.0.1:8081 を開く
```

### ローカルLLM（Bonsai等）を使う場合

```bash
# ターミナル①: Bonsaiサーバーを起動
cd ~/projects/1bit_LLM && bash start_bonsai.sh

# ターミナル②: Webアプリを起動（.env で LLM_BACKEND=local に設定済みであること）
cd ~/projects/RAG_met && source venv/bin/activate && python src/web_app.py
# → ブラウザで http://127.0.0.1:8081 を開く
```

> **ポートに注意**: Bonsaiは 8080 を使用するため、Webアプリは **8081** で起動する。

### CLIインターフェース

```bash
source venv/bin/activate
python src/app.py
```

---

## .env 設定項目

```
ANTHROPIC_API_KEY=sk-ant-...            # Claude APIキー（Claude使用時は必須）
CLAUDE_MODEL=claude-haiku-4-5-20251001  # 使用モデル（claude-sonnet-4-6 も可）
LLM_BACKEND=claude                      # claude / local（ローカルLLM使用時は local）
RAG_MODE=hybrid                         # hybrid or strict（下記参照）
CHROMA_DB_PATH=./db                     # ベクトルDB保存先
DATA_DIR=./data                         # データディレクトリ

# ローカルLLM使用時のみ設定
LOCAL_API_BASE=http://localhost:8080/v1  # Bonsai / Ollama 等のエンドポイント
LOCAL_MODEL=bonsai-8b                    # モデル名
```

---

## RAGモードの切り替え

| モード | 設定値 | 動作 | 用途 |
|--------|--------|------|------|
| hybrid | `RAG_MODE=hybrid` | LLM知識＋登録データを併用し、**2セクションに分けて回答** | Claude API使用時（推奨） |
| strict | `RAG_MODE=strict` | 登録データのみから回答（ハルシネーション防止） | ローカルLLM使用時（推奨） |

> **注意**: Bonsai・Qwen3 などの小型ローカルモデルは `hybrid` モードの2セクション指示に従えない場合がある。
> RAGからの回答をAIからの回答として誤表示するなどの問題が発生するため、ローカルLLMでは `strict` を推奨。

---

## RAGの基本フロー

```
[ユーザーの質問]
      ↓
[質問をベクトル化（multilingual-e5-small）]
      ↓
[ChromaDBから関連チャンクを検索（Top-5）]
      ↓
[コンテキスト構築（出典タイトル＋ページ番号付き）]
      ↓
[Claude APIに質問＋コンテキストを送信]
      ↓
[回答を生成（hybridモードはLLM知識とRAGを分けて表示）]
```

---

## 参考記事との実装比較

参考記事：[RAGで自社データAIチャットボットを作る：Claude Code×ベクトルDB実践ガイド](https://note.com/clever_dog/n/ne9a2c46b0367)

| 項目 | 参考記事 | 本プロジェクト |
|------|---------|--------------|
| インターフェース | Claude Code（CLI） | カスタムWebUI（標準ライブラリのみ） |
| 埋め込みモデル | 記事未明示（汎用） | multilingual-e5-small（日本語特化・ローカル） |
| RAGモード | strictのみ | hybrid / strict 切替可能 |
| 回答形式 | 単一回答 | hybridモードでLLM知識とRAG知識を分離表示 |
| データソース | ローカルファイルのみ | PDF＋気象庁WebスクレイピングによるTXT |
| 出典表示 | ファイル名のみ | 正式タイトル＋ページ番号 |
| ローカルLLM対応 | 非対応 | `LLM_BACKEND=local` で対応（Bonsai / Ollama / Qwen3 等 OpenAI互換API） |

---

## 参考資料

- [Anthropic Claude API ドキュメント](https://docs.anthropic.com/)
- [LangChain ドキュメント](https://python.langchain.com/)
- [ChromaDB ドキュメント](https://docs.trychroma.com/)
- [気象庁予報用語](https://www.jma.go.jp/jma/kishou/know/yougo_hp/sakuin.html)
