# Example: CS/ML Research

## Invocation

```
/auto-academic "Retrieval-augmented generation for low-resource language question answering" --venue "ACL"
```

## Expected Pipeline Trace

### INIT
- Topic: RAG for low-resource language QA
- Venue: ACL (Tier 1, conference format, LaTeX)

### RESEARCH
- deep-research (full mode) produces synthesis on RAG architectures + low-resource NLP
- Hypotheses:
  - H1: RAG with cross-lingual retrieval outperforms monolingual retrieval for low-resource QA (F1 improvement > 5%)
  - H2: Smaller retriever models (< 100M params) achieve 90%+ of large model performance when fine-tuned on target language

### EXPERIMENT
- H1: ML classification pattern — implement RAG pipeline, evaluate on MLQA/TyDi-QA subsets, compare mono vs cross-lingual
- H2: ML classification pattern — train multiple retriever sizes, plot performance curve
- Inner loop: ~6 runs (baseline + 2 per hypothesis + 1 ablation)
- Venue calibration: multiple seeds, ablation studies, SOTA comparison

### WRITE
- academic-paper (full mode) produces conference paper with reproducibility section
- Includes model architecture diagram, performance tables, ablation analysis

### REVIEW
- Round 1: Minor Revision — add error analysis
- Round 2: Accept

### FINALIZE
- LaTeX formatted for ACL submission (acl2024.sty)
