# PAMMELA: Policy Administration Methodology using Machine Learning (with 5 Categorical Evasion Attacks)

An end-to-end implementation and empirical robustness evaluation of **PAMMELA (Policy Administration Methodology using Machine Learning)** against **Five Distinct Categorical Adversarial Evasion Attacks**, replicating and extending the research paper:

> **Paper Reference:**  
> P. Sen, A. K. Biswas, and D. Bhattacharyya.  
> *"PAMMELA: Policy Administration Methodology using Machine Learning"*,  
> **Proceedings of the 19th International Conference on Security and Cryptography (SECRYPT 2022)**, pages 105–115.  
> DOI: [10.5220/0011270800003280](https://doi.org/10.5220/0011270800003280)

---

## 📌 Overview

Attribute-Based Access Control (ABAC) is widely used to enforce fine-grained access policies based on subject and object attributes. However, authoring and maintaining ABAC policies across dynamic organizational environments requires significant administrative overhead.

**PAMMELA** automates ABAC administration by framing policy evaluation as a supervised machine learning classification task:
1. **Phase 1 (Synthetic Rule Expansion):** Generates a candidate universe $U = S \times O$ of subject-object attribute pairs from initial policy rules, extracting positive rules ($PR$) and negative rules ($NR = U \setminus PR$).
2. **Phase 2 (Monotonic Categorical Encoding):** Maps categorical attributes into an ordered integer feature space with missing-value handling (`NA = 0`).
3. **Phase 3 (Model Training & Inference):** Trains 6 diverse classifiers (ANN, Decision Tree, Random Forest, Extra Trees, Gradient Boosting, XGBoost) to classify arbitrary access requests as `Allow` (1) or `Deny` (0).
4. **Adversarial Extension (This Repository):** Comprehensive benchmark of **Five Categorical Evasion Attacks** ($L_0 \le 1$ Hamming distance) targeting privilege escalation (`Deny` $\to$ `Allow`) to evaluate the adversarial robustness of tabular ABAC classifiers.

---

## 🏗️ Repository Structure

```
├── pammela.ipynb                           # Complete Jupyter Notebook with code, outputs, and all plots
├── pammela.pdf                             # Original SECRYPT 2022 research paper
├── requirements.txt                        # Python dependencies
├── .gitignore                              # Excludes temporary cache and checkpoints
├── inferred_abac_augmented_rules.csv       # Phase 1 generated labeled universe dataset
├── dataset/                                # University ABAC policy dataset
│   ├── pol_data-v1.csv                     # Initial 53 core ABAC rules
│   ├── sub_data.csv                        # 700 subject profiles (Role, Dept, Degree, Year)
│   ├── obj_data.csv                        # 1,348 object profiles (Type, Dept, Year, Degree)
│   ├── model_ABAC_train_data_set.xlsx      # Pre-compiled training dataset
│   └── model_ABAC_new_test_data_set.csv    # 1,010 test requests (598 Allow, 412 Deny)
└── plots/                                  # High-resolution exported benchmark charts
    ├── 01_eda_dataset_distributions.png
    ├── 02_benchmark_metrics_comparison.png
    ├── 03_security_annotated_confusion_matrices.png
    ├── 04_roc_and_pr_curves.png
    ├── 05_feature_importance_and_metrics_heatmap.png
    ├── 06_accuracy_f1_before_after_attack.png
    ├── 07_attack_success_rate_comparison.png
    └── 08_most_exploited_attributes.png
```

---

## ⚙️ Evaluated Classifiers & Clean Baseline Performance

All models were evaluated on the **University ABAC Dataset 1** (1,010 real access requests: 598 Allow, 412 Deny):

| Model | Clean Acc | Clean Recall | Clean F1 | Paper Table 4 Acc | Paper Table 4 Rec |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Extra Trees (ET)** | **0.9426** | **0.9247** | **0.9502** | 0.880 | 0.840 |
| **XGBoost (XGB)** | **0.8842** | **0.8328** | **0.8949** | 0.864 | 0.831 |
| **Decision Tree (DT)** | **0.7673** | **0.6656** | **0.7721** | 0.779 | 0.654 |
| **Gradient Boosting (GB)** | **0.7584** | **0.5920** | **0.7437** | 0.803 | 0.709 |
| **Neural Net (ANN)** | **0.7574** | **0.6371** | **0.7567** | 0.710 | 0.623 |
| **Random Forest (RF)** | **0.6851** | **0.4900** | **0.6475** | 0.762 | 0.640 |

![Benchmark Comparison](plots/02_benchmark_metrics_comparison.png)

---

## ⚔️ The 5 Evaluated Categorical Evasion Attacks

Unlike network security systems (e.g. ELAT) where continuous gradient perturbations ($\ell_\infty$-bounded FGSM/PGD) are applied, tabular ABAC systems require **discrete categorical perturbations** adhering to valid attribute domains $\mathcal{D}_j$:

| Evasion Method | How it Finds an Adversarial Request | Applicable Models |
|---|---|---|
| **1. Exhaustive Categorical Search** | Enumerates every permitted category substitution within budget $L_0 \le 1$. Provides the exact upper-bound benchmark for evasion existence. | All six (ANN, DT, RF, ET, GB, XGB) |
| **2. Greedy Categorical Search** | Greedily selects the category substitution that maximizes the model's predicted probability of `Allow` ($P(\hat{y}=1)$). | All six models |
| **3. Evolutionary Search (GA)** | Stochastically explores multi-path perturbations using population-based mutation, uniform crossover, and fitness ranking. | All six models |
| **4. Tree-Based Optimization** | Inverts tree decision paths by finding the minimal-distance valid category combination that satisfies leaves predicting `Allow`. | DT, RF, ET, GB, XGBoost |
| **5. Probabilistic Categorical (PCAA)** | Employs continuous categorical relaxation (Softmax with temperature) to optimize categorical logits $\boldsymbol{\alpha}$ via gradient ascent on the ANN loss surface, then discretizes back. | Artificial Neural Network (ANN) |

---

## 📊 Comprehensive Master Benchmark Table (All 5 Attacks)

| Attack Method | Classifier | Clean Acc | Adv Acc | Acc Drop (%) | Clean F1 | Adv F1 | F1 Drop (%) | Clean Breaches | Adv Breaches | Attack Success Rate (ASR) | Time (s) |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Exhaustive Search** | **ANN** | 0.7574 | 0.5950 | -16.24% | 0.7567 | 0.6507 | -10.60% | 28 | 192 | **46.60%** | 0.78s |
| **Exhaustive Search** | **DT** | 0.7673 | 0.5683 | -19.90% | 0.7721 | 0.6461 | -12.60% | 35 | 236 | **57.28%** | 0.75s |
| **Exhaustive Search** | **RF** | 0.6851 | 0.4713 | -21.39% | 0.6475 | 0.5224 | -12.51% | 12 | 228 | **55.34%** | 2.38s |
| **Exhaustive Search** | **ET** | 0.9426 | 0.7218 | -22.08% | 0.9502 | 0.7974 | -15.28% | 13 | 236 | **57.28%** | 2.28s |
| **Exhaustive Search** | **GB** | 0.7584 | 0.7149 | -4.36% | 0.7437 | 0.7108 | -3.29% | 0 | 44 | **10.68%** | 0.78s |
| **Exhaustive Search** | **XGB** | 0.8842 | 0.6673 | -21.68% | 0.8949 | 0.7477 | -14.71% | 17 | 236 | **57.28%** | 2.47s |
| **Greedy Search** | **ANN** | 0.7574 | 0.5950 | -16.24% | 0.7567 | 0.6507 | -10.60% | 28 | 192 | **46.60%** | 1.18s |
| **Greedy Search** | **DT** | 0.7673 | 0.5683 | -19.90% | 0.7721 | 0.6461 | -12.60% | 35 | 236 | **57.28%** | 1.07s |
| **Greedy Search** | **RF** | 0.6851 | 0.4713 | -21.39% | 0.6475 | 0.5224 | -12.51% | 12 | 228 | **55.34%** | 4.76s |
| **Greedy Search** | **ET** | 0.9426 | 0.7218 | -22.08% | 0.9502 | 0.7974 | -15.28% | 13 | 236 | **57.28%** | 4.83s |
| **Greedy Search** | **GB** | 0.7584 | 0.7149 | -4.36% | 0.7437 | 0.7108 | -3.29% | 0 | 44 | **10.68%** | 1.38s |
| **Greedy Search** | **XGB** | 0.8842 | 0.6673 | -21.68% | 0.8949 | 0.7477 | -14.71% | 17 | 236 | **57.28%** | 5.08s |
| **Evolutionary (GA)** | **ANN** | 0.7574 | 0.6089 | -14.85% | 0.7567 | 0.6586 | -9.81% | 28 | 178 | **43.20%** | 3.00s |
| **Evolutionary (GA)** | **DT** | 0.7673 | 0.5822 | -18.51% | 0.7721 | 0.6535 | -11.85% | 35 | 222 | **53.88%** | 2.51s |
| **Evolutionary (GA)** | **RF** | 0.6851 | 0.4921 | -19.31% | 0.6475 | 0.5324 | -11.51% | 12 | 207 | **50.24%** | 10.47s |
| **Evolutionary (GA)** | **ET** | 0.9426 | 0.7446 | -19.80% | 0.9502 | 0.8109 | -13.93% | 13 | 213 | **51.70%** | 16.28s |
| **Evolutionary (GA)** | **GB** | 0.7584 | 0.7149 | -4.36% | 0.7437 | 0.7108 | -3.29% | 0 | 44 | **10.68%** | 6.74s |
| **Evolutionary (GA)** | **XGB** | 0.8842 | 0.6871 | -19.70% | 0.8949 | 0.7591 | -13.57% | 17 | 216 | **52.43%** | 12.90s |
| **Tree Optimization** | **DT** | 0.7673 | 0.5683 | -19.90% | 0.7721 | 0.6461 | -12.60% | 35 | 236 | **57.28%** | 1.13s |
| **Tree Optimization** | **RF** | 0.6851 | 0.4713 | -21.39% | 0.6475 | 0.5224 | -12.51% | 12 | 228 | **55.34%** | 3.89s |
| **Tree Optimization** | **ET** | 0.9426 | 0.7218 | -22.08% | 0.9502 | 0.7974 | -15.28% | 13 | 236 | **57.28%** | 3.75s |
| **Tree Optimization** | **GB** | 0.7584 | 0.7149 | -4.36% | 0.7437 | 0.7108 | -3.29% | 0 | 44 | **10.68%** | 1.42s |
| **Tree Optimization** | **XGB** | 0.8842 | 0.6673 | -21.68% | 0.8949 | 0.7477 | -14.71% | 17 | 236 | **57.28%** | 3.46s |
| **PCAA (Continuous)** | **ANN** | 0.7574 | 0.6396 | -11.78% | 0.7567 | 0.6767 | -8.00% | 28 | 147 | **35.68%** | 1.61s |

---

## 📈 Visualizations & Robustness Analysis

### 1. Multi-Attack Attack Success Rate Comparison
![Multi-Attack ASR](plots/06_accuracy_f1_before_after_attack.png)
*Comparison of Attack Success Rate (ASR %) across models and attack methods.*

### 2. Accuracy & F1-Score Degradation Across Attack Paradigms
![Degradation Heatmaps](plots/07_attack_success_rate_comparison.png)
*Heatmap matrix showing Accuracy and F1-score drops across all combinations.*

### 3. Computational Trade-offs & Security Breach Explosion
![Efficiency & Breaches](plots/08_most_exploited_attributes.png)
*Execution time vs. attack power scatter analysis and XGBoost security breach explosion.*

---

## 🔍 Key Findings

1. **Greedy and Tree Optimization Match Exhaustive Search:**
   Under a single-attribute budget ($k=1$), **Greedy Search** and **Tree Optimization** achieve the exact same peak Attack Success Rate (**57.28%**) as Exhaustive Search, proving that greedy path traversal identifies the global optimum in single-attribute categorical evasion.
2. **Gradient Boosting (GB) Demonstrates Inherent Resilience:**
   Across all 4 applicable attacks, Gradient Boosting limited ASR to **10.68%** with an accuracy drop of only **4.36%**. Its conservative shrinkage parameter ($\eta = 0.1$) prevents any single attribute from overpowering the ensemble log-odds margin.
3. **PCAA Validates Differentiable Categorical Relaxation for ANNs:**
   PCAA achieved **35.68% ASR** against the Neural Network in just **1.61 seconds** by backpropagating gradients through expected categorical distributions.
4. **Security Breach Reality:**
   Unauthorized access grants (False Positive Accesses) exploded from **17 clean errors to 236 illegal grants** in XGBoost—a **+1,288% increase in security breaches**.

---

## 🛡️ Recommended Defenses

- **Discrete Adversarial Training (Tabular AT):** Augment the Phase 1 training set with discrete single-attribute perturbations of negative rules.
- **Relational Invariant Feature Engineering:** Provide explicit boolean predicates ($\text{Subject.Dept} == \text{Object.Dept}$, $\text{Subject.Year} == \text{Object.Year}$) to eliminate single-attribute axis-aligned bypasses.
- **Defense-in-Depth Verification:** Enforce deterministic two-person or cryptographic verification for high-risk resources (`q_pr`, `dept_bud`).

---

## 🚀 How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/Harpil70/pammla-implementation-with-attacks.git
   cd pammla-implementation-with-attacks
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Launch Jupyter Notebook and run all cells in `pammela.ipynb`:
   ```bash
   jupyter notebook pammela.ipynb
   ```
