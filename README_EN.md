# Medical NLP: Biomedical NER & RAG System

A medical NLP system combining BioBERT-based Named Entity Recognition (NER) with Retrieval-Augmented Generation (RAG).

## Overview

Three integrated components:
1. **Biomedical-NER**: Extract diseases and chemicals from medical text
2. **Medical-RAG**: Medical Q&A using MedQuAD dataset
3. **Integrated-Medical-NLP**: Combined NER + RAG pipeline

## Projects

### 01-Biomedical-NER
- **Model**: BioBERT v1.1
- **Dataset**: bigbio/bc5cdr
- **Performance**: F1 = 0.9733
- **Entities**: Disease (🟠), Chemical (🔵)

### 02-Medical-RAG
- **LLM**: GLM-4.7
- **Dataset**: MedQuAD (~16,000 Q&A)
- **Embedding**: MiniLM-L6-v2
- **Vector DB**: ChromaDB

### 03-Integrated-Medical-NLP
- **Pipeline**: NER + RAG integration
- **Evaluation**: 80/100

## Setup

```bash
pip install torch transformers langchain sentence-transformers chromadb
```

## Usage

See individual project READMEs:
- `01-Biomedical-NER/README.md`
- `02-Medical-RAG/README.md`
- `03-Integrated-Medical-NLP/README.md`

## Requirements

- GPU: NVIDIA RTX 3060 or equivalent
- CUDA: 12.1
- PyTorch: 2.5.1+

## Disclaimer

For educational purposes only. Not for medical diagnosis.

## License

MIT License

---

*Created: 2026-03-29*
*Updated: 2026-03-30*
