# DIS_Hughen: Adversarial Machine Learning & IDS Robustness

Assessing the Robustness of ML-Trained Intrusion Detection Systems under Adversarial Perturbations

## Artificial Intelligence Disclosure

This README was drafted with the assistance of Claude (Anthropic AI) and verified against the repository's notebooks, dataset, and the accompanying dissertation manuscript. Researchers should still confirm technical details against the original notebook implementations before citing or building upon this documentation.

## Overview

This repository investigates the vulnerability of machine learning–based intrusion detection systems (IDS) to adversarial attacks, and the extent to which adversarial training restores robustness. Using the Unified Multimodal NIDS (UM-NIDS) CIC-IDS2019 dataset, the study trains a gradient-boosted (XGBoost) IDS classifier and evaluates it under two threat models: **data poisoning** (training-time) and **evasion** (inference-time). Evasion examples are crafted on a deep-neural-network **surrogate** using projected gradient descent (PGD) under an L∞ budget and **transferred** to the XGBoost target, reflecting a black-box attacker with no access to the target's parameters.

Each research question reports and decides on the metric named in its hypothesis, with 95% confidence intervals from bootstrap resampling.

**Research Questions (as stated in the manuscript):**

- **RQ1 — Training-time poisoning.** Does injecting semantically valid, maliciously altered flow perturbations into the *training* set (a poisoning attack) significantly degrade classifier performance? Decided on precision, recall, and benign false-positive rate.
- **RQ2 — Inference-time evasion.** Does introducing the same class of perturbations at *inference* (an evasion attack) significantly degrade performance? Decided on precision, recall, and benign false-positive rate.
- **RQ3 — Surrogate transfer across composition rates.** Using a surrogate model, do perturbations that combine legitimate and perturbed traffic at varying composition rates reduce the robustness of the XGBoost target? Decided on three-class **macro F1**.
- **RQ4 — Black-box evasion vs. adversarially hardened model.** How do black-box (no-knowledge) evasion attacks compare against a model hardened with adversarial training? Decided on the change in macro F1, with a paired-bootstrap significance test.

## Repository Structure

```
DIS_Hughen/
├── data/
│   ├── undersampled_CIC2019_dataset.csv      # Preprocessed IDS data (BENIGN + 15 DoS/DDoS variants)
│   ├── clean_sample_*.csv / adv_sample_*.csv # Clean and attack working samples
│   ├── adversarial_epsilon_*.csv             # PGD evasion examples (ε = 0.01–320)
│   └── noise_epsilon_*.csv                    # PGD poisoning perturbations (ε = 0.0–5)
├── DNN Model/                                 # PyTorch surrogate / DNN experiments
├── XGBoost Model/                             # Gradient-boosted target-model experiments
├── Misc/                                      # Exploratory & auxiliary notebooks (incl. HopSkipJump)
├── EDA_CIC2019.ipynb                          # Exploratory data analysis
├── Generate_Noise.ipynb                       # PGD poisoning perturbations → noise_epsilon_*.csv
├── Generate_Adversarial.ipynb                # PGD evasion examples (surrogate) → adversarial_epsilon_*.csv
├── RQ1.ipynb                                  # RQ1: training-time poisoning (P / R / FPR)
├── RQ2.ipynb                                  # RQ2: inference-time evasion (P / R / FPR)
├── RQ3.ipynb                                  # RQ3: surrogate transfer across composition rates (macro F1)
├── RQ4.ipynb                                  # RQ4: black-box evasion vs. adversarial training (Δ macro F1)
└── README.md                                  # This file
```

## Key Components

### Dataset

- **`undersampled_CIC2019_dataset.csv`** (primary) — a balanced subset of the UM-NIDS CIC-IDS2019 dataset. The raw label column contains **16 classes** (BENIGN plus 15 DoS/DDoS families such as `DrDoS_DNS`, `DrDoS_LDAP`, `MSSQL`, `NetBIOS`, `UDP`, `WebDDoS`). For modeling these are reduced to a **three-class** scheme — `BENIGN` (0), `DoS_ATTACK` (1), `NON_DoS_ATTACK` (2) — with a binary (attack vs. benign) variant used in specific experiments. Each observation is one bidirectional flow described by 88 CICFlowMeter statistical features.

- **Adversarial / perturbation files**
  - `adversarial_epsilon_*.csv` — PGD **evasion** examples generated on the DNN surrogate across ε = 0.01, 0.05, 0.1, 0.2, 0.3, 0.4, 0.5, 2, 5, 20, 50, 100, 150, 320.
  - `noise_epsilon_*.csv` — PGD **poisoning** perturbations across ε = 0.0, 0.01, 0.05, 0.1, 0.2, 0.3, 0.4, 0.5, 2, 5.

### Attack Implementations

#### `Generate_Adversarial.ipynb` — evasion examples (surrogate, transferred)
PGD attack against the DNN surrogate, then transferred to the XGBoost target:
- **L∞ budget**, sign-gradient steps (`α = ε/7`), 100 steps, 10 random restarts with momentum.
- Rotating loss across restarts (cross-entropy / Carlini–Wagner / DLR) for stronger attacks.
- **Semantic validity enforced:** integer features are rounded and all features are clipped to their observed min/max after inverse-scaling, so perturbed flows remain plausible network traffic.
- Class-balanced export; fixed random seed for reproducibility.

