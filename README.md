<div align="center">

# Dual-Agent ER-TRPO Framework for DDoS Attack Detection

### An Intelligent Dual-Agent Reinforcement Learning Framework for DDoS Attack Detection

**Active Learning · ER-TRPO · Multi-Step Returns · LIME · Imbalance-Aware Learning · Homotopy-BOHB**

</div>

---

## Overview

This repository provides the implementation of the methodology introduced in the manuscript:

> **An Intelligent Dual-Agent Reinforcement Learning Framework for DDoS Attack Detection**

The proposed framework integrates a coordinated **dual-agent reinforcement learning architecture** with an enhanced **Trust Region Policy Optimization (TRPO)** strategy for adaptive DDoS attack detection.

The framework is designed to address several important limitations of existing DDoS detection systems, including:

- inefficient use of labeled training samples,
- limited adaptive feature selection,
- insufficient model interpretability,
- class imbalance,
- unstable policy learning,
- and sensitivity to hyperparameter configurations.

The proposed architecture contains two interacting reinforcement-learning agents:

- **Agent 1:** adaptive sample acquisition using Active Learning (AL),
- **Agent 2:** sequential feature selection and DDoS attack classification.

Both agents are optimized using an enhanced TRPO formulation incorporating:

- **Entropy Regularization (ER)** for improved exploration,
- **multi-step return estimation** for more stable policy learning,
- and an explicit **KL-divergence trust-region constraint**.

The complete framework also integrates **LIME-guided feature analysis**, **class-frequency-aware reward adaptation**, and **Homotopy-guided BOHB hyperparameter optimization**.

---

## Key Features

The implementation includes the complete experimental pipeline described in the manuscript.

### Dual-Agent Reinforcement Learning

The proposed architecture separates sample acquisition and attack detection into two coordinated reinforcement-learning agents.

#### Agent 1 – Active Learning Agent

Agent 1 determines whether an unlabeled network-traffic instance should be requested for annotation.

Its state combines:

- the LSTM-derived representation of the sample,
- and the corresponding prediction probability produced by Agent 2.

Agent 1 estimates uncertainty using class-centroid affinities calculated exclusively from currently labeled training samples.

The acquisition decision accounts for:

- uncertainty,
- annotation cost,
- and the resulting change in validation balanced accuracy.

#### Agent 2 – Feature Selection and Detection Agent

Agent 2 performs:

1. LSTM-based representation learning,
2. MLP-based feature relevance scoring,
3. sequential feature selection,
4. LIME-guided feature evaluation,
5. and binary DDoS classification.

Feature-selection decisions consider:

- normalized LIME feature attribution,
- feature-subset complexity,
- and validation-performance improvement.

The classification reward is weighted according to the class frequencies in the current labeled training set to reduce bias toward the majority class.

---

## ER-TRPO Policy Optimization

Both reinforcement-learning agents are optimized using an enhanced version of TRPO.

The implementation includes:

- policy networks,
- value networks,
- multi-step return estimation,
- advantage estimation,
- entropy regularization,
- importance-sampling policy ratios,
- KL-divergence constraints,
- conjugate-gradient optimization,
- Fisher-vector products,
- and backtracking line search.

Entropy regularization encourages exploration, while the KL-divergence constraint limits excessive changes between consecutive policies.

This combination provides a controlled balance between exploration and stable policy improvement.

---

## Multi-Step Return Estimation

Instead of relying exclusively on one-step return estimation, the framework employs **n-step returns**.

The resulting multi-step advantage estimates provide information from multiple successive interactions and are used directly during ER-TRPO optimization.

This mechanism is intended to improve policy-learning stability for both agents.

---

## LIME-Guided Feature Analysis

The detection agent integrates **Local Interpretable Model-Agnostic Explanations (LIME)** into its feature-selection mechanism.

For each selected feature, the absolute LIME attribution is normalized relative to the total attribution magnitude of all candidate features.

