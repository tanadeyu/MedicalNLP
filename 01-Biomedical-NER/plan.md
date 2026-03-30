# Biomedical-NER プロジェクト計画

作成日：2026-03-28
更新日：2026-03-28

---

## 📋 プロジェクト概要

### 目的
医療テキストから**疾患・薬剤**の固有表現を抽出するNER（Named Entity Recognition）システムを構築する。

### 技術スタック
| 項目 | 内容 |
|------|------|
| **モデル** | BioBERT系（`dmis-lab/biobert-v1.1`） |
| **タスク** | BERT Token Classification |
| **データ** | BC5CDr（CC0） |
| **評価** | Entity-level F1, Precision, Recall |

---

## 🎯 成果物

### デモンストレーション
- 医療テキストから疾患名・薬剤名を抽出
- **色分けハイライト表示**（疾患：薄いオレンジ、薬剤：薄い青）
- 複数のサンプル例を提示

### 評価指標
- Entity-level Precision/Recall/F1
- エンティティ種類ごとの精度（Disease F1, Chemical F1）
- 混同行列

---

## 📊 データセット: BC5CDr

| 項目 | 内容 |
|------|------|
| **エンティティ** | **疾患（Disease）+ 薬剤（Chemical）** |
| **ライセンス** | CC0（商用利用OK） |
| **商用利用** | ✅ OK |
| **書籍化** | ✅ OK |
| **サイズ** | 約15,000文 |

### ラベル構造（5つ）

| ラベル | 説明 | ハイライト色 |
|--------|------|-------------|
| **O** | 固有表現ではない | - |
| **B-Disease** | 疾患の開始 | 🟠 #ffe6b3（薄いオレンジ） |
| **I-Disease** | 疾患の継続 | 🟠 #ffe6b3（薄いオレンジ） |
| **B-Chemical** | 薬剤の開始 | 🔵 #b3d9ff（薄い青） |
| **I-Chemical** | 薬剤の継続 | 🔵 #b3d9ff（薄い青） |

### データ形式
```
テキスト: "Patient has diabetes and takes insulin"
ラベル: [O, O, B-Disease, O, O, B-Chemical]
```

---

## 📁 プロジェクト構成

```
Biomedical-NER/
├── plan.md                    # このファイル
├── README.md                  # プロジェクト説明
├── 注意事項.md                 # datasetsバージョンについて
├── requirements.txt           # ライブラリ
├── notebooks/
│   ├── 01_data_preparation.ipynb    # データ準備
│   ├── 02_model_training.ipynb      # モデル学習
│   ├── 03_evaluation.ipynb          # 評価
│   └── 04_demo.ipynb                # デモ（色分け表示）
├── results/
│   ├── biobert-ner-bc5cdr/          # 学習済みモデル
│   ├── f1_score_by_label.png
│   ├── confusion_matrix.png
│   └── ner_demo.html
└── data/
    └── processed_bc5cdr/            # 前処理済みデータ
```

---

## 🔧 実装ステップ

### Step1: データ準備
- BC5CDrデータセットをダウンロード（tner/bc5cdr）
- データ形式の確認（5ラベル）
- 前処理（トークン化、ラベル付け）

### Step2: モデル学習
- BioBERTモデルをロード
- Token Classificationヘッドを追加（num_labels=5）
- ファインチューニング

### Step3: 評価
- テストデータで評価
- Entity-level F1スコア算出
- Disease F1, Chemical F1, 全体F1を別々に表示
- 混同行列作成

### Step4: デモ
- サンプルテキストで推論
- **色分けハイライト表示**（疾患：オレンジ、薬剤：青）
- HTML出力で保存

---

## 📝 必要なライブラリ

```txt
torch>=2.0.0
transformers>=4.30.0
datasets
seqeval
scikit-learn
matplotlib
numpy
pandas
tqdm
```

---

## ⏱️ 推定工数

| ステップ | 推定時間 |
|----------|----------|
| データ準備 | 2時間 |
| モデル学習 | 3時間 |
| 評価・分析 | 2時間 |
| デモ作成 | 1時間 |
| **合計** | **1日（8時間）** |

---

## 🎯 成功基準

- [x] Entity-level F1 > 0.85（達成: 0.973）
- [x] Disease F1, Chemical F1 別々に表示
- [x] 混同行列の可視化
- [x] 色分けハイライト表示のデモ
- [x] README.mdの完成
- [x] ノートブック4つの完成

---

## 💡 ポートフォリオ価値

### ⭐⭐⭐⭐⭐ レベルの理由

1. **疾患×薬剤の組み合わせ**
   - 医療現場で実際に使われている（薬物相互作用など）
   - 実務的価値が高い

2. **色分けハイライト**
   - 視覚的に分かりやすい
   - デモとしてインパクトがある

3. **技術バランス**
   - 5ラベルで学習コストが軽い
   - 中級者向け完成度が高い

---

## 📚 参考資料

- BC5CDr: https://huggingface.co/datasets/tner/bc5cdr
- BioBERT: https://github.com/dmis-lab/biobert
- HuggingFace NER: https://huggingface.co/docs/transformers/tasks/token_classification

---

## 🚀 将来の拡張（オプション）

### NER + RAG統合
抽出した疾患・薬剤を使って、関連情報を検索・表示するシステム。

```
入力: "Patient has diabetes and takes insulin"
  ↓
【NER】diabetes(疾患), insulin(薬剤)を抽出
  ↓
【RAG】関連情報を検索（薬物相互作用など）
  ↓
【LLM】回答生成
```

---

*作成日：2026-03-28*
*更新日：2026-03-28（BC5CDr対応）*
*ステータス: 完了（F1: 0.973）*
