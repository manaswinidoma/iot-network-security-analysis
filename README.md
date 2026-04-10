# IoT Network Intrusion Detection – Multi-Class Classification

A machine learning project for classifying network traffic attacks targeting IoT devices. The goal is to accurately identify three types of intrusions — **Mirai-greip_flood**, **Recon-OSScan**, and **DictionaryBruteForce** — to support security analytics in real IoT environments.

---

## Dataset

- **Samples:** 15,600
- **Features:** 48 (network traffic metrics including flow duration, header length, protocol types, flag counts, inter-arrival times, packet length statistics, and a class label)
- **Class Distribution (imbalanced):**
  | Class | Samples |
  |---|---|
  | Mirai-greip_flood | 13,561 |
  | Recon-OSScan | 1,780 |
  | DictionaryBruteForce | 259 |

---

## Preprocessing

17 non-informative features were dropped. Two separate preprocessing pipelines were implemented based on classifier type:

### For Decision Trees & Random Forest
- **Equi-Depth Binning** on `Duration` and `Protocol Type` (to handle skewness)
- **Z-Score Normalization** on `Weight`
- **Discretization** of `flow_duration` into Small / Medium / Large categories
- **Binarization** of `Header_Length` using median threshold
- **One-Hot Encoding** of target labels

### For SVM, KNN & MLP
- **StandardScaler** on all continuous features
- **OneHotEncoder** on categorical features (flags, protocol indicators)
- **One-Hot Encoding** of target labels
- **SMOTE** (Synthetic Minority Over-sampling Technique) applied within the pipeline for KNN and MLP to address class imbalance

---

## Classifiers & Results

All models used **GridSearchCV** with **5-fold Stratified K-Fold cross-validation** and were evaluated using Precision, Recall, F1 Score, and AUC-ROC.

| Classifier | Mean CV F1 | AUC-ROC |
|---|---|---|
| **Random Forest** | **0.908** | **0.9995** |
| Decision Tree | 0.892 | 0.9947 |
| MLP (+ SMOTE) | 0.792 | 0.9912 |
| SVM | 0.730 | 0.9972 |
| KNN (+ SMOTE) | 0.722 | 0.9281 |

### Best Model — Random Forest
**Optimal Hyperparameters:**
- `n_estimators`: 400
- `max_depth`: 15
- `min_samples_leaf`: 4
- `min_samples_split`: 2
- `class_weight`: balanced

**Per-class Performance:**
| Class | Precision | Recall | F1 |
|---|---|---|---|
| Mirai-greip_flood | 0.972 | 0.931 | 0.951 |
| Recon-OSScan | 1.000 | 0.999 | 0.999 |
| DictionaryBruteForce | 0.981 | 0.996 | 0.988 |

**Top Features by Importance:** IAT, Min, rst_count, urg_count, Variance

---

## Requirements

```bash
pip install pandas numpy scikit-learn imbalanced-learn matplotlib seaborn openpyxl
```

---

## How to Run

1. Clone or download the repository
2. Install the required packages (see above)
3. Open and run `iot-network-security-analysis.ipynb` in Jupyter Notebook or JupyterLab

---

## Key Findings

- **Random Forest** was selected as the best model due to its strong handling of class imbalance via `class_weight='balanced'`, high performance across all three classes, and interpretable feature importances.
- **SMOTE** improved minority class detection for KNN and MLP but did not surpass tree-based ensemble methods overall.
- **IAT (Inter-Arrival Time)** and **Min packet length** were consistently the most influential features across classifiers.
- Tailored preprocessing — separate pipelines for tree-based vs. distance/gradient-based models — was essential for maximising each classifier's performance.
