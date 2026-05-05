# HumanRisk Crypt

HumanRisk Crypt is a privacy preserving phishing detection and human vulnerability scoring framework. It combines local feature extraction, BFV compatible encrypted inference, ATT&CK aligned ontology reasoning, and E HVSS risk scoring for secure social engineering analytics.

The project is designed for reproducible research on encrypted phishing detection, human centric cybersecurity risk modeling, and privacy preserving machine learning.

## Key Features

* Local extraction of URL, email, behavioral, persuasion, and metadata features
* BFV compatible encrypted inference interface
* Mock BFV backend for easy testing without Microsoft SEAL installation
* Polynomial surrogate inference for encrypted model evaluation
* Encrypted Human Vulnerability Scoring System, E HVSS
* ATT&CK aligned technique mapping
* Plaintext and encrypted baseline comparison scripts
* Synthetic dataset generation
* Mean and standard deviation reporting across repeated runs
* Modular Python package structure

## Repository Structure

```text
HumanRisk-Crypt_code/
├── configs/
│   └── default.yaml
├── datasets/
│   ├── openphish/
│   ├── phishtank/
│   ├── apwg/
│   ├── processed/
│   └── synthetic/
├── docs/
│   ├── implementation_notes.md
│   └── baseline_comparison_protocol.md
├── humanrisk_crypt/
│   ├── crypto/
│   ├── data/
│   ├── ehvss/
│   ├── evaluation/
│   ├── models/
│   ├── ontology/
│   └── utils/
├── scripts/
│   ├── run_demo.py
│   ├── prepare_datasets.py
│   └── run_baseline_comparisons.py
├── tests/
├── results/
├── README.md
└── requirements.txt
