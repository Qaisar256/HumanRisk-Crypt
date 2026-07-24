# HumanRisk-Crypt

HumanRisk-Crypt is a reproducibility-oriented framework for privacy-preserving phishing detection and human vulnerability risk analysis. It combines local feature extraction, a BFV-compatible polynomial classifier, real homomorphic inference through TenSEAL and Microsoft SEAL, ATT&CK-aligned ontology reasoning, and expert-calibrated E-HVSS scoring.

This repository distinguishes genuine encrypted experiments from software-only demonstrations. Results produced with the mock backend or synthetic data must not be reported as cryptographic, security, latency, or real-world detection evidence.

## Experimental Modes

### 1. Paper experiment mode

Paper experiment mode is intended for the experiments reported in the manuscript. It requires:

- authorized OpenPhish, PhishTank, or APWG data
- chronological and domain-disjoint data splitting
- a 1,024-dimensional feature representation
- PyTorch training with AdamW
- a real TenSEAL BFV backend
- measured encryption, evaluation, serialization, communication, and decryption costs
- exact encrypted-to-plaintext integer-circuit verification
- expert-provided E-HVSS reference scores when E-HVSS calibration is evaluated

The experiment stops when required datasets, timestamps, cryptographic dependencies, or expert labels are unavailable. It does not silently substitute synthetic data or fixed manuscript values.

### 2. Smoke-test mode

Smoke-test mode validates package integration and data flow. It uses:

- an explicitly non-cryptographic mock backend
- synthetic data generated from a shared latent-risk process
- lightweight local feature representations

Smoke-test output is for software testing only. It cannot validate BFV security, ciphertext noise, homomorphic latency, communication overhead, or phishing-detection performance on real threat data.

## Implemented Components

- local extraction of URL, text, behavioral, persuasion, and metadata features
- 512-dimensional local semantic representation
- 512-dimensional structured URL and metadata representation
- 1,024-dimensional combined feature vector
- PyTorch quadratic polynomial classifier trained with AdamW
- real BFV encrypted inference using TenSEAL and Microsoft SEAL
- polynomial modulus degree of 8192
- plaintext modulus of 65537
- coefficient modulus chain of 218 total bits
- public evaluation context without the secret key on the server side
- measured cryptographic stage latency using `perf_counter`
- measured serialized ciphertext and context sizes
- plaintext-modulus overflow checks
- encrypted and plaintext integer-circuit equality verification
- invariant noise-budget collection when exposed by the installed TenSEAL build
- chronological, domain-disjoint train and test splitting
- public-suffix-aware registered-domain extraction
- RDFLib and SPARQL ATT&CK-aligned evidence lookup
- expert-calibrated E-HVSS scoring
- reproducibility manifests, environment records, and dataset hashes
- release verification scripts and automated tests

## Encrypted Inference Circuit

The BFV server evaluates the following integer quadratic logit:

```text
z = b + sum_i(w_i x_i) + sum_i(q_i x_i^2)
```

The client encrypts the quantized feature vector and sends a serialized ciphertext with the public evaluation context. The server evaluates the polynomial without access to the secret key and returns the encrypted logit. The client decrypts the integer logit and applies the sigmoid function locally.

The repository does not claim that sigmoid, probability calibration, ATT&CK reasoning, or E-HVSS scoring is evaluated homomorphically.

## Measured Cryptographic Evidence

For every encrypted sample, the framework records:

- context generation time
- public-context serialization time
- feature encoding and encryption time
- request serialization time
- server-side request deserialization time
- homomorphic polynomial evaluation time
- response serialization time
- client-side response deserialization time
- decryption time
- request and response ciphertext sizes
- one-time public evaluation-context size
- total and amortized communication overhead
- plaintext and decrypted integer logits
- exact-match verification status
- plaintext-modulus overflow status
- invariant noise budget, when supported

No latency floor, manuscript-aligned runtime, or hard-coded value is inserted into the reported results.

## Data Protocol

The repository does not redistribute OpenPhish, PhishTank, or APWG data. Place authorized CSV exports at:

```text
datasets/openphish/openphish.csv
datasets/phishtank/phishtank.csv
datasets/apwg/apwg.csv
```

Each real-data file must contain at least:

```text
url,label,timestamp
```

Recommended optional fields include:

```text
html_text,registered_domain,brand,sector,domain_age_days,
hosting_stability,urgency_score,authority_score,reward_score,fear_score
```

For expert-calibrated E-HVSS experiments, include:

```text
expert_ehvss
```

The optional `expert_id` field supports expert-level auditing and grouped validation.

The `timestamp` field must represent the collection or observation time of the threat artifact. A supplied `registered_domain` is used directly. Otherwise, the loader derives the registered domain using the bundled public-suffix snapshot without requiring network access.

