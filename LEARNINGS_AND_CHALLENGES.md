# Learnings and Challenges (Reflection)

## What I learned
- Label definition is as important as model selection. A strict churn horizon can inflate baseline churn and distort interpretation.
- Segmenting on model probabilities is useful only when scores are calibrated and interpreted relative to baseline.
- Separating one-time and repeat buyers creates cleaner business logic and more realistic retention actions.
- SMOTE helps class-imbalance learning but does not solve weak signal quality by itself.
- Visualization is essential for diagnosing model quality, segment quality, and business usability.

## Challenges faced
### 1. Segment imbalance in early approach
- Initial threshold-based segmentation pushed most customers into one bucket.
- Resolution: switched to KMeans segmentation on churn probabilities.

### 2. High churn rates across all segments
- Even low-risk group had high absolute churn.
- Root cause: high baseline churn from label definition.
- Resolution: revised churn logic and reported baseline-aware comparisons.

### 3. One-time buyers dominated the dataset
- One-time customers heavily outnumbered repeat buyers and changed overall behavior patterns.
- Resolution: split customer types, use separate churn logic, and build repeat-only model.

### 4. Model quality remained limited
- Repeat-only model achieved low ROC-AUC (~0.52), indicating weak separability.
- Resolution: added richer diagnostics (probability distribution, decile checks, segment profiles).

## What I would improve next
- Add more predictive features (delivery delays, order intervals, category-level behavior, discount usage).
- Try gradient boosting models (XGBoost/LightGBM/CatBoost) and compare calibration.
- Build separate models for one-time and repeat customers.
- Introduce time-based validation to better reflect real deployment.
- Track campaign outcomes to connect predicted risk to intervention success.

## Personal takeaway
This project moved from basic churn reporting to a structured, action-oriented retention analytics workflow. The most important learning is that business framing and target definition determine whether model outputs are actually usable by decision-makers.