The feature-selection reward considers:

- local feature relevance,
- feature-subset complexity,
- and improvement in validation balanced accuracy.

Therefore, LIME is incorporated directly into the reinforcement-learning process rather than being used only for post-hoc visualization.

---

## Imbalance-Aware Classification Reward

DDoS datasets frequently contain substantial class imbalance.

The framework dynamically calculates class-dependent reward weights from the labeled training data.

Less frequent classes receive larger reward magnitudes, while more frequent classes receive smaller reward magnitudes.

A correct classification therefore produces a positive class-dependent reward, whereas an incorrect prediction produces a negative reward with the corresponding magnitude.

This mechanism allows class imbalance to influence policy learning without relying on a manually fixed minority-class reward multiplier.

---

## Active Learning and Adaptive Annotation

The Active Learning agent evaluates unlabeled samples using representations generated by the current detection model.

Class centroids are calculated only from labeled training samples.

For an unlabeled representation, distances to all labeled-data class centroids are converted into a normalized affinity distribution.

The entropy of this distribution is then used as an uncertainty measure.

The annotation threshold changes dynamically during training using exponential decay.

Consequently, the sample-selection criterion evolves as the detector becomes more stable.

---

## Homotopy-Guided BOHB Optimization

The framework integrates **Homotopy continuation** with **Bayesian Optimization and Hyperband (BOHB)** for hyperparameter optimization.

Rather than immediately optimizing the complete ER-TRPO objective, optimization progresses through multiple Homotopy stages.

At the initial stage, policy optimization emphasizes the importance-weighted advantage term.

The entropy contribution is then progressively introduced until the complete ER-TRPO objective is reached.

At each Homotopy stage, BOHB performs resource-aware black-box hyperparameter optimization.

High-performing configurations from one stage are transferred to the following stage to warm-start the model-based search.

---

## Framework Workflow

The overall training procedure follows the coordinated process below:

```text
Network Traffic Data
        │
        ▼
Data Cleaning and Validation
        │
        ▼
70% Training / 15% Validation / 15% Test
        │
        ▼
Training-Only Preprocessing
        │
        ├── Percentile-based clipping
        ├── One-hot encoding
        └── Min-Max normalization
        │
        ▼
Initial Labeled / Unlabeled Pools
        │
        ▼
┌─────────────────────────────────────────────┐
│                 Agent 2                     │
│                                             │
│ LSTM Representation                         │
│        ↓                                    │
│ MLP Feature Scores                          │
│        ↓                                    │
│ Sequential Feature Selection                │
│        ↓                                    │
│ LIME-Guided Feature Evaluation              │
│        ↓                                    │
│ DDoS Classification                         │
│        ↓                                    │
│ Imbalance-Aware Reward                      │
└─────────────────────────────────────────────┘
        │
        │ Representations + Predictions
        ▼
┌─────────────────────────────────────────────┐
│                 Agent 1                     │
│                                             │
│ Active Learning                             │
│        ↓                                    │
│ Class-Centroid Affinity                     │
│        ↓                                    │
│ Entropy-Based Uncertainty                   │
│        ↓                                    │
│ Request Annotation / Skip                   │
└─────────────────────────────────────────────┘
        │
        │ Newly Annotated Samples
        ▼
Updated Labeled Training Set
        │
        └──────────────► Agent 2

Both Agents
        │
        ▼
ER-TRPO Policy Optimization
        │
        ├── Multi-Step Returns
        ├── Entropy Regularization
        ├── KL Trust Region
        ├── Conjugate Gradient
        └── Backtracking Line Search

Hyperparameters
        │
        ▼
Homotopy + BOHB Optimization
```

---

## Datasets

The framework supports the four benchmark datasets evaluated in the study:

