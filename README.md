# NepError: A Hierarchical Diagnostic Taxonomy of Failure Modes in Nepali Natural Language Inference

**Pema Tshering Sherpa** 
**Status:** Inference complete · Annotation phase in progress

---

## What is NepError?

Every Nepali NLP paper reports F1 scores. NepError builds the first cross-model failure map for Nepali Natural Language Inference — diagnosing *why* models fail, not just *how often*.

Current evaluations of Nepali NLP, including the NLUE 2025 benchmark produced by KU ILPRL, rely entirely on aggregate accuracy and F1. Models are ranked by score, but no framework exists to categorize the residual failures. A model that hits 80% accuracy has 20% of predictions wrong — and nobody knows why.

NepError proposes a four-tier hierarchical error taxonomy covering eight failure codes, applied across four models and three datasets. The output is a diagnostic map of Nepali NLI failures — actionable evidence for the next generation of Nepali language models.

---

## The Research Gap

1. No systematic error analysis exists for Nepali NLI anywhere in the literature.
2. NLUE 2025 (Nyachhyon, Sharma, Thapa, Bal — IJCNLP Findings 2025, arXiv:2411.19244) produced the first public Nepali NLI datasets. NepError is the first study to analyze the failure modes on these datasets.
3. The NLUE datasets were built by machine-translating English GLUE/XGLUE benchmarks using GPT-4o-mini and Gemini-2.5-flash. This translation pipeline introduces label noise whose extent has not been quantified. NepError quantifies it.

---

## The Four-Tier Taxonomy

Every failure case is assigned exactly one code using a strict eight-step decision checklist applied in priority order.

| Tier | Name | Code | Description |
|------|------|------|-------------|
| T1 | Surface / Input | T1a | **Tokenization Fragmentation** — subword tokenizer splits a Nepali word mid-morpheme, stripping grammatical meaning |
| T1 | Surface / Input | T1b | **Romanization / Script Noise** — Nepali written in Roman script, treated as out-of-vocabulary by Devanagari-trained models |
| T2 | Morpho-Syntactic | T2a | **Negation Particle Drop** — model ignores negation suffix (छैन, होइन, दैन, नाई, etc.) that inverts the NLI label |
| T2 | Morpho-Syntactic | T2b | **Honorific-Verb Agreement Clash** — model fails to track honorific register (हजुर vs तँ) encoded in verb endings |
| T3 | Socio-Pragmatic | T3a | **Code-Switching (Neplish)** — Devanagari-English mixed sentence; consecutive tokens from different linguistic systems |
| T3 | Socio-Pragmatic | T3b | **Domain Vocabulary Gap** — rare term from legal, agricultural, forestry, or medical domain |
| T4 | Ambiguous / Noise | T4a | **Genuine Semantic Ambiguity** — two native speakers would independently assign different labels |
| T4 | Ambiguous / Noise | T4b | **Translation Artifact** — machine translation produced unnatural or semantically broken Nepali |

---

## Models

| Model | HuggingFace ID | Type | Role |
|-------|---------------|------|------|
| XLM-R Large | `joeddav/xlm-roberta-large-xnli` | Multilingual encoder | Global baseline |
| mDeBERTa-v3 | `MoritzLaurer/mDeBERTa-v3-base-xnli-multilingual-nli-2mil7` | Multilingual encoder | Strong baseline |
| CHiPSAL-BERT | `IRIISNEPAL/RoBERTa_Nepali_110M` | Nepali-specific encoder | Fine-tuned on MNLI-Nepali |
| Llama 3.1 8B | via Groq API (`llama-3.1-8b-instant`) | Instruction-tuned LLM | Zero-shot frontier reference |

XLM-R and mDeBERTa are used zero-shot with their existing NLI heads. CHiPSAL-BERT was fine-tuned by this project on the MNLI-Nepali training set (3 epochs, 40,800 rows). Llama 3.1 8B was prompted zero-shot via the Groq API on a 999-row subset.

---

## Datasets

