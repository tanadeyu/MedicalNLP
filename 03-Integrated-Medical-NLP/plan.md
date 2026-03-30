# Integrated-Medical-NLP プロジェクト計画

作成日：2026-03-29

---

## 目的

Biomedical-NER（疾患・薬剤抽出）とMedical-RAG（医療QA）を統合したシステムを構築する。

---

## 技術スタック

| 技術 | 用途 |
|------|------|
| BioBERT v1.1 | NER（疾患・薬剤抽出） |
| GLM-4.7 | 質問応答・評価 |
| MiniLM-L6-v2 | 埋め込み |
| ChromaDB | ベクトルDB |

---

## 実装ステップ

### Step1：NERモデル準備

- Biomedical-NERの学習済みモデルを使用
- 疾患・薬剤を抽出

### Step2：RAGシステム準備

- Medical-RAGと同様の構造
- ChromaDB + GLM-4

### Step3：統合パイプライン

```
質問 → NERでエンティティ抽出 → エンティティを加味してRAG検索 → GLM-4で回答
```

---

## プロジェクト構成

```
Integrated-Medical-NLP/
├── plan.md                    ✅
├── README.md                  ✅
├── requirements.txt           ✅
├── notebooks/
│   ├── 01_ner_demo.ipynb     # NER単体デモ（ミニマム）✅
│   ├── 02_rag_demo.ipynb     # RAG単体デモ（ミニマム）✅
│   └── 03_integrated.ipynb   # 統合パイプライン（詳細）✅
├── models/
│   └── biobert-ner-bc5cdr/   # 学習済みNERモデル ✅
└── .env                       ✅
```

## 成功基準

- [x] NERでエンティティ抽出ができる
- [x] RAGで質問応答ができる
- [x] 統合パイプラインが動作する
- [x] README.md完成
- [x] デモノートブック完成

---

## 実装完了日

2026-03-29

---

*作成日：2026-03-29*
*更新日：2026-03-29（完了）*
