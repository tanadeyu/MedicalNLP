# Medical NLP: 医療テキスト分析システム

BioBERTによる固有表現抽出（NER）とRAGによる医療QAを統合した、包括的な医療NLPシステム。
Biomedical-NER + Medical-RAG + Integrated-Medical-NLP

---

## 概要

本プロジェクトは、以下の3つの構成要素からなる医療NLPシステムです：

1. **Biomedical-NER**: 医療テキストから疾患・薬剤を抽出
2. **Medical-RAG**: MedQuADデータセットによる医療QAシステム
3. **Integrated-Medical-NLP**: NER + RAG の統合パイプライン

---

## プロジェクト構成

```
Medical-NLP/
├── 01-Biomedical-NER/          # BioBERTによる疾患・薬剤抽出
│   ├── notebooks/
│   │   ├── 01_data_preparation.ipynb    # データ準備
│   │   ├── 02_model_training.ipynb      # モデル学習
│   │   ├── 03_evaluation.ipynb          # 評価
│   │   └── 04_demo.ipynb                # デモ
│   ├── data/                            # データセット
│   ├── results/
│   │   └── biobert-ner-bc5cdr/          # 学習済みモデル
│   ├── plan.md                          # プラン/仕様書
│   ├── requirements.txt                 # 依存パッケージ
│   ├── 注意事項.md
│   └── README.md
│
├── 02-Medical-RAG/             # 医療QAシステム
│   ├── notebooks/
│   │   ├── 00_medquad_exploration.ipynb  # データ探索
│   │   ├── 01_prepare.ipynb              # データ準備
│   │   ├── 02_build_db.ipynb             # ベクトルDB構築
│   │   └── 03_demo.ipynb                 # デモ
│   ├── data/                            # データセット
│   ├── results/
│   │   └── chroma_db/                    # ベクトルDB
│   ├── temp_medquad/                    # 一時データ
│   ├── plan.md                          # プラン/仕様書
│   ├── medical_rag.py仕様書.md
│   ├── requirements.txt                 # 依存パッケージ
│   ├── .env.sample                      # 環境変数テンプレート
│   └── README.md
│
└── 03-Integrated-Medical-NLP/  # 統合システム
    ├── notebooks/
    │   ├── 01_ner_demo.ipynb             # NERデモ
    │   ├── 02_rag_demo.ipynb             # RAGデモ
    │   └── 03_integrated.ipynb           # 統合パイプライン
    ├── models/
    │   └── biobert-ner-bc5cdr/          # NERモデル（01からコピー）
    ├── plan.md                          # プラン/仕様書
    ├── requirements.txt                 # 依存パッケージ
    └── README.md
```

---

## 01-Biomedical-NER: 医療固有表現抽出

### 目的

BioBERT v1.1をBC5CDrデータセットでファインチューニングし、医療テキストから**疾患**と**薬剤**を抽出します。

### 技術スタック

| 項目 | 内容 |
|------|------|
| モデル | BioBERT v1.1 (dmis-lab/biobert-v1.1) |
| タスク | BERT Token Classification |
| データセット | bigbio/bc5cdr（正しいラベル順序） |
| フレームワーク | PyTorch 2.5.1 + Transformers |

### エンティティ定義

```
label_list = ['O', 'B-Chemical', 'I-Chemical', 'B-Disease', 'I-Disease']
```

| エンティティ | 説明 | 色 |
|------------|------|---|
| Disease | 疾患名 | 🟠 #ffe6b3（薄いオレンジ） |
| Chemical | 薬剤名 | 🔵 #b3d9ff（薄い青） |

### 性能

| 指標 | スコア |
|------|--------|
| **F1 (micro)** | **0.9733** |
| Disease F1 | 約0.97 |
| Chemical F1 | 約0.98 |

### デモ出力例

```
テキスト: "Patient has diabetes mellitus and takes insulin for treatment."

抽出結果:
- Disease: diabetes mellitus 🟠 (スコア: 0.8559)
- Chemical: insulin 🔵 (スコア: 0.6995)
```

### 重要な技術的注意点

#### 1. データセット選択

- ✅ **使用**: bigbio/bc5cdr（正しいラベル順序）
- ❌ **非使用**: tner/bc5cdr（ラベルが逆転しているバグあり）

詳細は `注意事項.md` を参照。

#### 2. aggregation_strategy="first"

BioBERTはWordPieceトークナイザーを使用するため、`aggregation_strategy="first"` が必須です。

```python
ner_pipeline = pipeline(
    "token-classification",
    model="biobert-ner-bc5cdr",
    aggregation_strategy="first",  # 重要
    device=0 if torch.cuda.is_available() else -1
)
```