Paper experiment mode applies chronological splitting and removes domain overlap between training and testing partitions. Random stratified splitting is not used for the reported encrypted experiment.

## Repository Structure

```text
HumanRisk-Crypt_code/
├── configs/
│   ├── default_config.yaml
│   └── demo_config.yaml
├── datasets/
│   ├── openphish/
│   ├── phishtank/
│   ├── apwg/
│   ├── processed/
│   └── synthetic/
├── docs/
│   ├── implementation_notes.md
│   ├── baseline_comparison_protocol.md
│   ├── reviewer3_remediation.md
│   └── local_validation_status.md
├── humanrisk_crypt/
│   ├── crypto/
│   ├── data/
│   ├── ehvss/
│   ├── evaluation/
│   ├── models/
│   ├── ontology/
│   ├── utils/
│   └── pipeline.py
├── scripts/
│   ├── evaluate_models.py
│   ├── prepare_datasets.py
│   ├── run_baseline_comparisons.py
│   ├── run_demo.py
│   └── verify_release.py
├── tests/
│   └── test_pipeline.py
├── results/
├── Dockerfile
├── pyproject.toml
├── requirements.txt
├── requirements-crypto.txt
├── LICENSE
└── README.md
```

## Installation

Python 3.11 is recommended because TenSEAL wheel availability depends on the operating system and Python version.

### Standard environment

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements-crypto.txt
```

On Windows PowerShell:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements-crypto.txt
```

### Docker environment

```bash
docker build -t humanrisk-crypt .
docker run --rm \
  -v "$PWD/datasets:/app/datasets" \
  -v "$PWD/results:/app/results" \
  humanrisk-crypt
```

## Prepare the Datasets

```bash
python scripts/prepare_datasets.py
```

The preparation script validates required columns, normalizes timestamps, derives domains when needed, removes malformed entries, records dataset statistics, and produces processed files for the experiment pipeline.

## Run the Paper Experiment

```bash
python scripts/evaluate_models.py --config configs/default_config.yaml --allow-model-download
```

The script creates a reproducibility bundle under:

```text
results/paper_experiment/
├── predictions.csv
├── crypto_trace.csv
├── metrics.json
├── split_audit.json
├── encrypted_audit.json
├── quantized_polynomial_model.npz
├── training_metadata.json
├── feature_manifest.json
├── ehvss_calibration.json
├── dataset_manifest.json
├── environment.json
└── resolved_config.yaml
```

The split audit must report zero domain overlap and strict temporal ordering. The encrypted audit must identify a cryptographic backend, successful modulus checks, and exact agreement between the decrypted and plaintext integer circuits.

## Verify the Released Results

```bash
python scripts/verify_release.py results/paper_experiment
```

The verifier rejects a result bundle when it detects:

- a mock or non-cryptographic backend
- missing dataset hashes
- temporal leakage
- domain overlap
- encrypted and plaintext integer-logit disagreement
- plaintext-modulus overflow
- missing cryptographic provenance
- inconsistent latency or communication totals

Only result bundles that pass this verification should be used in the manuscript or supplementary material.

## Plaintext Baselines

```bash
python scripts/run_baseline_comparisons.py --config configs/default_config.yaml --allow-model-download
```

The script reports only baselines that are actually implemented. It does not label plaintext estimators as encrypted models and does not assign fixed encrypted latency values.

## Run the Smoke Test

```bash
python scripts/run_demo.py --config configs/demo_config.yaml
```

The console and output files clearly mark this execution as non-cryptographic. The synthetic dataset is intended only to test software interfaces and cannot replace the real phishing datasets.

## Tests

```bash
pytest -q
```

The real BFV integration test is skipped when TenSEAL is unavailable. All remaining unit tests run without external datasets or model downloads.

## Reproducibility Records

Each paper-mode run stores:

- the resolved YAML configuration
- Python, operating-system, and package versions
- dataset paths and SHA-256 hashes
- dataset class and source distributions
- temporal and domain split statistics
- random seeds
- feature dimensions and feature manifest
- model hyperparameters and training metadata
- model quantization parameters
- per-sample cryptographic measurements
- E-HVSS calibration parameters

These files should be archived together with the corresponding manuscript results.

## Explicit Limitations

The current repository does not implement:

- searchable encryption
- attribute-based encryption
- multi-key fully homomorphic aggregation
- encrypted E-HVSS computation
- encrypted CNN, Transformer, random-forest, or federated-learning baselines

These functions must not be presented as experimentally validated components of this release. They may be discussed only as architectural extensions or future work unless separate implementations and reproducible evidence are provided.

## Security Notice

This is research software, not a production phishing gateway. Correct BFV parameter selection, key protection, deployment hardening, side-channel resistance, authentication, secure transport, access control, and operational monitoring remain the responsibility of the deployment team.


