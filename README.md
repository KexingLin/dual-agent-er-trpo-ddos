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

**Agent 1 – Active Learning Agent**

Agent 1 determines whether an unlabeled network-traffic instance should be requested for annotation.

Its state combines:

- the LSTM-derived representation of the sample,
- and the corresponding prediction probability produced by Agent 2.

Agent 1 estimates uncertainty using class-centroid affinities calculated exclusively from currently labeled training samples.

The acquisition decision accounts for:

- uncertainty,
- annotation cost,
- and the resulting change in validation balanced accuracy.

---

**Agent 2 – Feature Selection and Detection Agent**

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