| 設定 | 結果 |
|------|------|
| simple | サブワードが結合されない |
| first | 正しく結合される |

#### 3. データ拡張

BC5CDrデータセットには一般的な薬剤名が含まれていない場合があります。

**insulin検出の改善**:
- 追加前: insulin が検出されない
- 追加後: insulin が正常に検出（5サンプル追加 → 50サンプル追加で解決）

### 実行方法

```bash
cd 01-Biomedical-NER/notebooks
jupyter 01_data_preparation.ipynb
jupyter 02_model_training.ipynb
jupyter 03_evaluation.ipynb
jupyter 04_demo.ipynb
```

---

## 02-Medical-RAG: 医療QAシステム

### 目的

MedQuADデータセットを使用して、医療に関する質問応答システムを構築します。

### 技術スタック

| 項目 | 内容 |
|------|------|
| LLM | GLM-4.7 |
| 埋め込みモデル | MiniLM-L6-v2 |
| ベクトルDB | ChromaDB |
| データセット | MedQuAD（約16,000 Q&A） |

### 性能

- GLM-4評価スコア: 約98/100
- 検索精度: 高い

### デモ出力例

```
Q: What are the side effects of insulin?

A: The side effects of insulin can vary depending on the type of insulin used.
Common side effects include:
- Hypoglycemia (Low Blood Sugar)
- Weight gain
- Injection site reactions
- Fluid retention (Edema)
- Hypokalemia (Low Potassium)
```

### 実行方法

```bash
cd 02-Medical-RAG/notebooks
jupyter 00_medquad_exploration.ipynb
jupyter 01_prepare.ipynb
jupyter 02_build_db.ipynb
jupyter 03_demo.ipynb
```

---

## 03-Integrated-Medical-NLP: 統合パイプライン

### 目的

NERとRAGを統合し、エンティティ抽出を加味した高度な医療QAシステムを構築します。

### 処理フロー

```
質問: "Patient has diabetes and takes insulin. What are the side effects?"

▼ Step 1: NER（エンティティ抽出）
diabetes(疾患), insulin(薬剤) を抽出

▼ Step 2: エンティティを加味して検索
拡張クエリ: "diabetes insulin side effects"

▼ Step 3: LLMで回答生成
 retrieved contextに基づいて回答
```

### 性能

**評価結果**: 80/100

| 項目 | スコア |
|------|--------|
| エンティティ抽出 | 25/25（完璧） |
| 検索クエリ | 25/25（完璧） |
| 回答品質 | 30/50（MedQuADデータセットの限界） |

### セットアップ

統合システムを動作させるには、NERモデルをコピーする必要があります。

```bash
# Windows
xcopy /E /I /Y "..\01-Biomedical-NER\results\biobert-ner-bc5cdr" "03-Integrated-Medical-NLP\models\biobert-ner-bc5cdr"
```

### 実行方法

```bash
cd 03-Integrated-Medical-NLP/notebooks
jupyter 01_ner_demo.ipynb
jupyter 02_rag_demo.ipynb
jupyter 03_integrated.ipynb
```

---

## インストール

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
pip install transformers datasets scikit-learn matplotlib seaborn
pip install langchain langchain-openai langchain-huggingface
pip install sentence-transformers chromadb
pip install python-dotenv jupyter
```

---

## データセット詳細

### bigbio/bc5cdr

| 項目 | 内容 |
|------|------|
| タスク | 疾患・薬剤の固有表現抽出 |
| サイズ | 5,000サンプル |
| ライセンス | CC0（商用利用OK） |
| エンティティ | Chemical（薬剤）, Disease（疾患） |

### MedQuAD

| 項目 | 内容 |
|------|------|
| タスク | 医療Q&A |
| サイズ | 約47,000 Q&Aペア |
| ライセンス | NIH Public Domain |
| ソース | 医療機関・組織のFAQ |

---

## システム要件

| 項目 | 要件 |
|------|------|
| GPU | NVIDIA RTX 3060 (12GB) 以上推奨 |
| CUDA | 12.1 |
| PyTorch | 2.5.1+ |
| API Key | GLM-4.7（RAGコンポーネント用） |

---

## 免責事項

- 本プロジェクトは**教育的・ポートフォリオ目的**です
- **医療診断や治療決定への使用は厳禁**です
- BC5CDrおよびMedQuADは研究用データセットであり、実臨床データとは異なります
- 医療に関する相談は、必ず資格のある医療専門家に行ってください

---

## ライセンス

MIT License - 詳細は [LICENSE](LICENSE) を参照

---

*作成日: 2026-03-29*
*更新日: 2026-03-30*