| Dataset | Description |
|---|---|
| **KDDCup99** | Benchmark intrusion-detection dataset containing normal traffic and several attack categories |
| **ISCX-UNB** | Network-security dataset containing realistic benign and malicious traffic profiles |
| **DARPA** | Dataset containing controlled multi-stage intrusion and DDoS attack scenarios |
| **CICDDoS** | Modern DDoS traffic dataset containing legitimate traffic and multiple reflection/amplification attacks |

The proposed methodology formulates the final detection problem as **binary classification**:

```text
0 → Benign / Normal traffic
1 → Attack / DDoS traffic
```

Multiclass attack labels are mapped to the attack class when necessary.

---

## Data Preprocessing

The implementation follows a leakage-safe preprocessing strategy.

The complete dataset is divided into:

```text
Training   : 70%
Validation : 15%
Test       : 15%
```

All data-dependent preprocessing operations are fitted **only on the training partition**.

The preprocessing pipeline includes:

1. duplicate-record removal,
2. incomplete-record removal,
3. invalid/infinite-value filtering,
4. percentile-based numerical clipping,
5. categorical one-hot encoding,
6. Min-Max normalization to `[0, 1]`,
7. fixed-dimensional tensor construction.

The preprocessing parameters estimated from the training set are reused unchanged for validation and test data.

This prevents information leakage from the validation and test partitions.

---

## Repository Structure

```text
dual-agent-er-trpo-ddos/
│
├── dual_agent_er_trpo_ddos.py
├── README.md
└── LICENSE
```

The complete methodological implementation is contained in:

```text
dual_agent_er_trpo_ddos.py
```

This includes:

- data loading,
- preprocessing,
- dataset splitting,
- LSTM-MLP detection,
- Agent 1,
- Agent 2,
- ER-TRPO,
- multi-step returns,
- LIME,
- imbalance-aware rewards,
- Active Learning,
- Homotopy continuation,
- BOHB optimization,
- model training,
- evaluation,
- statistical analysis,
- and model saving.

---

## Requirements

The implementation is written in Python.

Main dependencies include:

```text
Python
NumPy
Pandas
PyTorch
scikit-learn
SciPy
LIME
ConfigSpace
HPBandSter
Joblib
```

Create a virtual environment:

```bash
python -m venv venv
```

Activate it on Linux/macOS:

```bash
source venv/bin/activate
```

Activate it on Windows:

```bash
venv\Scripts\activate
```

Install the required packages:

```bash
pip install numpy pandas torch scipy scikit-learn lime ConfigSpace hpbandster joblib
```

---

## Main Implementation File

The main executable script is:

```text
dual_agent_er_trpo_ddos.py
```

The program contains a `main()` function and can be executed directly from the command line.

---

## Basic Usage

General syntax:

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset DATASET_NAME \
    --data-path PATH_TO_DATASET
```

Supported dataset identifiers are:

```text
kddcup99
iscx-unb
darpa
cicddos
```

### KDDCup99

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset kddcup99 \
    --data-path ./data/kddcup99
```

### CICDDoS

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset cicddos \
    --data-path ./data/cicddos \
    --label-column Label
```

### ISCX-UNB

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset iscx-unb \
    --data-path ./data/iscx-unb \
    --label-column Label
```

### DARPA

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset darpa \
    --data-path ./data/darpa \
    --label-column Label
```

---

## Label Column

The implementation attempts to automatically identify commonly used label-column names.

If automatic detection is not possible, specify the column explicitly:

```bash
--label-column Label
```

Example:

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset cicddos \
    --data-path ./data/cicddos.csv \
    --label-column Label
```

---

## PCAP Input and CICFlowMeter

If packet-capture files are provided instead of already extracted flow tables, the implementation can call an external **CICFlowMeter** command before training.

Example:

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset cicddos \
    --data-path ./pcaps \
    --cicflowmeter-command "YOUR_CICFLOWMETER_COMMAND {input} {output}"
