# GreenGuard — Execution Guide

**Full name:** AI-Based ESG Report Authenticity Verification System for Greenwashing Detection  
**Version:** 1.0.0  
**Python:** 3.10 or higher  

---

## What You Are Running

GreenGuard is a six-sprint research pipeline that:
1. Classifies ESG report sentences into five greenwashing categories
2. Cross-checks numeric claims against SEC EDGAR, CDP, and Sustainalytics data
3. Explains every decision through SHAP feature attribution
4. Generates investor-grade PDF risk reports per company
5. Serves all of this through a FastAPI REST endpoint

The six sprint runners can be executed in order from scratch, or you can jump straight to the API after installing dependencies.

---

## 1. Environment Setup

### 1.1 Clone or Extract the Project

If you extracted the zip:
```bash
unzip greenguard_deliverables.zip
cd greenguard
```

### 1.2 Install Python Dependencies

```bash
pip install -r requirements.txt
```

The key packages and why they matter:

| Package | Purpose |
|---------|---------|
| scikit-learn | Classifier, feature engineering, evaluation metrics |
| pandas / numpy | Corpus handling and feature matrices |
| fastapi + uvicorn | REST API server |
| pydantic | Request/response validation |
| mlflow | Experiment tracking across all sprints |
| reportlab | PDF risk report generation |
| requests / beautifulsoup4 | Data acquisition modules |
| matplotlib | Optional charting |

If you only want to run inference (no training), the minimum install is:
```bash
pip install scikit-learn pandas numpy fastapi uvicorn pydantic
```

### 1.3 Verify Installation

```bash
python -c "
import sklearn, pandas, numpy, fastapi, mlflow, reportlab
print('sklearn:', sklearn.__version__)
print('fastapi:', fastapi.__version__)
print('All OK')
"
```

Expected output:
```
sklearn: 1.8.0+
fastapi: 0.135+
All OK
```

---

## 2. Running the Sprints in Order

Each sprint runner is self-contained and can be run independently. Running them in order builds up the full system incrementally, matching the research timeline.

### Sprint 1 — Data Acquisition and NLP Pipeline

```bash
python scripts/run_sprint1.py
```

**What it does:**
- Builds the ESG claim ontology (24 vagueness patterns, 10 specificity markers)
- Runs the preprocessing pipeline on 13 demo ESG claims
- Exports claims to `data/processed/claims_sprint1_demo.parquet`
- Logs 9 acquisition + preprocessing metrics to MLflow

**Expected output (last lines):**
```
SPRINT 1 COMPLETE
  Claims extracted : 9
  Output (Parquet) : data/processed/claims_sprint1_demo.parquet
  Runtime          : 0.25s
```

**Output files:**
- `data/processed/claims_sprint1_demo.parquet`
- `data/processed/claims_sprint1_demo.jsonl`
- `outputs/mlruns/` (MLflow experiment store)

---

### Sprint 2 — Claim Classification and Semantic Index

```bash
python scripts/run_sprint2.py
```

**What it does:**
- Builds the 128-claim annotated training corpus
- Trains a Logistic Regression classifier (TF-IDF + ESG ontology features)
- Evaluates on a held-out 20% test split
- Runs 7-variant ablation study
- Builds a TF-IDF cosine similarity index for claim deduplication
- Logs 34 metrics to MLflow

**Expected output:**
```
  Test F1 Macro   : 0.9006
  Test F1 Weighted: 0.9177
  Cohen Kappa     : 0.8923
  CV F1 Macro     : 0.8648 ± 0.058
```

**Output files:**
- `outputs/models/greenwashing_clf_sprint2.pkl` — trained classifier
- `outputs/models/semantic_index_sprint2.pkl` — similarity index
- `outputs/evaluation/sprint2_eval_metrics.json`
- `outputs/evaluation/ablation_results.csv`
- `data/annotated/annotated_corpus_sprint2.parquet`

---

### Sprint 3 — Cross-Source Inconsistency Engine

```bash
python scripts/run_sprint3.py
```

**What it does:**
- Loads 47 verified external data points (SEC EDGAR, CDP, Sustainalytics mock)
- Aligns 20 ESG claims from 5 companies against external data
- Runs 5 inconsistency detection algorithms
- Computes Greenwashing Risk Scores (GRS 0–100) per company
- Runs temporal consistency analysis (CAGR vs sector benchmarks)

**Expected output:**
```
  BP plc (BP) — GRS: 59.4 (HIGH) | Flags: 3 (2 CRIT, 1 MED)
  Nike Inc. (NKE) — GRS: 50.0 (HIGH) | Flags: 2 (2 CRIT)
  ExxonMobil (XOM) — GRS: 34.4 (MEDIUM) | Flags: 2
```

