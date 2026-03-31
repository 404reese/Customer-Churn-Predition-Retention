# Power BI Dashboard Blueprint

This blueprint is built for the files already generated in this project:
- output/repeat_customer_segments.csv
- output/repeat_customers.csv
- output/one_time_customers.csv
- output/ecommerce_churn_processed_data.csv (optional legacy all-customer model output)

## 1) Recommended Power BI data model

Use one primary table first for speed:
- FactRepeat = output/repeat_customer_segments.csv

Optional additional tables:
- FactOneTime = output/one_time_customers.csv
- FactRepeatRaw = output/repeat_customers.csv

If you keep only one table for now, use FactRepeat.

## 2) Data prep in Power Query

For FactRepeat:
- Set data types:
  - customer_unique_id: Text
  - churned: Whole Number
  - churn_probability: Decimal Number
  - churn_probability_pct: Decimal Number
  - risk_segment: Text
  - recency_days, total_orders, payment_types: Whole Number
  - total_spent, avg_payment, avg_review, min_review, pct_low_review: Decimal Number
- Add a custom column (optional) for sorting risk labels:
  - RiskOrder = if [risk_segment] = "Low Risk" then 1 else if [risk_segment] = "Medium Risk" then 2 else if [risk_segment] = "High Risk" then 3 else 99
- In Model view, sort risk_segment by RiskOrder.

## 3) DAX measures (create in FactRepeat)

### Core KPIs
Total Customers = COUNTROWS(FactRepeat)

Churned Customers = CALCULATE(COUNTROWS(FactRepeat), FactRepeat[churned] = 1)

Retained Customers = CALCULATE(COUNTROWS(FactRepeat), FactRepeat[churned] = 0)

Churn Rate % = DIVIDE([Churned Customers], [Total Customers], 0)

Avg Predicted Churn % = AVERAGE(FactRepeat[churn_probability])

### Segment KPIs
High Risk Customers = CALCULATE(COUNTROWS(FactRepeat), FactRepeat[risk_segment] = "High Risk")

Medium Risk Customers = CALCULATE(COUNTROWS(FactRepeat), FactRepeat[risk_segment] = "Medium Risk")

Low Risk Customers = CALCULATE(COUNTROWS(FactRepeat), FactRepeat[risk_segment] = "Low Risk")

High Risk Share % = DIVIDE([High Risk Customers], [Total Customers], 0)

### Value and behavior KPIs
Avg Spend = AVERAGE(FactRepeat[total_spent])

Avg Orders = AVERAGE(FactRepeat[total_orders])

Avg Review = AVERAGE(FactRepeat[avg_review])

### Baseline-relative risk (works by risk segment in visuals)
Segment Actual Churn % = AVERAGE(FactRepeat[churned])

Overall Churn % (Ignore Segment) = CALCULATE([Churn Rate %], ALL(FactRepeat[risk_segment]))

Churn Lift vs Overall = DIVIDE([Segment Actual Churn %], [Overall Churn % (Ignore Segment)], 0)

## 4) Page design (4 pages)

## Page 1: Executive Overview
Goal: Fast status for stakeholders.

Visuals:
- Card: Total Customers
- Card: Churn Rate %
- Card: Avg Predicted Churn %
- Card: High Risk Customers
- Clustered column chart:
  - Axis: risk_segment
  - Values: Count of customer_unique_id
- Clustered column chart:
  - Axis: risk_segment
  - Values: Segment Actual Churn %

Slicers:
- risk_segment
- payment_types (or bin)
- total_orders (as between)

## Page 2: Risk Segmentation Deep Dive
Goal: Compare segment quality and business profile.

Visuals:
- Matrix:
  - Rows: risk_segment
  - Values: Total Customers, Segment Actual Churn %, Avg Predicted Churn %, Churn Lift vs Overall, Avg Spend, Avg Orders, Avg Review
- Bar chart:
  - Axis: risk_segment
  - Values: Avg Spend
- Bar chart:
  - Axis: risk_segment
  - Values: Avg Review
- Scatter:
  - X: avg_review
  - Y: churn_probability
  - Size: total_spent
  - Legend: risk_segment

## Page 3: High-Risk Action List
Goal: Retention operations view.

Visuals:
- Table:
  - customer_unique_id
  - risk_segment
  - churn_probability (format as %)
  - total_orders
  - total_spent
  - avg_review
  - pct_low_review
  - recency_days
- Add visual-level filter:
  - risk_segment = High Risk
- Add Top N filter:
  - Top 500 by churn_probability

## Page 4: One-Time vs Repeat View (optional)
Goal: Separate strategy by customer type.

If both FactOneTime and FactRepeat are loaded:
- Create two cards:
  - Repeat Customer Count = COUNTROWS(FactRepeat)
  - One-Time Customer Count = COUNTROWS(FactOneTime)
- Create two churn cards:
  - Repeat Churn % = AVERAGE(FactRepeat[churned])
  - One-Time Churn % = AVERAGE(FactOneTime[churned])
- Add side-by-side column charts for:
  - churn rate
  - avg_review
  - avg_payment

## 5) Formatting recommendations

- Use consistent colors:
  - Low Risk: #4682B4
  - Medium Risk: #F4A300
  - High Risk: #DC143C
- Use percentage format with 1 decimal for churn measures.
- Turn on data labels for segment charts.
- Keep y-axis start at 0 for churn comparisons.

## 6) Refresh workflow

1. Re-run notebook: eda_and_customer_split.ipynb
2. Re-run notebook: repeat_customers_modeling.ipynb
3. In Power BI Desktop, click Refresh.
4. Validate row counts on KPI cards.

## 7) Minimum viable dashboard (if you want fastest build)

Build these first:
- Card: Total Customers
- Card: Churn Rate %
- Card: High Risk Customers
- Chart: Customers by risk_segment
- Chart: Segment Actual Churn % by risk_segment
- Table: High-risk customer list

That gives an immediately usable retention dashboard in less than 30 minutes.
