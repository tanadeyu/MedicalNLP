# Medical-RAG

MedQuADを使った医療QA RAGシステム。

---

## 目的

MedQuADを使って、医療QAに特化したRAG（検索＋生成）システムを構築する。

---

## 技術スタック

| 技術 | 理由 |
|------|------|
| GLM-4.7 | 医療QAでも安定した生成性能 |
| MiniLM-L6-v2 | 高速で軽量、ローカルRAGに最適 |
| ChromaDB | シンプルで扱いやすいローカル向けベクトルDB |
| LangChain | RAGチェーン構築が容易 |
| MedQuAD | 医療QAデータセット（NIHパブリックドメイン） |

---

## プロジェクト構成

```
Medical-RAG/
├── plan.md
├── README.md
├── requirements.txt
├── src/
│   └── medical_rag.py      # RAGモジュール（Step3統合対応）
├── data/
│   └── medquad_processed.json
├── temp_medquad/           # git cloneでダウンロード（ノートブック内で実行）
├── notebooks/
│   ├── 01_prepare.ipynb  # データ準備（git clone + 確認含む）
│   ├── 02_build_db.ipynb # ChromaDB構築
│   └── 03_demo.ipynb     # デモ + 評価
├── .env                    # GLM-4.7 APIキー
└── results/
    └── chroma_db/
```

---

## 使い方

### 1. データ準備

`notebooks/01_prepare.ipynb` を実行

- ノートブック内でgit clone実行（temp_medquad/に保存）
- クローン確認（サンプルQ&A表示）
- ローカルXMLファイルをJSONに変換
- **所要メモリ**: 約50-100 MB（一時的にメモリに格納）

### 2. ChromaDB構築

`notebooks/02_build_db.ipynb` を実行

- RTX 3060最適化（batch_size=32）
- 約10-15分で完了（約47,000件）

### 3. デモ

`notebooks/03_demo.ipynb` で質問応答

- 基本的な質問応答
- ソース情報表示
- GLM-4による評価（100点満点）
- Step3統合デモ

---

## モジュール利用例

```python
from src.medical_rag import MedicalRAG

# 初期化
rag = MedicalRAG(
    db_path="./results/chroma_db",
    use_gpu=True
)

# 質問応答
answer = rag.answer("What are the symptoms of diabetes?")
print(answer)

# ソース情報付き
result = rag.answer("What are the treatment for hypertension?", return_source=True)
print(result['answer'])
print(result['source_documents'])
```

---

## Step3統合（Integrated-Medical-NLP）

```python
# NERでエンティティ抽出
entities = {"diseases": ["diabetes"], "chemicals": ["insulin"]}

# エンティティを加味して回答
answer = rag.answer_with_entities(
    question="What are the side effects?",
    entities=entities
)
```

---

## 最適化

### GPU最適化（RTX 3060）

```python
HuggingFaceEmbeddings(
    model_name=MODEL_NAME,
    model_kwargs={'device': 'cuda'},
    encode_kwargs={
        'batch_size': 32,  # バッチ処理で高速化
        'normalize_embeddings': True  # 正規化で精度向上
    }
)
```

### プロンプト調整

- 寛容なプロンプト（情報が限られていても回答）
- GLM-4による評価機能（100点満点）

---

## 評価

### GLM-4による評価（100点満点）

| 指標 | スコア |
|------|--------|
| 回答品質 | 約98/100 |
| 検索精度 | 高い |

---

## データセット

### MedQuAD

| 項目 | 内容 |
|------|------|
| **サイズ** | 約47,000件のQ&Aペア |
| **ライセンス** | パブリックドメイン（NIH） |
| **ソース** | 12のNIHウェブサイト |
| **形式** | XML（JSONに変換して使用） |

### データソース一覧

1. Cancer.gov
2. Genetic and Rare Diseases (GARD)
3. Genetics Home Reference (GHR)
4. MedlinePlus Health Topics
5. NIDDK（糖尿病・消化器・腎臓疾患）
6. NINDS（神経疾患）
7. NIH Senior Health
8. NHLBI（心臓・肺・血液）
9. CDC
10. MedlinePlus ADAM
11. MedlinePlus Drugs
12. MedlinePlus Herbs/Supplements

---

## 免責事項

- 本プロジェクトは**教育的・ポートフォリオ目的**です
- **医療診断や治療決定への使用は厳禁**です
- MedQuADデータセットは研究用であり、実臨床データとは異なります
- 医療に関する相談は、必ず資格のある医療専門家に行ってください

## ライセンス

- MedQuAD: パブリックドメイン（NIH）
- 本プロジェクト: MIT License

---

*作成日: 2026-03-29*
*更新日: 2026-03-30（MedQuAD移行）*
