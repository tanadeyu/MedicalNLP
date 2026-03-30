# Medical-RAG プロジェクト計画

作成日：2026-03-29
更新日：2026-03-29

---

## 目的

MedQuADを使って、医療QAに特化したRAG（検索＋生成）システムを構築する。

---

## 技術スタック（選んだ理由）

| 技術 | 理由 |
|------|------|
| GLM-4.7 | 医療QAでも安定した生成性能 |
| MiniLM-L6-v2 | 高速で軽量、ローカルRAGに最適 |
| ChromaDB | シンプルで扱いやすいローカル向けベクトルDB |
| MedQuAD | 医療QAデータセット（NIHパブリックドメイン） |

---

## プロジェクト構成

```
Medical-RAG/
├── plan.md
├── README.md
├── requirements.txt
├── data/
│   └── medquad_processed.json
├── temp_medquad/           # git cloneでダウンロード（ノートブック内で実行）
├── notebooks/
│   ├── 01_prepare.ipynb  # データ準備（git clone + 確認）
│   ├── 02_build_db.ipynb # ChromaDB構築
│   └── 03_demo.ipynb     # デモ（RAGコード含む）
├── .env                    # GLM-4.7 APIキー
└── results/
    └── chroma_db/
```

---

## 実装ステップ（3ステップ）

### Step1：データ準備

MedQuAD XMLをGitHubから取得し、JSONに変換

```
ノートブック内でgit clone → ローカルXMLパース → Q&A抽出 → JSON保存
```

**メモリ要件**: 約50-100 MB（一時的にメモリに格納）
**保存先**: `temp_medquad/`（約200 MB、git cloneでダウンロード）

### Step2：検索基盤構築

MiniLMで埋め込み生成し、ChromaDBに保存（RTX 3060最適化）

```
ドキュメント → TextSplitterで分割 → MiniLMで埋め込み → ChromaDB保存
```

**最適化設定**:
- `batch_size=32`（GPU並列処理）
- `normalize_embeddings=True`（精度向上）
- 処理時間: 約10-15分（約16,000件）

### Step3：RAG実装

ノートブック内でRAGシステムを実装

```
質問 → ChromaDB検索（ローカル） → GLM-4.7で回答生成（API）
```

---

## デモ例

```
Q: What causes headaches?

A: 頭痛には種類や発生源によって様々な原因があります...

Score: 98/100 (GLM-4評価)
```

---

## 成功基準

- [x] ChromaDB構築完了（約16,000件）
- [x] 検索で関連ドキュメント取得
- [x] LLMで適切な回答生成
- [x] README.md完成
- [x] ノートブック3つ完成
- [x] GPU最適化（batch_size=32）
- [x] GLM-4評価機能

---

## 推定工数

1日（6時間）

---

## ポートフォリオ価値

**RAGの基本技術** + **医療ドメイン特化**

---

## データセット詳細

### MedQuAD

| 項目 | 内容 |
|------|------|
| **サイズ** | 約16,000件のQ&Aペア（処理済み） |
| **ライセンス** | パブリックドメイン（NIH） |
| **ソース** | 12のNIHウェブサイト |
| **形式** | XML（JSONに変換して使用） |

---

*作成日：2026-03-29*
*更新日：2026-03-29（完了）*
*ステータス: 完了*