#### `Generate_Noise.ipynb` — poisoning perturbations
PGD-based perturbations (`pgd_noise`, L∞ clamp to ±ε) injected into the training set to simulate data poisoning; outputs `noise_epsilon_*.csv`.

#### `Misc/Test.ipynb` — HopSkipJump (auxiliary)
A decision-based, black-box **L2** HopSkipJump attack (via the Adversarial Robustness Toolbox) applied directly to the XGBoost model, retaining only attack→BENIGN evasions (`adversarial_hsj_xgb.csv`). Exploratory; not part of the four primary RQ analyses.

### Models

- **Surrogate / DNN (`DNN Model/`)** — PyTorch multi-class, reduced (3-class), and binary networks, including PGD robustness variants.
- **Target / XGBoost (`XGBoost Model/`)** — gradient-boosted baselines (multi-class, reduced, binary) and inference notebooks scoring adversarial examples. Primary target configuration: 300 trees, max depth 6, learning rate 0.03, `hist` tree method.

## Analysis Notebooks

- **`EDA_CIC2019.ipynb`** — class distribution, feature statistics, data quality, preprocessing.
- **`RQ1.ipynb`** — poisons the training set at increasing composition rates and measures precision, recall, and benign FPR with bootstrap 95% CIs.
- **`RQ2.ipynb`** — applies evasion perturbations at inference and measures the same metrics.
- **`RQ3.ipynb`** — sweeps legitimate/perturbed composition rates via the surrogate and evaluates transfer to XGBoost on macro F1.
- **`RQ4.ipynb`** — compares black-box evasion against an adversarially hardened model; reports the change in macro F1 with a paired-bootstrap significance test.

## Workflow

```
1. EDA_CIC2019.ipynb                         # understand data, classes, features
2. Generate_Noise.ipynb                      # PGD poisoning perturbations  (→ noise_epsilon_*.csv)
   Generate_Adversarial.ipynb                # PGD evasion examples          (→ adversarial_epsilon_*.csv)
3. XGBoost Model / DNN Model baselines       # train clean target and surrogate
4. RQ1 → RQ2 → RQ3 → RQ4                      # poisoning, evasion, transfer, adversarial-training defense
```

## Dependencies

- **Python 3.10**
- **Data & ML:** `pandas`, `numpy`, `scikit-learn`, `xgboost`, `torch` (PyTorch — surrogate and DNN models)
- **Adversarial attacks:** `adversarial-robustness-toolbox` (ART — HopSkipJump)
- **Statistics & plotting:** `scipy` (bootstrap resampling), `matplotlib`, `seaborn`

Install via:
```bash
pip install pandas numpy scikit-learn xgboost torch adversarial-robustness-toolbox scipy matplotlib seaborn
```

## Running Experiments

1. Clone the repository and navigate to the root directory.
2. Open the desired notebook in Jupyter or JupyterLab:
   ```bash
   jupyter notebook RQ1.ipynb
   ```
3. Run cells in sequence; notebooks handle data loading and model initialization.

**Reproducibility.** Notebooks fix random seeds (`random_state=42`). Use the provided perturbation CSVs or regenerate them via `Generate_Adversarial.ipynb` / `Generate_Noise.ipynb` with identical parameters. Decision statistics use bootstrap resampling (5,000 iterations for RQ2–RQ4). Full model training can take from ~10 minutes to 2+ hours per notebook.

## Key Findings

- **Poisoning (RQ1)** and **evasion (RQ2)** each produce a statistically significant degradation in precision, recall, and benign FPR across the tested perturbation range.
- **Surrogate perturbations transfer (RQ3).** Examples crafted on the DNN surrogate degrade the XGBoost target's macro F1 — the transfer is **strongest at small ε** — so the gradient-boosted target is *not* immune to transferred evasion.
- **Adversarial training is effective (RQ4).** Hardening improves robustness across all evaluated conditions, with no regression at the largest tested budget.
- **Overall:** perturbation magnitude and attack surface jointly mediate vulnerability; small-budget surrogate perturbations transfer, and adversarial training substantially restores robustness.

## Citation

```
Hughen, R. A. (2026). Adversarial Machine Learning: Assessing the Robustness
of ML-Trained Intrusion Detection Systems. Doctoral dissertation, National University.
NU-Academics/DIS_Hughen. https://github.com/NU-Academics/DIS_Hughen
```

## License

Licensed under a **Creative Commons Attribution 4.0 International License** ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)).

**You are free to:** share and adapt the material for any purpose, including commercial, provided you give appropriate credit, link to the license, and indicate changes. **No additional restrictions.** See [https://creativecommons.org/licenses/by/4.0/](https://creativecommons.org/licenses/by/4.0/).

## Notes for Researchers

- **Data sensitivity:** the CIC-IDS2019 data contains network traffic patterns; use responsibly.
- **Semantic validity:** feature bounds and integer constraints are enforced in `Generate_Adversarial.ipynb`; adjust for datasets with different traffic types.
- **Threat model:** evasion examples are crafted on a DNN surrogate and transferred to XGBoost (black-box); the target's gradients are never used.
- **Auxiliary code:** the `Misc/` notebooks (HopSkipJump, targeted-attack, and RQ1 variants) are exploratory and are not part of the four primary RQ results.