```

The command template must contain:

```text
{input}
{output}
```

These placeholders are replaced automatically with the input PCAP and generated flow-file paths.

---

## Homotopy-BOHB Hyperparameter Optimization

Hyperparameter optimization is performed automatically unless explicitly disabled.

The implemented search space includes:

| Hyperparameter | Search Range |
|---|---|
| Agent 1 batch size | 32 – 1024 |
| Agent 2 batch size | 32 – 1024 |
| Agent 1 learning rate | `1e-4` – `5e-2` |
| Agent 2 learning rate | `1e-4` – `5e-2` |
| Activation function | Leaky ReLU, ReLU, Tanh, Linear, Sigmoid |
| Agent 1 dropout | 0.0 – 0.7 |
| Agent 2 dropout | 0.0 – 0.7 |
| Training epochs | 16 – 1024 |
| LSTM layers | 1 – 8 |
| MLP layers | 1 – 8 |
| Affinity temperature | 0.1 – 2.0 |
| Annotation cost | 0.0 – 1.0 |
| Skip reward coefficient | 0.0 – 1.0 |
| Initial uncertainty threshold | 0.1 – 1.0 |
| Threshold decay rate | `1e-4` – 0.1 |
| Annotation budget | 100 – 5000 |
| Feature relevance weight | 0.0 – 1.0 |
| Subset complexity weight | 0.0 – 1.0 |
| Validation improvement weight | 0.0 – 1.0 |
| Agent 1 entropy coefficient | `1e-4` – 0.1 |
| Agent 2 entropy coefficient | `1e-4` – 0.1 |

The BOHB resource range is:

```text
Minimum budget : 10 epochs
Maximum budget : 90 epochs
```

---

## Running Without Hyperparameter Optimization

A previously optimized configuration can be reused:

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset kddcup99 \
    --data-path ./data/kddcup99 \
    --skip-hpo \
    --fixed-config ./best_config.json
```

This option is useful for repeated evaluation or reproducing a previously optimized experiment.

---

## Repeated Runs

The implementation supports multiple independent random seeds.

Example:

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset kddcup99 \
    --data-path ./data/kddcup99 \
    --seeds 42,123,2024,3407,5189
