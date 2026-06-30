# DIS_Hughen: Adversarial Machine Learning & IDS Robustness

Assessing the Robustness of ML-Trained Intrusion Detection Systems under Adversarial Perturbations

## Overview

This repository investigates the vulnerability of machine learning-based intrusion detection systems (IDS) to adversarial attacks. Using the UNIDS CIC IDS 2019 dataset, we evaluate how data poisoning and adversarial examples—generated via projected gradient descent (PGD)—degrade the detection performance of trained models.

**Key Research Questions:**
- **RQ1**: How effectively do PGD adversarial examples fool trained IDS models?
- **RQ2**: Can data poisoning attacks degrade IDS training-time performance?
- **RQ3**: Robustness evaluation across attack parameters and feature perturbation strategies
- **RQ4**: Model-specific resilience patterns and defensive considerations

## Repository Structure

```
DIS_Hughen/
├── data/
│   ├── undersampled_CIC2019_dataset.csv      # Preprocessed IDS training data
│   └── adversarial_epsilon_*.csv             # Generated adversarial examples (ε=0.01, 0.05, 0.1, 0.5)
├── DNN Model/                                 # Deep neural network experiments
├── XGBoost Model/                            # Tree-based model experiments
├── Misc/                                      # Utilities and helper scripts
├── EDA_CIC2019.ipynb                         # Exploratory data analysis
├── Generate_Noise.ipynb                      # PGD attack implementation (noise perturbations)
├── Generate_Adversarial.ipynb                # Adversarial example generation (semantic perturbations)
├── RQ1.ipynb                                 # Research Question 1: XGBoost multi-class adversarial robustness
├── RQ1_Binary.ipynb                          # RQ1 variant: binary classification
├── RQ1-DNN.ipynb                             # RQ1 variant: DNN model evaluation
├── RQ2.ipynb                                 # Research Question 2: Data poisoning attacks
├── RQ3.ipynb                                 # Research Question 3: Epsilon sensitivity analysis
├── RQ4.ipynb                                 # Research Question 4: Model comparison & defenses
└── README.md                                 # This file
```

## Key Components

### Datasets

- **`undersampled_CIC2019_dataset.csv`** (Primary)  
  Balanced subset of the UNIDS CIC IDS 2019 dataset with 18 traffic classes (and variants with 3-class and binary reductions for specific experiments).

- **Adversarial Example Files**  
  Pre-generated PGD adversarial examples for multiple epsilon values:
  - `adversarial_epsilon_0.01.csv` – Minimal perturbation
  - `adversarial_epsilon_0.05.csv`
  - `adversarial_epsilon_0.1.csv`
  - `adversarial_epsilon_0.5.csv` – Larger perturbation

### Attack Implementations

#### `Generate_Noise.ipynb`
Implements **PGD (Projected Gradient Descent)** attack with noise-based perturbations:
- Gradient normalization (L2 scaling)
- Configurable epsilon (perturbation budget) and alpha (step size)
- Multiple restart attempts for improved attack success
- Outputs perturbed samples classified as adversarial misclassifications

#### `Generate_Adversarial.ipynb`
Generates adversarial examples with **semantically valid feature perturbations**:
- Respects network traffic feature bounds (e.g., protocol field ranges)
- Class-balanced export (handles imbalanced success rates)
- Supports multi-class and reduced classification tasks
- Fixed random seed for reproducibility

### Model Evaluations

#### Deep Neural Networks
- **`DNN-Multi.ipynb`** – 18-class classification baseline
- **`DNN-Multi-Reduced.ipynb`** – 3-class classification (simplified attack surface)
- **`DNN-Binary.ipynb`** – Binary (attack vs. benign) classification
- **`DNN-Multi-Reduced-PGD-*.ipynb`** – Adversarial robustness evaluation with noise and semantic perturbations

