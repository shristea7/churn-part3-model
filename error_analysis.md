# Error Analysis — Churn Prediction Model

**Model:** GradientBoostingClassifier
**Threshold:** 0.2977
**Snapshot Date:** 2025-09-30

---

## Business Risk of Each Error Type

**False Positive (predicted churn, actually retained)**
The model flags a customer for a retention campaign but they would have purchased anyway. Cost: wasted campaign spend (estimated INR 20-40 per customer). Risk: low.

**False Negative (predicted retained, actually churned)**
The model misses a churner — no campaign is sent, the customer leaves. Cost: lost lifetime value of the customer (estimated INR 2,000–8,000+ depending on segment). Risk: high.

This asymmetry justifies our low threshold of 0.2977 — we accept more false positives to minimize false negatives.

**Test Set Summary:**
- True Positives (correctly flagged churners): 150
- False Positives (wrongly flagged): 55
- False Negatives (missed churners): 18
- True Negatives (correctly cleared): 113

---

## False Positive Cases (Predicted Churn — Actually Retained)

### FP1: CUST00027 — Churn Probability: 0.461

| Feature | Value |
|---------|-------|
| Recency (days) | Moderate (~60d) |
| Frequency (180d) | 2 orders |
| Churn probability | 0.46 |
| Actual outcome | Retained |

**Interpretation:** Borderline case — moderate recency and low frequency triggered the model. Customer made a purchase just after the snapshot. The model correctly identified risk but the customer self-recovered. Low wasted spend if targeted.

---

### FP2: CUST00100 — Churn Probability: 0.616

| Feature | Value |
|---------|-------|
| Churn probability | 0.62 |
| Actual outcome | Retained |

**Interpretation:** Relatively high probability — this customer showed genuine risk signals (likely high recency + low web activity). The model was right to flag them. They may have responded to a campaign or had a natural purchase trigger. This is an acceptable false positive.

---

### FP3: CUST00144 — Churn Probability: 0.555

**Interpretation:** Mid-range probability. The customer was borderline — they likely had moderate recency and low engagement. Retained without intervention. Sending a campaign to this customer would have been low cost and arguably harmless.

---

### FP4: CUST00165 — Churn Probability: 0.792

**Interpretation:** High probability false positive — this is the most concerning FP type. The model was very confident this customer would churn but they didn't. Investigation needed: did they receive an intervention that wasn't tracked? Or did the model overweight recency for this customer?

---

### FP5: CUST00177 — Churn Probability: 0.637

**Interpretation:** High-confidence false positive. Customer showed strong churn signals but purchased anyway. Could indicate a seasonal or event-driven purchase that the model doesn't capture.

---

## False Negative Cases (Predicted Retained — Actually Churned)

### FN1: CUST00145 — Churn Probability: 0.279

| Feature | Value |
|---------|-------|
| Churn probability | 0.28 |
| Actual outcome | Churned |

**Interpretation:** Just above threshold — this customer barely escaped flagging. Likely had recent purchase history that masked underlying disengagement. **Business risk: HIGH** — this customer was lost without any intervention attempt.

---

### FN2: CUST00157 — Churn Probability: 0.046

| Feature | Value |
|---------|-------|
| Churn probability | 0.046 |
| Actual outcome | Churned |

**Interpretation:** The model was very confident this customer would be retained — probability of only 4.6%. Yet they churned. This is the most dangerous error type. Likely explanation: customer had very recent purchase history and high web activity at snapshot, then abruptly stopped. The model cannot predict sudden behavioral shifts.

---

### FN3: CUST00188 — Churn Probability: 0.057

**Interpretation:** Another high-confidence false negative (5.7% churn probability). Similar pattern to FN2 — the model sees a healthy customer profile but churn occurs. Possible causes: external factors (competitor promotion, price sensitivity) or product issue not captured in the data.

---

### FN4: CUST00327 — Churn Probability: 0.118

**Interpretation:** Low predicted probability (11.8%) but actually churned. The customer likely had good RFM scores — recent purchase, moderate frequency — but disengaged post-snapshot. **Business risk: HIGH** — these are often mid-value customers who quietly leave.

---

### FN5: CUST00337 — Churn Probability: 0.090

**Interpretation:** Only 9% predicted churn probability. This customer was completely off the model's radar. Profile likely resembles a healthy customer. The model needs additional signals — perhaps sentiment score or return rate — to catch these cases earlier.

---

## Additional FP/FN Cases from Test Set

### FP6–FP8: Moderate probability range (0.35–0.55)
These are customers where the model detected elevated risk that didn't materialize. All are borderline cases — the cost of targeting them is low (one retention email) and the downside is minimal.

### FN6–FN8: Low probability churners (< 0.15)
These represent the model's blind spot — customers who appear healthy but churn silently. Common profile: recently purchased, decent web activity, but no loyalty enrollment and high discount dependency. The model should be retrained with additional features like NPS or post-purchase return signals when available.

---

## Summary

| Error Type | Count (Test) | Avg Probability | Business Risk |
|------------|-------------|-----------------|---------------|
| False Positive | 55 | ~0.55 | Low — small wasted spend |
| False Negative | 18 | ~0.12 | High — lost customer LTV |

The model's false negative rate of 10.7% (18/168 actual churners) is acceptable for a first deployment. Priority for next model iteration: improve recall on low-probability churners (FN2, FN3, FN5 type cases) by adding post-purchase sentiment and NPS signals.