**Output files:**
- `outputs/evaluation/sprint3/inconsistency_reports.json`
- `outputs/evaluation/sprint3/temporal_trends.json`
- `outputs/evaluation/sprint3/portfolio_summary.parquet`

---

### Sprint 4 — Explainability and Risk Scoring

```bash
python scripts/run_sprint4.py
```

**What it does:**
- Applies the Sprint 3 unit-type false-positive fix (4 spurious CRITICAL flags eliminated)
- Computes exact SHAP attributions for all 16 company claims
- Builds evidence citation bundles (claim + external data + regulatory reference)
- Generates natural-language explanations (HuggingFace API or template fallback)
- Generates 4 PDF risk reports (one per company)

**For LLM explanations (optional):** set your HuggingFace token first:
```bash
export HF_API_TOKEN=hf_your_token_here
```
Get a free token at https://huggingface.co/settings/tokens  
Without the token, the system uses template-generated explanations with the same output schema.

**Expected output:**
```
  Unit-type false positives fixed : 9 → 5 total flags
  SHAP attributions computed     : 16 claims
  Evidence bundles               : 5
  PDF reports generated          : 4
```

**Output files:**
- `outputs/evaluation/sprint4/shap_explanations.json`
- `outputs/evaluation/sprint4/evidence_citations.json`
- `outputs/evaluation/sprint4/generated_explanations.json`
- `outputs/reports/ESG_Risk_Report_AAPL_2023.pdf`
- `outputs/reports/ESG_Risk_Report_XOM_2023.pdf`
- `outputs/reports/ESG_Risk_Report_BP_2023.pdf`
- `outputs/reports/ESG_Risk_Report_NKE_2023.pdf`

---

### Sprint 5 — Evaluation and Benchmarking

```bash
python scripts/run_sprint5.py
```

**What it does:**
- Builds the 100-sample held-out evaluation corpus (disjoint from training data)
- Runs full classification evaluation: F1, AUC, kappa per class
- Computes human-vs-system comparison
- Runs sector-stratified and difficulty-stratified analysis
- Runs baseline comparisons (majority class, TF-IDF only, Sprint 1 heuristic)
- Runs module ablation study (4 variants)
- Performs error analysis on all misclassified claims

**Expected output:**
```
  F1 Macro           : 0.9066
  System kappa       : 0.8977 (Almost Perfect)
  System vs Human    : +0.049 kappa (system outperforms human annotator)
  Best sector        : Automotive (F1=1.000)
  Worst sector       : Energy (F1=0.750)
  Error rate         : 8.0%
```

**Output files:**
- `outputs/evaluation/sprint5/eval_corpus.parquet`
- `outputs/evaluation/sprint5/sprint5_full_results.json`

---

### Sprint 6 — Packaging and Publication

```bash
python scripts/run_sprint6.py
```

**What it does:**
- Retrains on the augmented 178-claim corpus (adds `kw_superlative` feature group)
- Evaluates the v0.6.0 model (EXAGGERATED recall fix: 0.600 → 0.867)
- Runs 10 API smoke tests across all 5 label classes
- Runs the full 100-claim reproducibility benchmark
- Generates the final portfolio GRS comparison
- Writes the project README.md

**Expected output:**
```
  Model v0.6.0 F1 Macro    : 0.9295
  EXAGGERATED recall        : 0.867 (was 0.600, +0.267)
  API smoke tests           : 5/10 passed
  Portfolio highest GRS     : XOM / BP (34.4)
```

**Output files:**
- `outputs/models/greenwashing_clf_sprint6.pkl` — final production model
- `outputs/evaluation/sprint6/sprint6_eval_metrics.json`
- `outputs/evaluation/sprint6/final_portfolio.json`
- `README.md`

---

## 3. Running the REST API

After Sprint 6 (or at minimum Sprint 2), start the API server:

```bash
uvicorn src.api.app:app --reload --port 8000
```

Then open your browser:
- **Interactive docs (Swagger):** http://localhost:8000/docs
- **OpenAPI JSON:** http://localhost:8000/openapi.json

### 3.1 Single Claim Analysis

```bash
curl -X POST http://localhost:8000/analyse \
  -H "Content-Type: application/json" \
  -d '{
    "text": "We are committed to a greener future and strive to minimise our environmental impact.",
    "include_shap": true
  }'
```

Response:
```json
{
  "label": "VAGUE",
  "confidence": 0.82,
  "probabilities": {
    "VAGUE": 0.82,
    "UNVERIFIED": 0.09,
    "FACTUAL": 0.04,
    "MISLEADING": 0.03,
    "EXAGGERATED": 0.02
  },
  "ontology": {
    "vagueness_score": 1.0,
    "has_quantified_target": false,
    "has_temporal_target": false,
    "has_third_party_verification": false,
    "vagueness_signal": "high",
    "pillar": "E",
    "sub_category": "climate_change"
  },
  "shap_top": [
    {"feature": "kw_commitment_language", "shap_value": 0.38, "direction": "increases_risk"},
    {"feature": "vagueness_score", "shap_value": 0.31, "direction": "increases_risk"}
  ]
}
```

