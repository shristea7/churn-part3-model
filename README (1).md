# Part 3: Churn Prediction Model & Model Card

D2C Customer Churn Capstone

## Overview

This repository contains a full churn prediction pipeline — data preparation, leakage checks, baseline and stronger model training, threshold selection, error analysis, and a structured model card.

## Repository Structure

```
part3/
├── churn_model.ipynb    # Full modeling notebook with all outputs
├── model.pkl            # Saved GradientBoosting model + encoders + threshold
├── metrics.json         # All evaluation metrics (test set)
├── error_analysis.md    # 10 FP/FN cases with business interpretation
├── model_card.md        # Structured model card for stakeholders
├── requirements.txt     # Python dependencies
├── data/                # Place all 7 CSV files here (see Data Setup)
└── charts/              # Auto-generated evaluation charts
```

## Data Setup

Download the dataset from the Google Drive link provided in the assignment and place all CSV files inside the `data/` folder:

```
https://drive.google.com/drive/folders/1PmLapJI1VSDgvl_AxARNKwM1MCd3WFX0?usp=sharing
```

## Setup & Run

```bash
pip install -r requirements.txt
jupyter notebook churn_model.ipynb
```

Run all cells top to bottom. `model.pkl` and `metrics.json` are generated automatically.

## Loading the Model

```python
import pickle, pandas as pd

with open('model.pkl', 'rb') as f:
    artifact = pickle.load(f)

model     = artifact['model']
encoders  = artifact['encoders']
features  = artifact['features']
threshold = artifact['threshold']

# Predict on new data (must have same feature columns)
proba = model.predict_proba(X_new)[:,1]
pred  = (proba >= threshold).astype(int)
```

## Why Gradient Boosting over Logistic Regression

Logistic Regression achieved slightly higher validation AUC (0.8845 vs 0.8773), but Gradient Boosting was selected as the final model because it achieved higher recall (89.3%) and F1-score.

Since missing a churner is significantly more costly than sending an unnecessary retention campaign, recall was prioritized over precision. A missed churner means lost lifetime value (INR 2,000–8,000+). A false alarm means one wasted campaign email (INR 20–40).


## Model Performance (Test Set)

| Metric | Value |
|--------|-------|
| AUC-ROC | 0.8646 |
| F1 Score | 0.8043 |
| Precision | 0.7317 |
| Recall | 0.8929 |
| Accuracy | 0.7827 |
| Threshold | 0.2977 |

## Key Finding

`recency_days` is the single most important feature (57.4% importance). Customers who haven't ordered in 90+ days are very likely to churn. The model correctly identifies 89.3% of actual churners on the held-out test set.