```

Results are reported individually and aggregated as:

```text
mean ± standard deviation
```

---

## Evaluation Metrics

The final test-set evaluation includes:

- Accuracy
- Sensitivity / Recall
- F-measure / F1-score
- Balanced Accuracy
- Matthews Correlation Coefficient (MCC)
- Precision
- Specificity
- False Positive Rate (FPR)
- False Negative Rate (FNR)
- ROC-AUC
- PR-AUC

The implementation additionally reports:

- mean number of selected features,
- number of acquired annotations,
- and final labeled-set size.

All metrics are computed from actual model predictions.

No reported experimental result is hard-coded into the implementation.

---

## Statistical Evaluation

The implementation can perform paired statistical comparisons when repeated-run baseline results are available.

Supported statistical analysis includes:

- paired two-sided t-test,
- 95% confidence intervals,
- Cohen's d effect size,
- Holm-Bonferroni correction for multiple comparisons.

A baseline-results JSON file can be supplied using:

```bash
--baseline-results ./baseline_results.json
```

Example structure:

```json
{
  "Baseline_A": {
    "accuracy": [0.91, 0.92, 0.90, 0.93, 0.91],
    "sensitivity": [0.90, 0.91, 0.89, 0.92, 0.90],
    "f_measure": [0.90, 0.91, 0.89, 0.92, 0.90],
    "balanced_accuracy": [0.90, 0.91, 0.89, 0.92, 0.90],
    "mcc": [0.80, 0.82, 0.78, 0.84, 0.81]
  }
}
```

The proposed method and every baseline should contain results from the same number of independent runs.

---

## Output Files

For each random seed, the implementation creates a separate result directory.

Example:

```text
dual_agent_ddos_results/
│
├── run_settings.json
├── aggregate_metrics.json
│
├── seed_42/
│   ├── model.pt
│   ├── preprocessor.joblib
│   ├── metrics.json
│   ├── training_history.json
│   ├── best_config.json
│   │
│   └── homotopy_bohb/
│       ├── best_config.json
│       ├── bohb_records.json
│       └── ...
│
├── seed_123/
│   └── ...
│
└── paired_statistics_holm.json
```

### `model.pt`

Contains:

- trained LSTM-MLP detector,
- Agent 1 policy network,
- Agent 1 value network,
- Agent 2 policy network,
- Agent 2 value network,
- hyperparameter configuration,
- feature names,
- fixed implementation settings.

### `preprocessor.joblib`

Contains preprocessing transformations fitted exclusively on the training data.

### `metrics.json`

Contains final test-set metrics for the corresponding independent run.

### `training_history.json`

Stores training statistics across epochs.

### `best_config.json`

Stores the hyperparameter configuration identified by Homotopy-BOHB.

### `aggregate_metrics.json`

Contains mean and standard deviation across independent experimental runs.

---

## Important Command-Line Options

Important options include:

```text
--dataset
--data-path
--label-column
--output-dir
--max-rows
--seeds
--skip-hpo
--fixed-config
--baseline-results
--cicflowmeter-command
--device
```

Additional methodological settings can also be controlled:

```text
--initial-labeled-fraction
--clip-lower-quantile
--clip-upper-quantile
--lstm-hidden-size
--policy-hidden-size
--value-hidden-size
--gamma
--n-step
--kl-delta
--cg-iterations
--cg-damping
--line-search-steps
--line-search-backtrack
--value-iterations
--homotopy-stage-count
--bohb-iterations-per-stage
--bohb-eta
--lime-num-samples
--lime-kernel-width
```

View all available options with:

```bash
python dual_agent_er_trpo_ddos.py --help
```

---

## Reproducibility

The implementation includes reproducibility controls for:

- Python random number generation,
- NumPy,
- PyTorch CPU execution,
- PyTorch CUDA execution,
- deterministic PyTorch operations when supported,
- reproducible dataset splitting,
- and reproducible DataLoader shuffling.

Multiple independent random seeds can be evaluated through the command line.

---

## Prevention of Data Leakage

The implementation explicitly separates:

```text
Training Set
Validation Set
Test Set
```

The **test set is not used during model training, active learning, reward calculation, feature selection, or hyperparameter optimization**.

The validation set is used for:

- Active Learning reward evaluation,
- feature-selection reward evaluation,
- BOHB configuration selection.

The test set is accessed only during the final evaluation stage.

Preprocessing transformations are estimated using training data only.

---

## Implementation Assumptions

Some low-level implementation details required for executable software are not explicitly specified in the manuscript.

The implementation therefore exposes these settings rather than deriving them from reported experimental outcomes.

Examples include:

- initial labeled-data fraction,
- exact percentile clipping thresholds,
- LSTM hidden dimension,
- policy-network hidden dimension,
- value-network hidden dimension,
- discount factor,
- n-step horizon,
- KL trust-region radius,
- conjugate-gradient iterations,
- conjugate-gradient damping,
- line-search settings,
- number of Homotopy stages,
- and LIME perturbation settings.

These choices are clearly separated from quantities selected through the proposed optimization procedure.

Reported results from the manuscript are **not used as computational inputs**.

---

## Computational Considerations

The full methodology is computationally demanding.

The major training costs originate from:

- LSTM-MLP forward and backward computation,
- repeated ER-TRPO policy optimization,
- conjugate-gradient trust-region updates,
- repeated validation during Active Learning,
- LIME perturbation-based explanations,
- detector updates following newly annotated samples,
- and multi-stage Homotopy-BOHB optimization.

For initial debugging, a smaller subset can be used:

```bash
--max-rows 10000
```

Example:

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset kddcup99 \
    --data-path ./data/kddcup99 \
    --max-rows 10000
```