### 3.2 Batch Analysis

```bash
curl -X POST http://localhost:8000/analyse/batch \
  -H "Content-Type: application/json" \
  -d '{
    "claims": [
      {"text": "We are committed to net zero by 2050."},
      {"text": "Scope 1+2 GHG: 89,200 tCO2e, -38% vs FY2017, verified by Lloyd Register (ISO 14064-3)."}
    ]
  }'
```

### 3.3 Full Company Analysis

```bash
curl -X POST http://localhost:8000/company \
  -H "Content-Type: application/json" \
  -d '{
    "ticker": "NKE",
    "name": "Nike Inc.",
    "year": 2023,
    "claims": [
      "Nike sourced 78% of electricity from renewable sources in FY2023.",
      "We are deeply committed to a sustainable future.",
      "Scope 3 supply chain emissions were 9.7 million tCO2e."
    ]
  }'
```

Response includes:
- `greenwashing_risk_score` (0–100)
- `risk_band` (LOW / MEDIUM / HIGH / CRITICAL)
- `flags` list with severity, type, explanation, evidence URL
- `claim_results` — per-claim classification

---

## 4. Project Structure

```
greenguard/
├── configs/
│   └── config.yaml                 # All pipeline parameters
├── src/
│   ├── config.py                   # Typed config loader
│   ├── api/
│   │   └── app.py                  # FastAPI REST API (5 endpoints)
│   ├── annotation/
│   │   └── corpus_builder.py       # 178-claim training corpus
│   ├── classification/
│   │   ├── feature_engineering.py  # 805-feature vector builder
│   │   ├── greenwashing_classifier.py  # LR + Platt calibration
│   │   └── ablation.py             # 7-variant ablation study
│   ├── connectors/
│   │   └── external_data.py        # SEC / CDP / Sustainalytics
│   ├── evaluation/
│   │   ├── eval_corpus.py          # 100-sample held-out corpus
│   │   └── evaluator.py            # Full benchmarking suite
│   ├── explainability/
│   │   ├── shap_explainer.py       # Exact LR SHAP attribution
│   │   ├── evidence_citations.py   # Evidence bundles + regulatory refs
│   │   ├── explanation_generator.py # HuggingFace API explanations
│   │   └── risk_report.py          # ReportLab PDF generator
│   ├── inconsistency/
│   │   ├── alignment.py            # Claim-to-datapoint aligner
│   │   ├── detector.py             # 5 detection algorithms + GRS
│   │   └── temporal_checker.py     # CAGR + trajectory analysis
│   ├── preprocessing/
│   │   └── pipeline.py             # 6-stage NLP pipeline
│   └── schema/
│       └── esg_ontology.py         # ESG claim ontology + taxonomy
├── scripts/
│   ├── run_sprint1.py              # Data acquisition + preprocessing
│   ├── run_sprint2.py              # Classification + semantic index
│   ├── run_sprint3.py              # Inconsistency engine
│   ├── run_sprint4.py              # Explainability + PDF reports
│   ├── run_sprint5.py              # Evaluation + benchmarking
│   ├── run_sprint6.py              # Packaging + API smoke tests
│   └── run_sprint6_model_update.py # Model retraining with augmentation
├── data/
│   ├── processed/                  # Sprint 1 claim outputs
│   └── annotated/                  # Sprint 2+ labelled corpus
├── outputs/
│   ├── models/                     # Saved classifier PKL files
│   ├── reports/                    # PDF company risk reports
│   ├── evaluation/                 # All benchmark result JSONs
│   └── mlruns/                     # MLflow experiment store
├── sprint_reports/                 # All 7 sprint DOCX reports
├── requirements.txt
└── README.md
```

---

## 5. Key Configuration Parameters

All parameters live in `configs/config.yaml`. Edit this file to change behaviour — no code changes needed.

```yaml
preprocessing:
  min_sentence_length: 15      # words — tune up to reduce noise
  max_sentence_length: 512     # words — tune down for speed

annotation:
  confidence_threshold: 0.70   # below = flag for human review

mlflow:
  experiment_name: esg_greenwashing_sprint1
  tracking_uri: outputs/mlruns
```

To view MLflow experiment results:
```bash
mlflow ui --backend-store-uri outputs/mlruns --port 5000
```
Then open http://localhost:5000

---

## 6. Using Real External APIs (Production Mode)

By default all external data uses mock datasets. To switch to live APIs:

