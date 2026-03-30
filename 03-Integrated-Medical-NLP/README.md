# Integrated-Medical-NLP

Biomedical-NER（疾患・薬剤抽出）とMedical-RAG（医療QA）を統合したシステム。

---

## 目的

医療テキストからエンティティ（疾患・薬剤）を抽出し、それを加味して質問応答を行う統合パイプラインを構築する。

---

## 技術スタック

| 技術 | 用途 |
|------|------|
| BioBERT v1.1 | NER（疾患・薬剤抽出） |
| GLM-4.7 | 質問応答・評価 |
| MiniLM-L6-v2 | 埋め込み |
| ChromaDB | ベクトルDB |

---

## プロジェクト構成

```
Integrated-Medical-NLP/
├── plan.md
├── README.md
├── requirements.txt
├── notebooks/
│   ├── 01_ner_demo.ipynb     # NER単体デモ（ミニマム）
│   ├── 02_rag_demo.ipynb     # RAG単体デモ（ミニマム）
│   └── 03_integrated.ipynb   # ★統合パイプライン（詳細）
├── models/
│   └── biobert-ner-bc5cdr/   # 学習済みNERモデル（01-Biomedical-NERからコピー）
└── .env
```

---

## ノートブック概要

### 01_ner_demo.ipynb（ミニマム）
BioBERTでテキストから疾患・薬剤を抽出する簡単なデモ。

### 02_rag_demo.ipynb（ミニマム）
ChromaDB + GLM-4で医療QAを行う簡単なデモ。

### 03_integrated.ipynb（★本命）
NERとRAGを統合したパイプライン。

**処理フロー**:
```
質問: "Patient has diabetes and takes insulin. What are the side effects?"

▼ Step 1: NER（エンティティ抽出）
diabetes(疾患), insulin(薬剤) を抽出

▼ Step 2: エンティティを加味して検索
拡張クエリ: "diabetes insulin side effects"

▼ Step 3: LLMで回答生成
```

---

## 使用方法

### 1. 環境構築

```bash
# パッケージインストール
pip install -r requirements.txt

# APIキー設定（.envファイル）
OPENAI_API_KEY=your_key_here
```

### 2. モデルのコピー（必須）

このプロジェクトを動作させるには、01-Biomedical-NERの学習済みモデルをコピーする必要があります。

**Windows**:
```bash
# コマンドプロンプトまたはPowerShell
xcopy /E /I /Y "..\01-Biomedical-NER\results\biobert-ner-bc5cdr" "models\biobert-ner-bc5cdr"
```

**または手動で**:
1. `01-Biomedical-NER/results/biobert-ner-bc5cdr/` フォルダをコピー
2. `03-Integrated-Medical-NLP/models/` フォルダにペースト

### 3. ノートブック実行

1. `01_ner_demo.ipynb` - NERの動作確認
2. `02_rag_demo.ipynb` - RAGの動作確認
3. `03_integrated.ipynb` - 統合パイプラインの実行

---

## 重要な技術的注意点

### aggregation_strategy="first"

BioBERTはWordPieceトークナイザーを使用するため、`aggregation_strategy="first"` を使用します。

- ❌ `aggregation_strategy="simple"` - サブワードが結合されない
- ✅ `aggregation_strategy="first"` - 正しく結合される

```python
ner_pipeline = pipeline(
    "token-classification",
    model="models/biobert-ner-bc5cdr",
    aggregation_strategy="first",  # 重要
    device=0 if torch.cuda.is_available() else -1
)
```

---

## 依存関係

| 依存元 | コピー先 | 用途 |
|--------|---------|------|
| `01-Biomedical-NER/results/biobert-ner-bc5cdr/` | `models/biobert-ner-bc5cdr/` | NERモデル |
| `02-Medical-RAG/results/chroma_db/` | （参照のみ） | ベクトルDB |

---

## 成功基準

- [x] NERでエンティティ抽出ができる
- [x] RAGで質問応答ができる
- [x] 統合パイプラインが動作する
- [x] README.md完成
- [x] デモノートブック完成

---

## 免責事項

- 本プロジェクトは**教育的・ポートフォリオ目的**です
- **医療診断や治療決定への使用は厳禁**です
- 使用するデータセットは研究用であり、実臨床データとは異なります
- 医療に関する相談は、必ず資格のある医療専門家に行ってください

## ライセンス

MIT License

---

*作成日：2026-03-29*
*更新日：2026-03-30（モデルコピー手順追加、aggregation_strategy明記）*
