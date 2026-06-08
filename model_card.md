# Model Card — D2C Churn Prediction Model

---

## Model Details

| Field | Value |
|-------|-------|
| Model type | GradientBoostingClassifier (scikit-learn) |
| Version | 1.0 |
| Snapshot date | 2025-09-30 |
| Target window | 60 days post-snapshot (2025-10-01 to 2025-11-29) |
| Decision threshold | 0.2977 |
| Output | Churn probability (0–1) + binary prediction |

---

## Intended Use

**Primary use:** Identify customers at risk of churning in the next 60 days so the retention team can prioritize outreach and campaign spend.

**Intended users:** CRM team, marketing team, customer success team.

**Intended workflow:** Run model scores monthly after snapshot. Pass high-risk customers (predicted = 1) to the retention campaign pipeline. Do not use scores to make irreversible decisions (e.g. account termination).

---

## Data Used

The model was trained on the `rfm_modeling_snapshot.csv` — a pre-built feature table containing:

- **RFM signals:** recency_days, frequency_180d, monetary_180d
- **Behavioral signals:** return_rate_180d, avg_discount_pct_180d, avg_rating_180d, category_diversity_180d
- **Support signals:** ticket_count_90d, negative_ticket_rate_90d, avg_resolution_hours_90d
- **Web/app signals:** sessions_30d, product_views_30d, cart_adds_30d, wishlist_adds_30d, abandoned_carts_30d, email_opens_30d, campaign_clicks_30d, last_visit_days_ago
- **Profile signals:** city_tier, age_group, acquisition_channel, loyalty_tier, preferred_category, marketing_consent, days_since_signup

**Leakage prevention:** All features are aggregated as of 2025-09-30. Post-snapshot order data was explicitly excluded. The target variable (churn_next_60d) was never used as a feature.

**Train/val/test split:** 1,728 / 336 / 336 customers using pre-assigned splits from churn_labels.csv.

---

## Model Approach

1. **Baseline:** Logistic Regression (threshold = 0.50) — establishes a simple linear benchmark.
2. **Final model:** Gradient Boosting Classifier (200 estimators, depth 4, learning rate 0.05) — captures non-linear interactions between recency, web activity, and support behavior.
3. **Threshold:** Optimized at 0.2977 using F1-score on validation set. Lower threshold chosen to prioritize recall — missing a churner is more costly than a false alarm.

---

## Performance

### Validation Set

| Metric | Logistic Regression | Gradient Boosting |
|--------|--------------------|--------------------|
| AUC-ROC | 0.8845 | 0.8773 |
| F1 Score | 0.7900 | 0.7951 |
| Precision | 0.8284 | 0.7222 |
| Recall | 0.7551 | 0.8844 |

### Test Set (Final — Gradient Boosting)

| Metric | Value |
|--------|-------|
| AUC-ROC | 0.8646 |
| F1 Score | 0.8043 |
| Precision | 0.7317 |
| Recall | 0.8929 |
| Accuracy | 0.7827 |
| True Positives | 150 |
| False Positives | 55 |
| False Negatives | 18 |
| True Negatives | 113 |

The model correctly identifies **89.3% of actual churners** on the test set.

---

## Top Features

| Rank | Feature | Importance | Business Meaning |
|------|---------|------------|-----------------|
| 1 | recency_days | 0.5737 | Days since last order — strongest single predictor |
| 2 | monetary_180d | 0.0918 | Total spend — high-value customers churn differently |
| 3 | days_since_signup | 0.0457 | Customer tenure — newer customers churn more |
| 4 | last_visit_days_ago | 0.0387 | Web inactivity — strong early warning signal |
| 5 | product_views_30d | 0.0283 | Browsing intent — customers still looking may convert |
| 6 | return_rate_180d | 0.0259 | Dissatisfaction proxy |
| 7 | avg_discount_pct_180d | 0.0209 | Discount dependency — price-sensitive customers churn when deals stop |
| 8 | avg_rating_180d | 0.0185 | Satisfaction signal |
| 9 | sessions_30d | 0.0167 | App/web engagement |
| 10 | cart_adds_30d | 0.0166 | Purchase intent |

**Key insight:** Recency alone accounts for 57% of model importance. A customer who hasn't ordered in 90+ days is very likely to churn regardless of other signals.

---

## Limitations

- **Recency dominance:** The model is heavily driven by recency. It may underperform for customers who purchase seasonally or infrequently by nature.
- **No causal inference:** The model predicts churn probability, not the cause of churn. A high score tells you who is at risk, not why.
- **Static snapshot:** The model scores customers at a fixed point in time. It cannot react to real-time events (e.g. a bad delivery happening today).
- **Cold start problem:** New customers with fewer than 30 days of history will have sparse features and lower prediction reliability.
- **No NPS or qualitative data:** Customer satisfaction signals beyond ticket sentiment are not available. Hidden dissatisfaction is the model's main blind spot (see false negatives FN2, FN3).

---

## Ethical Risks

- **Discount targeting:** Using this model to automatically send discounts to high-risk customers may disproportionately benefit customers who were already planning to purchase — eroding margin without reducing churn.
- **Segment bias:** If certain demographic groups (e.g. Tier 3 city customers) have systematically different purchase patterns, the model may under-serve or over-target them. Audit performance by city_tier and age_group before full deployment.
- **Feedback loop:** If only model-flagged customers receive retention campaigns, the training data for the next model iteration will be biased — customers who were not flagged and churned will appear as "natural churners" rather than "missed cases." Maintain a holdout group that receives no intervention for ongoing calibration.

---

## Monitoring Needs

| Signal | Frequency | Alert Threshold |
|--------|-----------|-----------------|
| Predicted churn rate | Monthly | >60% or <35% — investigate data drift |
| Model AUC on new labels | Quarterly | AUC drops below 0.80 — retrain |
| False negative rate | Quarterly | FN rate > 15% — lower threshold |
| Feature distribution shift | Monthly | Recency or sessions mean shifts >20% — flag for review |
| Business outcome | Quarterly | Compare churn rate of targeted vs untargeted customers |

---

## When NOT to Use This Model

- Do not use scores to deny customers access to products or services.
- Do not use as the sole reason for offering or withholding a discount — combine with segment logic from Part 2.
- Do not use beyond 60 days after the snapshot date without rerunning predictions on a fresh snapshot.
- Do not use for customers with fewer than 1 order in history — predictions will be unreliable.
- Do not use if the underlying data pipeline changes and feature definitions are modified — recalibrate threshold before redeploying.
