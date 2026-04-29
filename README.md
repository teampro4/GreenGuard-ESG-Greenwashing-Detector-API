
# GreenGuard: ESG Greenwashing Detection System

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![HuggingFace](https://img.shields.io/badge/HuggingFace-Mistral--7B-yellow.svg)](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3)

## Overview
GreenGuard is an explainable AI framework for automated ESG report authenticity verification and greenwashing detection. It classifies ESG claims into five categories (FACTUAL, VAGUE, EXAGGERATED, UNVERIFIED, MISLEADING) with calibrated confidence, cross-source inconsistency detection, and SHAP attributions.

**Sprint 6 model performance (v0.6.0):**
- F1 Macro: 0.9295 | Accuracy: 93.0% | Cohen's kappa: 0.9236
- EXAGGERATED recall: 0.867 (was 0.600 in v0.5.0, +26.7pp)
- Energy sector F1: 0.850 (was 0.750, +10pp)

## Quick Start
```bash
git clone https://github.com/esg-research/greenguard
cd greenguard
pip install -r requirements.txt

# Run analysis
python scripts/run_sprint6.py

# Start API server
uvicorn src.api.app:app --reload --port 8000
# Then open http://localhost:8000/docs
```

## API Usage
```python
import requests

# Single claim analysis
r = requests.post("http://localhost:8000/analyse", json={
    "text": "We are committed to a greener future.",
    "include_shap": True
})
print(r.json())  # {"label": "VAGUE", "confidence": 0.82, ...}

# Full company analysis
r = requests.post("http://localhost:8000/company", json={
    "ticker": "NKE", "name": "Nike Inc.", "year": 2023,
    "claims": ["We are committed to sustainability...", "Scope 3 = 9.7Mt CO2e..."]
})
print(r.json()["greenwashing_risk_score"])  # 50.0
```

## Repository Structure
```
greenguard/
├── src/
│   ├── api/                    # FastAPI REST endpoints
│   ├── annotation/             # Corpus builder + IAA tools
│   ├── classification/         # Feature engineering + classifier
│   ├── connectors/             # SEC EDGAR, CDP, Sustainalytics
│   ├── evaluation/             # Sprint 5 benchmarking suite
│   ├── explainability/         # SHAP, evidence citations, PDF reports
│   ├── inconsistency/          # Cross-source detection engine
│   ├── preprocessing/          # NLP pipeline
│   └── schema/                 # ESG ontology + label taxonomy
├── scripts/                    # Sprint runners
├── configs/config.yaml         # All parameters
├── data/annotated/             # Labelled training corpus
├── outputs/
│   ├── models/                 # Saved classifiers
│   ├── reports/                # PDF risk reports
│   └── evaluation/             # Benchmark results
└── requirements.txt
```

## Citation
```bibtex
@article{greenguard2026,
  title={GreenGuard: An Explainable AI Framework for ESG Greenwashing Detection},
  author={ESG Research Team},
  year={2026},
  journal={arXiv preprint}
}
```