All datasets are publicly available at: [https://huggingface.co/collections/IRIIS-RESEARCH/nepali-lanuguage-understanding-evaluation-benchmark](https://huggingface.co/collections/IRIIS-RESEARCH/nepali-lanuguage-understanding-evaluation-benchmark)

| Dataset | Test Rows | Task | Label Encoding | Scope |
|---------|-----------|------|----------------|-------|
| MNLI-Nepali | 10,200 | 3-class NLI | 0=entailment, 1=neutral, 2=contradiction | Primary analysis |
| RTE-Nepali | 503 | Binary NLI | 0=entailment, 1=not_entailment | Secondary analysis |
| XNLI-Nepali | 12,794 | 3-class NLI | 0=entailment, 1=neutral, 2=contradiction | Generalizability |

QNLI-Nepali is out of scope (QA reasoning ≠ NLI reasoning). CoRef-Nepali is future work.

---

## Inference Results

### MNLI-Nepali (10,200 test rows, except Llama)

| Model | Accuracy | Total Failures | Sampled for Annotation |
|-------|----------|---------------|----------------------|
| XLM-R Large | 80.83% | 1,955 | 600 (200/label) |
| mDeBERTa-v3 | 80.77% | 1,961 | 600 (200/label) |
| CHiPSAL-BERT | ~74% | ~2,600 | 600 (200/label) |
| Llama 3.1 8B | 88.59% | 114 | 114 (all, 999-row subset) |

### RTE-Nepali (503 test rows)

| Model | Accuracy | Total Failures | Sampled for Annotation |
|-------|----------|---------------|----------------------|
| XLM-R Large | 64.41% | 179 | 62 |
| mDeBERTa-v3 | 60.24% | 200 | 104 |
| CHiPSAL-BERT | Pending | — | — |
| Llama 3.1 8B | Pending | — | — |

### XNLI-Nepali

Inference pending. Code identical to MNLI pipeline.

---

## Key Findings

### Finding 1 — CHiPSAL-BERT underperforms multilingual models despite Nepali-specific pretraining

XLM-R and mDeBERTa both achieve 80.8% on MNLI-Nepali with zero Nepali-specific training. CHiPSAL-BERT, trained on 27.5GB of Nepali newspaper text, reaches approximately 74% after fine-tuning on the same task. The six to seven point gap is attributed to domain mismatch: CHiPSAL's pretraining corpus is Nepali news text, while MNLI-Nepali sentences are machine-translated from English and structurally unlike natural Nepali. The Nepali-specific pretraining advantage is erased by translation artifacts in the target domain.

Hypothesis: CHiPSAL-BERT will rank higher than multilingual models on the 150 natively authored Nepali pairs (collection pending), where the domain gap disappears.

### Finding 2 — Each model has a distinct dominant failure signature

On the same test set, four models produce four qualitatively different failure patterns:

| True → Predicted | XLM-R | mDeBERTa | CHiPSAL | Llama |
|-----------------|-------|----------|---------|-------|
| ENT → NEU | 135 | 152 | 120 | 31 |
| ENT → CON | 65 | 48 | 80 | 7 |
| NEU → ENT | 145 | 137 | 85 | **36** |
| NEU → CON | 55 | 63 | 115 | 2 |
| CON → ENT | **118** | 68 | 58 | 22 |
| CON → NEU | 82 | **132** | **142** | 16 |

- XLM-R: dominant failure is CON→ENT (118 cases) — conflates contradiction with entailment
- mDeBERTa: dominant failure is CON→NEU (132 cases) — retreats to neutral on contradictions
- CHiPSAL: dominant failure is CON→NEU (142 cases) — same retreat pattern, stronger signal
- Llama: dominant failure is NEU→ENT (36 cases) — zero-shot LLM overclaims entailment on neutral pairs

### Finding 3 — Negation is a universal failure signal (~23–25% across all models)

Negation markers (छैन, छैनन्, दैन, दैनन्, होइन, होइनन्, नाई, नगर्, नहुने, नभएको, नगरेको) appear in approximately 23–25% of failures across all four models including Llama. This consistency across architectures suggests dataset-level difficulty rather than model-specific weakness.

**Important methodological note:** The corpus base rate of negation across all 10,200 MNLI-Nepali test rows is 25.2%. Negation appears in 25.2% of the training corpus AND in 24–25% of failures. Enrichment factor = ~1.0. This means negation is NOT more common in failures than in the overall dataset. The initial claim that "models specifically fail on negation" is not supported by corpus frequency alone. 
 The question changed from "do models fail more often on negation sentences?" to "when a model fails on a negation sentence, is the negation the cause?" That's an annotation question, not a statistics question.

### Finding 4 — 69% ambiguous by dataset cartography (unusually high)

Dataset cartography (Swayamdipta et al., EMNLP 2020) was applied during CHiPSAL-BERT fine-tuning to classify training instances by confidence trajectory across epochs.

| Region | Count | Percentage |
|--------|-------|------------|
| Easy (mean confidence ≥ 0.5, variability < 0.1) | 8,113 | 22.1% |
| Ambiguous (variability ≥ 0.1) | 25,401 | 69.2% |
| Hard (mean confidence < 0.5, variability < 0.1) | 3,206 | 8.7% |

English NLI datasets (MultiNLI, SNLI) typically show 40–50% ambiguous under cartography. The 69.2% figure suggests significant label noise from the GPT-4o-mini / Gemini translation pipeline.

**Open question:** High variability could reflect translation noise (T4b) or genuine linguistic difficulty in Nepali NLI. A native speaker check on a sample of ambiguous-region instances is planned to separate these two explanations.

### Finding 5 — Llama is the best zero-shot performer but avoids predicting contradiction

Llama 3.1 8B achieves 88.6% accuracy with no NLI training. However, across its 114 failures, it predicted "contradiction" only 9 times. Instruction-tuned LLMs trained for helpfulness resist asserting contradiction in zero-shot settings — contradiction requires certainty they won't express without explicit supervision. This is a known zero-shot LLM behavior, not a Nepali-specific issue.

### Finding 6 — 16–20pp accuracy drop on RTE vs MNLI

XLM-R: 80.8% → 64.4%. mDeBERTa: 80.8% → 60.2%. RTE sentences are longer-form news and academic text with higher domain-specific vocabulary density. T3b (domain vocabulary gap) is expected to dominate the RTE failure distribution, unlike MNLI where T2a (negation) is the primary hypothesis.

### Finding 7 — Cross-model failure overlap

| Overlap | Instance Count |
|---------|---------------|
| XLM-R ∩ mDeBERTa | 106 |
| XLM-R ∩ CHiPSAL | 93 |
| mDeBERTa ∩ CHiPSAL | 76 |
| All three encoder models | **18** |

The 18 instances all three encoders fail on are the highest-priority annotation targets. They are likely T4a (genuine ambiguity) or severe compounded T2 errors. Annotated first.

---

## Annotation Plan

**Total target: ~1,080 rows across three datasets, split 50/50 between two annotators**

| File | Rows to Annotate | Priority |
|------|-----------------|----------|
| failures_xlmr_mnli.csv | 200 | HIGH — pilot set drawn from here |
| failures_mdeberta_mnli.csv | 150 | HIGH |
| failures_chipsalbert_mnli.csv | 200 | HIGH |
| failures_llama_mnli.csv | 114 | HIGH (all failures) |
| failures_xlmr_rte.csv | 62 | MEDIUM |
| failures_mdeberta_rte.csv | 104 | MEDIUM |
| XNLI failures (not yet extracted) | ~150 | MEDIUM |

**Within each file, annotate in this order:**
1. All 18 instances in the three-encoder overlap — highest diagnostic value
2. All rows where `flag_negation = 1` — T2a candidates
3. Random sample from remainder to reach target count

**Inter-annotator agreement protocol:**
- Pilot: both annotators independently annotate the same 50 rows from `failures_xlmr_mnli.csv`
- No communication during pilot
- Cohen's Kappa computed via `sklearn.metrics.cohen_kappa_score`
- Target: κ ≥ 0.70 (substantial agreement, Landis & Koch 1977)
- If κ < 0.70: calibration session, re-pilot on fresh 50 rows

---

## Repository Structure

```
neperror/
│
├── notebooks/
│   ├── 01_inference_xlmr_mdeberta.ipynb      # XLM-R and mDeBERTa inference + failure extraction
│   ├── 02_chipsal_bert_finetune.ipynb         # CHiPSAL-BERT fine-tuning + inference + cartography
│   ├── 03_llama_inference.ipynb               # Llama 3.1 8B zero-shot via Groq API
│   └── 04_annotation_analysis.ipynb           # Cohen's Kappa, taxonomy distribution, paper tables
│
├── data/
│   ├── failures/
│   │   ├── failures_xlmr_mnli.csv             # 600 rows, annotation-ready
│   │   ├── failures_xlmr_rte.csv              # 62 rows, annotation-ready
│   │   ├── failures_mdeberta_mnli.csv         # 600 rows, annotation-ready
│   │   ├── failures_mdeberta_rte.csv          # 104 rows, annotation-ready
│   │   ├── failures_chipsalbert_mnli.csv      # 600 rows (no conf columns — known issue)
│   │   └── failures_llama_mnli.csv            # 114 rows (includes raw_response column)
│   ├── cartography_dynamics.csv               # 36,720 rows, CHiPSAL training dynamics
│   ├── inference_summary.csv                  # Accuracy and failure counts per model/dataset
│   └── cross_model_comparison.csv             # Cross-model overlap and summary table
│
├── annotation/
│   └── neperror_annotation_guidelines.pdf     # Full annotator training manual (XeLaTeX compiled)
│
└── README.md
```

---

## Failure File Schema

All failure CSV files share this column schema. Annotators fill in the last three columns.

| Column | Description |
|--------|-------------|
| `instance_id` | Unique row identifier |
| `premise` | Premise sentence in Nepali |
| `hypothesis` | Hypothesis sentence in Nepali |
| `true_label` | Gold label (0=entailment, 1=neutral, 2=contradiction) |
| `pred_label` | Model's incorrect prediction |
| `correct` | Always False in failure files |
| `model` | Model that made this prediction |
| `dataset` | Source dataset |
| `conf_label0` | Model confidence for label 0 (XLM-R and mDeBERTa only) |
| `conf_label1` | Model confidence for label 1 (XLM-R and mDeBERTa only) |
| `conf_label2` | Model confidence for label 2 (XLM-R and mDeBERTa only) |
| `flag_romanization` | 1 if Roman script detected in text |
| `flag_codeswitching` | 1 if code-switching detected |
| `flag_negation` | 1 if negation marker detected in text |
| `taxonomy_tier` | **Annotator fills:** one of T1a, T1b, T2a, T2b, T3a, T3b, T4a, T4b |
| `annotator_note` | **Annotator fills:** 1–2 sentence justification |
| `annotator_id` | **Annotator fills:** assigned annotator code |

---

## Pre-Annotation Computational Tasks

The following must be completed before main annotation begins:

**1. Negation base rate on full corpus**

Run the negation regex across all 10,200 MNLI-Nepali test rows (not just failures) and compute the corpus base rate. Compare against the ~23–25% failure rate. The regex must cover Devanagari suffixal forms (-एन, -छैन, -दैन), Devanagari prefixal forms (न-), and Romanized equivalents (chhaina, hoina, daina). If the base rate is close to 25%, the enrichment claim in Finding 3 does not hold.

**2. XNLI-Nepali inference**

Run XLM-R and mDeBERTa on the full XNLI-Nepali test set using the same pipeline as Notebook 01. CHiPSAL-BERT and Llama to follow. Extract failure files using the same schema.

**3. CHiPSAL-BERT and Llama on RTE-Nepali**

Add to Notebooks 02 and 03 respectively. Required before the RTE failure files can go to annotation.

---

## Pending Work

- [ ] XNLI-Nepali inference (all models)
- [ ] CHiPSAL-BERT and Llama inference on RTE-Nepali
- [ ] 150 natively authored Nepali NLI pairs (3 native speakers, 4 domains: family hierarchy, rural agriculture, political discourse, urban code-switching) — required to populate T1b and T3a tiers
- [ ] Minimal pairs: 30 per T2 error category for causal attribution
- [ ] Human baseline: 3 native speakers annotate the 150 native pairs
- [ ] Main annotation (~1,080 rows)
- [ ] Cohen's Kappa computation and calibration (Notebook 04)
- [ ] Paper writing

---

## Related Work

| Paper | Venue | Relevance |
|-------|-------|-----------|
| Nyachhyon et al. — NLUE | IJCNLP Findings 2025, arXiv:2411.19244 | Source datasets and CHiPSAL-BERT context |
| Thapa et al. — CHiPSAL-BERT | arXiv:2411.15734 | Base model for fine-tuning |
| Swayamditta et al. — Dataset Cartography | EMNLP 2020 | Cartography methodology |
| McCoy, Pavlick, Linzen — HANS | ACL 2019 | NLI shortcut heuristics, context for failure analysis |
| Gururangan et al. — Annotation Artifacts | NAACL 2018 | Theoretical grounding for T4b |
| Conneau et al. — XLM-R | ACL 2020 | Baseline model |
| He et al. — DeBERTa | ICLR 2021 | Baseline model |
| Williams, Nangia, Bowman — MultiNLI | NAACL 2018 | Source of MNLI-Nepali |
| Bowman et al. — SNLI | EMNLP 2015 | NLI task foundation |

---

## Citation

If you use this taxonomy, annotation guidelines, or failure datasets, please cite:

```
@misc{sherpa2026neperror,
  title     = {NepError: A Hierarchical Diagnostic Taxonomy of Failure Modes in Nepali Natural Language Inference},
  author    = {Sherpa, Pema Tshering},
  year      = {2026},
  note      = {}
}
```

---

## Contact

**Pema Tshering Sherpa** — ptssherpa5@gmail.com
B.Tech AI & Data Science (Minor: Fintech), RV University Bengaluru

---

*This project aims to produce a free, open-source diagnostic resource for the Nepali NLP community. Developed in collaboration with the KU ILPRL lab, the architects of CHiPSAL-BERT and NLUE.*
