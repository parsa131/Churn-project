# Telco Customer Churn Prediction

Predicts whether a telecom customer will churn, using IBM's public
Telco Customer Churn dataset (Demographics, Location, Population,
Services, and Status tables).

## Overview

- **Goal:** classify whether a customer will churn (binary classification)
- **Data:** 7,043 customers, merged from 5 relational Excel files
- **Models compared:** Logistic Regression, XGBoost, SVM

## Data Cleaning & Feature Engineering

- Merged the 5 source tables on `Customer ID` (and `Zip Code` →
  `Population`, using `Location` as the bridge table between them)
- **Dropped leakage columns:**
  - `Churn Score` — a pre-computed churn probability from another model;
    training on it would mean copying that model instead of learning
    from real customer behavior
  - `Churn Reason` — only populated *after* a customer has already
    churned, so it's not knowable at prediction time
- **Dropped high-cardinality / redundant identifiers:**
  `Customer ID`, `Location ID`, `Service ID`, `Status ID`, `City`
  (1,111 unique values — too sparse to one-hot encode without
  overfitting), `Zip Code`, `State` (constant, no variance),
  `Lat Long` (redundant with `Latitude`/`Longitude`)
- **Filled meaningful nulls** rather than dropping rows:
  - `Offer` → `"No Offer"` (blank meant no promotion was given)
  - `Internet Type` → `"No Internet Service"` (blank meant no
    internet subscription)
- **Encoding:**
  - One-hot encoded nominal categories with no natural order:
    `Offer`, `Internet Type`, `Payment Method`
  - Ordinal-encoded `Contract` (Month-to-Month → One Year → Two Year),
    since commitment length is a genuinely ordered scale
  - Binary Yes/No columns mapped directly to 0/1

## Results

| Model | Accuracy | Precision (Churn) | Recall (Churn) | F1 (Churn) |
|---|---|---|---|---|
| Logistic Regression | 92.19% | 0.90 | 0.80 | 0.85 |
| XGBoost | 96.80% | 0.94 | 0.89 | 0.91  |
| SVM | 96.24% | 0.97 | 0.88 | 0.93 |

XGBoost performed best overall. Since churn is imbalanced (~26.5% of
customers), recall on the churned class was prioritized alongside
accuracy — missing an at-risk customer is typically costlier for a
business than a false alarm.

## Key Findings

- `Satisfaction Score` had the strongest correlation with churn
  (-0.75) — the single most predictive feature in the dataset
- `Tenure in Months` was strongly negatively correlated with churn
  (-0.35): newer customers churn far more than long-tenured ones
- Month-to-month contract customers churned significantly more than
  those on one- or two-year contracts
- `Tenure`, `Total Charges`, and `Total Revenue` were highly
  correlated with each other (up to 0.97), reflecting that they're
  largely derived from the same underlying spend-over-time signal

## Tech Stack

Python, pandas, scikit-learn, XGBoost, seaborn, matplotlib

## Repo Structure