This option is intended for implementation testing and should not replace the full dataset in final experimental reproduction.

---

## Hardware Acceleration

CUDA is automatically used when available.

Use GPU explicitly:

```bash
--device cuda
```

Use CPU explicitly:

```bash
--device cpu
```

Example:

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset cicddos \
    --data-path ./data/cicddos \
    --device cuda
```

---

## Methodological Components Implemented

- [x] Dataset loading
- [x] Duplicate removal
- [x] Incomplete-record filtering
- [x] Train/validation/test splitting
- [x] Training-only preprocessing
- [x] Percentile clipping
- [x] One-hot encoding
- [x] Min-Max normalization
- [x] LSTM representation learning
- [x] MLP feature scoring
- [x] Dual-agent reinforcement learning
- [x] Active Learning
- [x] Class-centroid affinity computation
- [x] Entropy-based sample uncertainty
- [x] Adaptive uncertainty threshold
- [x] Annotation-cost-aware reward
- [x] Sequential feature selection
- [x] LIME feature attribution
- [x] Feature-subset complexity penalty
- [x] Validation-performance reward
- [x] Class-frequency-aware classification reward
- [x] TRPO policy optimization
- [x] Multi-step returns
- [x] Entropy regularization
- [x] KL-divergence constraint
- [x] Conjugate-gradient optimization
- [x] Backtracking line search
- [x] Homotopy continuation
- [x] BOHB hyperparameter optimization
- [x] Repeated independent runs
- [x] Binary classification metrics
- [x] Statistical testing
- [x] Model persistence
- [x] Preprocessor persistence
- [x] Experimental-result reporting

---

## Reproducing an Experiment

A typical complete experiment can be executed as follows:

```bash
python dual_agent_er_trpo_ddos.py \
    --dataset cicddos \
    --data-path ./data/cicddos \
    --label-column Label \
    --seeds 42,123,2024,3407,5189 \
    --device cuda \
    --output-dir ./results/cicddos
```

The program will:

1. load the dataset,
2. remove invalid records,
3. generate training, validation, and test partitions,
4. fit preprocessing transformations using training data,
5. construct labeled and unlabeled training pools,
6. perform Homotopy-guided BOHB optimization,
7. initialize the optimized dual-agent architecture,
8. train Agent 2,
9. exchange detector representations and predictions with Agent 1,
10. perform adaptive sample acquisition,
11. update the labeled pool,
12. perform LIME-guided feature selection,
13. optimize both policies using ER-TRPO,
14. train the final configuration,
15. evaluate the untouched test set,
16. save the trained models and preprocessing pipeline,
17. and report the final metrics.

---

## Citation

If this repository is used in academic research, please cite the associated manuscript.

```bibtex
@article{dual_agent_er_trpo_ddos,
  title   = {An Intelligent Dual-Agent Reinforcement Learning Framework for DDoS Attack Detection},
  author  = {Authors},
  journal = {Journal},
  year    = {2026}
}
```

The BibTeX entry should be updated with the final publication metadata after publication.

---

## Paper

**Title:**  
*An Intelligent Dual-Agent Reinforcement Learning Framework for DDoS Attack Detection*

The framework combines:

> **Active Learning + LSTM/MLP + LIME + Imbalance-Aware Reinforcement Learning + ER-TRPO + Multi-Step Returns + Homotopy-BOHB**

for adaptive DDoS detection.

---

## License

A license should be added before public distribution of the repository.

Common options for academic software include:

- MIT License
- BSD 3-Clause License
- Apache License 2.0

The selected license should be consistent with the intended use and publication requirements.

---

## Contact

For questions regarding the methodology, implementation, or experimental reproduction, please use the repository **Issues** section.

---

<div align="center">

### Dual-Agent ER-TRPO for Adaptive and Interpretable DDoS Detection

</div>
