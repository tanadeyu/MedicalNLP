# Biomedical-NER

医療テキストから**疾患・薬剤**の固有表現を抽出するNER（Named Entity Recognition）システム。

## 概要

BioBERT v1.1をBC5CDrデータセットでファインチューニングした固有表現抽出モデル。医療テキストから疾患名と薬剤名を自動抽出します。

## 特徴

- ✅ **疾患（Disease）**と**薬剤（Chemical）**の2種類を抽出
- ✅ 色分けハイライト表示（疾患：オレンジ🟠、薬剤：青🔵）
- ✅ BC5CDrデータセット（bigbio/bc5cdr、CC0ライセンス）
- ✅ Entity-level F1: **0.9733**

## 技術スタック

| 項目 | 内容 |
|------|------|
| **モデル** | BioBERT v1.1 (dmis-lab/biobert-v1.1) |
| **タスク** | BERT Token Classification |
| **データセット** | bigbio/bc5cdr (正しいラベル順序) |
| **フレームワーク** | PyTorch 2.5.1 + Transformers |
| **トークナイザー** | WordPiece (aggregation_strategy="first") |

## エンティティと色分け

| エンティティ | ラベル | ハイライト色 |
|------------|------|-------------|
| 疾患 | B-Disease, I-Disease | 🟠 #ffe6b3（薄いオレンジ） |
| 薬剤 | B-Chemical, I-Chemical | 🔵 #b3d9ff（薄い青） |
| その他 | O | - |

## ラベル定義

```
label_list = ['O', 'B-Chemical', 'I-Chemical', 'B-Disease', 'I-Disease']
```

- **Chemical（薬剤）**: insulin, aspirin, metformin, ibuprofen など
- **Disease（疾患）**: diabetes mellitus, hypertension, arthritis, pneumonia など

## パフォーマンス

### 最終スコア

| 指標 | スコア |
|------|--------|
| **全体F1 (micro)** | **0.9733** |
| 全体F1 (macro) | 0.97xx |

### エンティティ別

| エンティティ | F1スコア |
|------------|----------|
| Disease 🟠 | 約0.97 |
| Chemical 🔵 | 約0.98 |

*詳細は `results/evaluation_results.json` を参照*

## データ拡張

BC5CDrデータセットには一般的な薬剤名 "insulin" が含まれていなかったため、50サンプルの追加学習を実施。

- **追加前**: insulin が検出されなかった
- **追加後**: insulin が正常に検出されるようになった（スコア: 0.6995）

### 追加データ内容

- 50サンプルの insulin を含む医療テキスト
- 様々な文脈をカバー（治療、投与、患者ケア、症例など）
- トレーニングデータ: 1000 → 1050 サンプル

## 実行手順

ノートブックを順番に実行します：

1. **01_data_preparation.ipynb** - BC5CDrデータセットのダウンロードと準備
2. **02_model_training.ipynb** - BioBERTのファインチューニング（insulinサンプル含む）
3. **03_evaluation.ipynb** - モデルの評価と混同行列作成
4. **04_demo.ipynb** - デモ（色分けハイライト表示）

## 使用方法

### Pythonコードで使用

```python
from transformers import pipeline

# パイプライン作成
ner_pipeline = pipeline(
    "token-classification",
    model="./results/biobert-ner-bc5cdr",
    aggregation_strategy="first"  # WordPieceサブワード結合に必須
)

# 推論
text = "Patient has diabetes mellitus and takes insulin for treatment."
entities = ner_pipeline(text)

for ent in entities:
    icon = '🟠' if 'Disease' in ent['entity_group'] else '🔵'
    print(f"{ent['entity_group']}: {ent['word']} {icon} (スコア: {ent['score']:.4f})")
```

### 出力例

```
Disease: diabetes mellitus 🟠 (スコア: 0.8559)
Chemical: insulin 🔵 (スコア: 0.6995)
```

## プロジェクト構成

```
Biomedical-NER/
├── README.md
├── plan.md
├── 注意事項.md
├── requirements.txt
├── notebooks/
│   ├── 01_data_preparation.ipynb    # データ準備（bigbio/bc5cdr使用）
│   ├── 02_model_training.ipynb      # モデル学習（insulinサンプル追加）
│   ├── 03_evaluation.ipynb          # 評価と可視化
│   ├── 04_demo.ipynb                # デモ（色分け表示）
│   └── processed_bc5cdr/            # 前処理済みデータ
└── results/
    ├── biobert-ner-bc5cdr/          # 学習済みモデル
    ├── confusion_matrix.png         # 混同行列
    ├── f1_score_by_label.png        # ラベル別F1スコア
    └── evaluation_results.json      # 評価結果
```

## デモンストレーション

### サンプルテキスト

```
"Patient has diabetes mellitus and takes insulin for treatment."
```

### 抽出結果

| エンティティ | テキスト | スコア |
|------------|----------|--------|
| Disease 🟠 | diabetes mellitus | 0.8559 |
| Chemical 🔵 | insulin | 0.6995 |

## 重要な技術的注意点

### 1. データセット選択

- ✅ **使用**: bigbio/bc5cdr（正しいラベル順序）
- ❌ **非使用**: tner/bc5cdr（ラベルが逆になっているバグあり）

詳細は `注意事項.md` を参照。

### 2. サブワード結合

BioBERTはWordPieceトークナイザーを使用するため、単語がサブワードに分割されます。

```python
# 例: aspirin → ["as", "##pirin"]
```

正しく結合するため、`aggregation_strategy="first"` を使用します。

- ❌ `aggregation_strategy="simple"` - サブワードが結合されない
- ✅ `aggregation_strategy="first"` - 正しく結合される

### 3. データ拡張

BC5CDrデータセットの限界により、一般的な薬剤名が検出されない場合があります。その場合は、対象の単語を含むサンプルを30〜50個追加して再学習することで対応可能です。

## インストール

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
pip install transformers datasets scikit-learn matplotlib seaborn jupyter
```

## 免責事項

- 本プロジェクトは**教育的・ポートフォリオ目的**です
- **医療診断や治療決定への使用は厳禁**です
- BC5CDrデータセットは研究用であり、実臨床データとは異なります
- 医療に関する相談は、必ず資格のある医療専門家に行ってください

## ライセンス

本プロジェクトはBC5CDrデータセット（CC0ライセンス）を使用しています。

---

*作成日: 2026-03-28*
*更新日: 2026-03-30（bigbio/bc5cdr対応、insulin検出改善）*