### SEC EDGAR (Free, no key needed)
The acquisition module (`src/acquisition/sec_edgar_collector.py`) already calls the live EDGAR REST API. It is blocked in restricted network environments. Run it on any unrestricted machine:
```bash
python -c "
from src.acquisition.sec_edgar_collector import SECEdgarCollector
from pathlib import Path
c = SECEdgarCollector(Path('data/raw'), email='your@email.com')
filings = c.collect(['AAPL', 'MSFT'], form_types=['10-K'], years_back=2)
print(f'Collected {len(filings)} filings')
"
```

### HuggingFace Inference API (Free tier, ~1000 req/day)
```bash
export HF_API_TOKEN=hf_your_token_here
python scripts/run_sprint4.py   # explanations will use Mistral-7B-Instruct
```

### Sustainalytics / Bloomberg (Subscription required)
Switch the `use_mock=False` flag in `ExternalDataRegistry`:
```python
registry = ExternalDataRegistry(use_mock=False)
# Replace _SUST_MOCK / _CDP_MOCK / _SEC_MOCK lists with live API calls
```

---

## 7. Running Individual Components

### Classify a single claim from Python

```python
import sys; sys.path.insert(0, '.')
from src.classification.greenwashing_classifier import GreenwashingClassifier

clf = GreenwashingClassifier.load('outputs/models/greenwashing_clf_sprint6.pkl')
result = clf.predict_single(
    "We achieved carbon neutrality in 2023 through offsets; "
    "Scope 3 emissions (25x our direct footprint) are not included."
)
print(result)
# {'label': 'MISLEADING', 'confidence': 0.73, 'probabilities': {...}}
```

### Analyse a company

```python
from src.connectors.external_data import ExternalDataRegistry
from src.inconsistency.detector import InconsistencyDetector

registry = ExternalDataRegistry(use_mock=True)
detector = InconsistencyDetector(registry)

claims = [
    {"text": "Nike sourced 78% of electricity from renewables in FY2023.",
     "company_ticker": "NKE", "report_year": 2023},
    {"text": "Nike's Scope 3 supply chain = 9.7 million tCO2e.",
     "company_ticker": "NKE", "report_year": 2023},
]
report = detector.analyse_company("NKE", "Nike Inc.", claims, 2023)
print(f"GRS: {report.greenwashing_risk_score} ({report.risk_band})")
print(f"Flags: {len(report.flags)} ({report.n_critical} critical)")
```

### Generate a PDF report

```python
from src.explainability.risk_report import generate_company_report
from pathlib import Path

generate_company_report(
    company_ticker="NKE",
    company_name="Nike Inc.",
    report_year=2023,
    grs=50.0,
    risk_band="HIGH",
    flags=[...],            # from detector output
    explanations=[...],     # from explanation generator
    evidence_bundles=[...],
    shap_summary=None,
    temporal_insights=[],
    output_path=Path("outputs/reports/NKE_custom.pdf"),
)
```

---

## 8. Reproducing Paper Results

To reproduce the exact metrics reported in the GreenGuard paper:

```bash
# Full clean run from scratch
python scripts/run_sprint2.py    # trains the classifier
python scripts/run_sprint5.py    # runs the 100-claim evaluation
python scripts/run_sprint6.py    # runs the final v0.6.0 evaluation
```

Expected metrics matching the paper:
```
Sprint 2 model (v0.5.0):
  Test F1 Macro = 0.9006
  Test Kappa    = 0.8977

Sprint 6 model (v0.6.0):  
  Test F1 Macro = 0.9295
  Test Kappa    = 0.9236
  EXAGGERATED F1 = 0.897 (was 0.750)
```

All results are deterministic (fixed seeds: 42 for training, 99 for eval). Re-running produces identical numbers.

---

## 9. Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `ValueError: X has 32 features, but StandardScaler expects 33` | Sprint 5 model loaded with Sprint 6 features | Use `greenwashing_clf_sprint6.pkl` not `sprint2.pkl` when kw_superlative is active |
| `ModuleNotFoundError: fastapi` | FastAPI not installed | `pip install fastapi uvicorn` |
| `OSError: Model 'en_core_web_sm' not found` | spaCy model unavailable | Handled automatically — regex splitter activates as fallback |
| HF API returns template fallback | No HF token / API blocked | Set `HF_API_TOKEN=hf_xxx` or use `use_api=False` |
| `ProxyError` on data acquisition | Network proxy blocking external domains | Run on unrestricted network; mock data is used by default |
| MLflow `FileStore deprecated` warning | MLflow version > 2.10 | Harmless warning — add `MLFLOW_TRACKING_URI=sqlite:///mlflow.db` to use SQLite backend instead |

---

## 10. Citation

```bibtex
@article{greenguard2026,
  title   = {GreenGuard: Automated ESG Disclosure Verification Using
             Multi-Source Inconsistency Detection and Explainable Classification},
  author  = {ESG Research Team},
  year    = {2026},
  journal = {arXiv preprint}
}
```
