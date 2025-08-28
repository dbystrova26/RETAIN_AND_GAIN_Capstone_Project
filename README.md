
# RETAIN AND GAIN (Capstrone Project)

Using the 2019 Kaggle “Marketing Insights for E-commerce” data, enriched with Calendarific holidays, we estimate CLV with BG-NBD, Pareto/NBD, MBG-NBD, and a Gamma-Gamma spend model on RFM. Seasonality (month + holidays) is built into the CLV pipelines and the promo incrementality read to steer ROAS-positive budget decisions.

Purpose of this case study

 The project investigates methods applicable to non-contractual, continuous purchasing businesses to (i) estimate the potential revenue and value of active customers (CLV), (ii) quantify the incremental impact of discount programs on revenue and profit, and (iii) help to plan seasonal campaigns.

Data and context

The analysis uses transactional panel data enriched with product category, coupon activity, and marketing spend. Monthly aggregates are constructed to support diagnostics, treatment–control comparisons, and executive dashboards, while customer-level panels are preserved for lifetime value modeling and segmentation.

Primary dataset

Marketing Insights for E-Commerce Company (Kaggle). This curated dataset contains the core entities required for end-to-end marketing analytics:
Transactions / Sales lines (invoice-line grain):

 transaction_date, transaction_id, customer_id, produkt_sku, product_category, quantity, avg_price, delivery_charges, coupon_status, discount_pct, revenue 

Customer attributes:

 customer_id, gender, location, tenure_months.

Discount reference:
 month, product_category, coupon_code, discount_pct.

Tax table (GST):
 product_category, gst.


Marketing spend (monthly):
 date, offline_spend, online_spend.
 We normalize column naming and units (see “Engineering & QC” below).


From these, we also build a conformed mart (e.g., mart_all_data) that joins sales, customers, marketing spend (as day-level context), and holiday features.
Feature enrichment: Calendar
Calendarific API (2019) – U.S. holiday calendar for feature engineering. Fields include:
name, description, date_iso, type.
 We derive boolean flags (is_holiday) and one-hot encodings for holiday types used in retention/CLV segmentation and in uplift diagnostics.
Engineering & data quality controls
Date normalization: unify to YYYY-MM-DD and derive month keys as DATE (first-of-month) to ensure Tableau relationships (avoid mixed “2019-01” strings).


Spend units: confirm coupon/marketing spend in base currency units (not millions). Correct scaling errors before computing profit/ROAS.


Sign conventions: treat coupon budget and discounts as positive costs (subtract from profit).


De-duplication & validity: keep quantity > 0, non-negative prices, and drop null customer_id rows where appropriate.

Coupon normalization (treatment flag): we treat coupon_status = 'Clicked' as equivalent to Used. In other words, treated = 1 if coupon_status ∈ {'Used','Clicked'} and control = 0 otherwise. Discount/coupon costs and any treatment-based metrics (incremental revenue, ATT, ROAS) use this convention.


Methodological approach
Probabilistic CLV modeling. Classical buy-till-you-die models—Pareto-NBD, BG-NBD, MBG-NBD—are fit to repeat-purchase behavior; Gamma-Gamma is used to model spend conditional on purchase. These models yield forward-looking CLV distributions and cohort-level revenue forecasts.

Incrementality measurement. To estimate causal lift from discounts, we implement difference-in-differences (two-way fixed effects), reporting ATT with confidence intervals, discount dose–response (β with CI), and monthly incremental revenue with a cumulative view.


Diagnostics and financial synthesis. We construct treated vs control mean revenue traces, compute net profit after coupons, and reconcile against coupon budget to show ROI/ROAS at month and category levels.


Unsupervised segmentation. Clustering on behavioral and value features creates actionable customer segments that differ in purchase frequency, monetary value, price sensitivity, and inferred lifetime value.


Visualization and delivery. Reusable SQL tables power Tableau dashboards


Key insights 
CLV models provide stable, interpretable forecasts for non-contractual customers and identify high-value cohorts for retention and cross-sell.

Discount effectiveness is heterogeneous: monthly lift varies and cumulative impact can diverge from point estimates—making the dose–response and confidence bands essential for decisioning.

Category-level views highlight where coupons create profit versus where they erode margin, guiding budget reallocation.


Segments exhibit distinct elasticity and value profiles, enabling targeted promo intensity rather than uniform discounts.


Practical outcomes
A reproducible analytics pipeline (data prep → modeling → causal estimation → visualization).


A set of database tables engineered for BI consumption (EDA, cohort analysis, diagnostics, ATT, dose–response, incremental revenue, pCLV and Seasonality dashboards) and Tableau workbooks that translate results into decision support for finance and marketing.


Limitations and assumptions
Identification relies on parallel-trends and correct treatment assignment; remaining selection or timing bias may persist.
ROAS at the category level depends on an allocation rule for shared coupon spend.
Results reflect the observed horizon and available features; exogenous shocks and unobserved media may confound effects.
For analysis, coupons marked 'Clicked' are treated as 'Used' (i.e., included in the treated group and counted toward realized discount/coupon spend).


Implications and next steps

Use the segment × category heatmap to reallocate coupons toward profitable pockets and away from margin-destroying ones.


Operationalize CLV-based targeting (thresholds or percentiles) for acquisition and retention campaigns.


Extend the causal framework with alternative estimators (e.g., CUPED, synthetic controls) and run prospective hold-outs to validate lift.


Automate the pipeline for monthly refresh and add lifetime ROAS tracking that ties spend to modeled CLV, not only short-run revenue.

In sum, the study demonstrates an integrated approach—combining probabilistic CLV modeling, causal incrementality, and unsupervised segmentation—to quantify customer value and the true financial impact of discounting in a non-contractual retail setting, delivering both strategic insight and operational tooling.


>>>>>>> capstrone-local/main
