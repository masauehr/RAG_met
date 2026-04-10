# RAG_met — RAGベース気象データAIチャットボット

## 概要

RAG（Retrieval-Augmented Generation：検索拡張生成）技術を使って、
気象・業務データに基づいた回答を生成するAIチャットボットを構築するプロジェクト。

Claude（Anthropic）をAIモデルとして使用し、ローカルのドキュメントやデータを
ベクトルDBに格納して検索・回答生成を行う。

## 技術スタック

| 項目 | 技術 |
|------|------|
| AIモデル | Claude (Anthropic API) |
| RAGフレームワーク | LangChain |
| ベクトルDB | ChromaDB（ローカル）|
| 埋め込みモデル | sentence-transformers / OpenAI Embeddings |
| 言語 | Python 3.x |
| インターフェース | CLI / Web（予定） |

## ディレクトリ構成

```
RAG_met/
├── README.md         # このファイル
├── CLAUDE.md         # Claude向け指示
├── plan.md           # 実装計画
├── data/             # 元データ（テキスト、PDF等）
├── db/               # ベクトルDB保存先
├── src/
│   ├── ingest.py     # データ取り込み・ベクトル化
│   ├── query.py      # 検索・回答生成
│   └── app.py        # メインアプリ
├── requirements.txt  # 依存パッケージ
└── .env.example      # 環境変数テンプレート
```

## セットアップ手順

```bash
# 1. 仮想環境の作成
python -m venv .venv
source .venv/bin/activate

# 2. 依存パッケージのインストール
pip install -r requirements.txt

# 3. 環境変数の設定
cp .env.example .env
# .env を編集して ANTHROPIC_API_KEY を設定

# 4. データの取り込み
python src/ingest.py

# 5. チャットボットの起動
python src/app.py
```

## RAGの基本フロー

```
[ユーザーの質問]
      ↓
[質問をベクトル化]
      ↓
[ベクトルDBから関連ドキュメントを検索]
      ↓
[Claudeに質問 + 検索結果を渡してプロンプト生成]
      ↓
[Claudeが回答を生成]
      ↓
[ユーザーに回答を返す]
```

## 参考資料

- [RAG × Claude Code で自社データAIチャットボット構築ガイド](https://note.com/clever_dog/n/ne9a2c46b0367)
- [Anthropic Claude API ドキュメント](https://docs.anthropic.com/)
- [LangChain ドキュメント](https://python.langchain.com/)