#### XGBoost Models
- **`XGBoost-Base-Multi.ipynb`** – 18-class baseline
- **`XGBoost-Base-Multi-Reduced.ipynb`** – 3-class baseline
- **`XGBoost-Base-Binary.ipynb`** – Binary baseline
- **`XGBoost-Multi-Inference.ipynb`** – 18-class inference on adversarial examples
- **`XGBoost-Multi-Reduced_Inference.ipynb`** – 3-class inference on adversarial examples

### Analysis Notebooks

- **`EDA_CIC2019.ipynb`**  
  Exploratory data analysis: class distribution, feature statistics, data quality, and preprocessing details.

- **`RQ1*.ipynb`** (3 variants)  
  Adversarial example success rates and misclassification patterns across model architectures.

- **`RQ2.ipynb`**  
  Data poisoning: measures impact of adding perturbed samples to training sets on model accuracy.

- **`RQ3.ipynb`**  
  Sensitivity analysis: varies epsilon and other attack parameters; plots robustness curves.

- **`RQ4.ipynb`**  
  Comparative defense and resilience evaluation across XGBoost and DNN variants.

## Workflow

### 1. Data Preparation
```
EDA_CIC2019.ipynb → Understand dataset structure, class balance, features
```

### 2. Attack Generation
```
Generate_Noise.ipynb → PGD noise perturbations
Generate_Adversarial.ipynb → Semantically constrained perturbations
```

### 3. Model Training
```
DNN-Multi.ipynb / XGBoost-Base-Multi.ipynb → Train clean baseline models
```

### 4. Adversarial Evaluation
```
RQ1*.ipynb → Test models on adversarial examples
RQ2.ipynb → Poison training data and retrain
RQ3.ipynb → Vary attack parameters
RQ4.ipynb → Compare defenses
```

## Dependencies

- **Python 3.7+**
- **Data & ML:**
  - `pandas` – data manipulation
  - `numpy` – numerical computing
  - `scikit-learn` – preprocessing, metrics
  - `xgboost` – gradient boosting models
  - `tensorflow` / `keras` – deep neural networks
- **Analysis:**
  - `matplotlib` – plotting
  - `seaborn` – statistical visualization

Install via:
```bash
pip install pandas numpy scikit-learn xgboost tensorflow matplotlib seaborn
```

## Running Experiments

### Quick Start
1. Clone the repository and navigate to the root directory
2. Open desired notebook in Jupyter or JupyterLab:
   ```bash
   jupyter notebook RQ1.ipynb
   ```
3. Run cells in sequence; notebooks handle data loading and model initialization

### For Reproducibility
- All notebooks include random seed initialization (`np.random.seed(...)`)
- Use the provided pre-generated adversarial example CSVs or regenerate via `Generate_Adversarial.ipynb` with identical parameters
- Expected runtime varies by notebook (10 min to 2+ hours for full model training)

## Key Findings (Expected)

- **PGD adversarial examples** successfully degrade XGBoost and DNN model accuracy on held-out test sets
- **Semantic constraints** (preserving feature validity) reduce attack effectiveness but remain potent
- **Data poisoning** effectiveness depends on model architecture; tree-based models show compartmentalization resilience
- **Smaller epsilon budgets** still achieve meaningful misclassification with optimized multi-step attacks

## Citation

If using this repository in research, please cite:
```
Hughen et al. (2024). Adversarial Machine Learning: Assessing the Robustness 
of ML-Trained Intrusion Detection Systems. NU-Academics/DIS_Hughen.
https://github.com/NU-Academics/DIS_Hughen
```

## License

[See LICENSE file in repository, if present]

## Contact & Support

For questions or issues:
- Open an issue on GitHub
- Contact: NU-Academics (Northwestern University)

## Notes for Researchers

- **Data Sensitivity:** The CIC IDS 2019 dataset contains network traffic patterns; use responsibly in external environments
- **Reproducibility:** Generate_Adversarial.ipynb may require tuning for different feature normalization schemes
- **Model Variants:** Different random seeds may yield different poisoning attack success rates; multi-seed evaluation recommended for publications
- **Semantic Validity:** Feature perturbation bounds are hard-coded; adjust in `Generate_Adversarial.ipynb` for datasets with different traffic types
