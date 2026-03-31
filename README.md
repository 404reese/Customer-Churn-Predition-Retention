# Customer Churn Prediction and Retention Analysis

## What this project is
This project builds an end-to-end churn analytics workflow for an e-commerce dataset (Olist).
It combines:
- customer behavior feature engineering,
- churn labeling,
- machine learning prediction,
- customer risk segmentation,
- and export for Power BI reporting.

The goal is not only to report churn, but to identify which customers are at relatively higher risk and why, so retention teams can act earlier.

## Project goals
- Build a customer-level churn dataset from raw transactional tables.
- Train a model that estimates churn probability for each customer.
- Segment customers into risk groups for retention targeting.
- Produce a clean output file for dashboarding in Power BI.
- Validate whether segments are meaningful against actual churn behavior.

## Recent changes made now (March 31, 2026)
- Split the original single notebook flow into two focused notebooks:
  - `eda_and_customer_split.ipynb` for EDA and customer-type extraction.
  - `repeat_customers_modeling.ipynb` for repeat-customer-only modeling and segmentation.
- Added separate customer logic:
  - One-Time customers are handled as a dedicated group for export and nurture strategy.
  - Repeat customers are used for model training and risk segmentation.
- Implemented revised churn horizon logic in the split workflow:
  - One-Time buyers: churn if `recency_days > 210`.
  - Repeat buyers: churn if `recency_days > 180`.
- Added new exports from the split notebook:
  - `output/one_time_customers.csv`
  - `output/repeat_customers.csv`
- Added repeat-customer segmentation export:
  - `output/repeat_customer_segments.csv`
- Expanded visualization coverage in `repeat_customers_modeling.ipynb`:
  - Repeat class balance and order distribution by churn class.
  - Feature correlation heatmap.
  - Predicted-probability distribution by true class.
  - Decile diagnostics (actual churn vs average predicted probability).
  - Segment profile visuals (size, churn rate, spend, and probability spread).

## Dataset used
Source files are in the `data/` folder:
- `olist_customers_dataset.csv`
- `olist_orders_dataset.csv`
- `olist_order_payments_dataset.csv`
- `olist_order_reviews_dataset.csv`

## Methods and workflow
1. Data preparation and cleaning
- Joined customers, orders, payments, and reviews.
- Filtered to delivered orders.
- Parsed timestamps and removed rows with missing critical fields.

2. Feature engineering (customer level)
- Built one row per customer with behavior features:
  - `total_orders`, `total_spent`, `avg_review`, `min_review`, `pct_low_review`, `avg_payment`, `payment_types`
- Created `recency_days` from the customer last purchase date.

3. Churn label creation
- Churn label: customer inactive for more than `CHURN_THRESHOLD` days.
- Data-driven threshold chosen from repeat-buyer inter-purchase gap percentile.
- Current threshold observed in notebook run: 118 days.

4. Modeling
- Train/test split with stratification.
- RandomForestClassifier used for churn prediction.
- Class imbalance handling added using SMOTE on training data.

5. Segmentation
- Predicted churn probability for all customers.
- Applied StandardScaler to probability scores.
- Used KMeans (`n_clusters=3`) on scaled scores.
- Mapped clusters to ordered labels by centroid rank:
  - Low Risk, Medium Risk, High Risk.

6. Export
- Exported final dataset to:
  - `ecommerce_churn_processed_data.csv`

## Key outcomes (latest run)
- Total customers: 92,746
- Churned customers: 68,497 (73.9%)
- Retained customers: 24,249 (26.1%)

Risk segment distribution:
- Low Risk: 67,785 customers
- Medium Risk: 19,869 customers
- High Risk: 5,092 customers

Actual churn rate by segment:
- Low Risk: 71.2%
- Medium Risk: 79.9%
- High Risk: 86.1%

Lift vs overall churn baseline (73.9%):
- Low Risk: 0.96x
- Medium Risk: 1.08x
- High Risk: 1.17x

## What I learned
- Segmentation quality depends on label quality and class distribution, not only clustering method.
- KMeans can improve customer distribution across buckets, but does not fix an overly severe churn definition.
- Standardizing inputs before KMeans is important even in 1D score-based clustering for consistent behavior.
- Relative risk can be valid even when absolute churn percentages look high.
- Class imbalance handling (SMOTE) is useful in training, but upstream target definition still dominates final outcomes.
- Clear diagnostics (overall baseline, segment lift) are necessary to avoid misleading business interpretation.

## Challenges faced
1. Segment imbalance with threshold-based rules
- Initial fixed cutoffs pushed almost all customers into one segment.
- Action taken: replaced fixed bands with KMeans clustering.

2. High absolute churn in every segment
- Even Low Risk showed high churn percentage.
- Root cause: baseline churn rate in labeled data is very high (73.9%).
- Action taken: added baseline and lift reporting to interpret segments relatively.

3. Class imbalance in model training
- Majority class was churned customers.
- Action taken: introduced SMOTE for balanced training samples.

4. Interpreting risk labels for business users
- Label names can be misunderstood as absolute risk in high-baseline datasets.
- Action taken: documented relative-risk interpretation and segment lift.

## Limitations
- Churn definition is based on inactivity threshold and may be too strict for business use.
- Label policy strongly affects all downstream outputs.
- Model and segmentation are only as good as target definition and historical window design.

## Recommended next improvements
- Revisit churn definition (for example, evaluate 180/210-day horizons).
- Consider separate logic for one-time buyers versus repeat buyers.
- Add calibration checks for predicted probabilities.
- Compare models (XGBoost/LightGBM, calibrated logistic baseline).
- Track retention campaign outcomes to close the loop and improve label realism.

## How to run
1. Run `eda_and_customer_split.ipynb` from top to bottom.
2. Confirm these exports are created in `output/`:
  - `one_time_customers.csv`
  - `repeat_customers.csv`
3. Run `repeat_customers_modeling.ipynb` from top to bottom.
4. Confirm segmented export is created:
  - `repeat_customer_segments.csv`
5. (Optional) Run `ecommerce_churn_analysis.ipynb` if you want the legacy all-in-one pipeline.

## Files in this repository
- `ecommerce_churn_analysis.ipynb`: Full analysis pipeline.
- `eda_and_customer_split.ipynb`: EDA + one-time/repeat customer split and CSV export.
- `repeat_customers_modeling.ipynb`: Repeat-only model training, diagnostics, and segmentation.
- `ecommerce_churn_processed_data.csv`: Processed model-ready output.
- `output/ecommerce_churn_processed_data.csv`: Additional exported output copy.
- `output/one_time_customers.csv`: One-time customer dataset.
- `output/repeat_customers.csv`: Repeat customer dataset for modeling.
- `output/repeat_customer_segments.csv`: Repeat customers with risk segment labels.
- `churn dashboard.pbix`: Power BI dashboard file.

## Final summary
This project successfully built an actionable churn analytics pipeline from raw e-commerce records to model-based risk segmentation and BI output. The main insight is that improving segmentation logic alone is not enough when churn labels are highly imbalanced. The strongest improvement opportunity is now upstream: refine churn target policy so risk labels become more meaningful in absolute business terms.
