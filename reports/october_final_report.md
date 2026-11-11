# October Final Report

## 1. Baseline Metrics

All models are scored on the validation set: the newest 20% of campaigns by start date (82 campaigns).

### CPE

| Model | MAE | RMSE | R² |
|---|---|---|---|
| Predict training mean | $16.99 | 18.93 | −0.03 |
| Product-type lookup table | $0.00 | 0.00 | 1.00 |
| Linear regression | $0.12 | – | 0.999 |
| Ridge (leaky columns removed) | $0.90 | 2.91 | 0.976 |
| Ridge (planning-time features only) | $1.19 | 1.51 | 0.994 |
| Ridge (product features removed) | $8.81 | 11.30 | 0.63 |
| XGBoost (leaky columns removed, tuned) | $0.35 | 3.09 | 0.973 |
| XGBoost (product features removed) | $5.96 | 10.34 | 0.69 |
| XGBoost classifier (3 price tiers) | $0.00 | 0.00 | 1.00 |

### Total spend

| Model | MAE | R² | MAPE |
|---|---|---|---|
| Predict training mean | 1.47 B | −0.28 | 6,620% |
| Linear regression | 2.24 B | −2.97 | 7,073% |
| Target | – | > 0.55 | < 30% |

## 2. Feature Importance

- **Linear regression:** each coefficient is that product type's average CPE in dollars.
- **Ridge and XGBoost:** permutation importance, measured as how much validation MAE goes up when a feature is shuffled.

| Rank | Linear regression coefficient | Ridge (planning-time features) | XGBoost (all features) | XGBoost (product features removed) |
|---|---|---|---|---|
| 1 | BrandSpot: $68.00 | Email Alert: $11.83 | `is_email_alert`: ≈ $16.8 | `engagement_rate`: ≈ $10.1 |
| 2 | Brand Alert, Infosite, Point of Contact, Brand Play, BrandStory, Brand Intercept, Direct Mail: $39.00 | Brand Alert: $3.14 | `is_brandspot`: ≈ $1.0 | `company_MERCK`: ≈ $1.1 |
| 3 | Target List Media: $37.38 | BrandSpot: $1.79 | All others: ≈ $0 | `pre_avg_NRx`: ≈ $0.3 |
| 4 | Condition Message: $11.00 | `start_month`: $1.00 | — | `company_BRISTOL MYERS SQUIBB`: ≈ $0.3 |
| 5 | Email Alert: $3.00 | `start_quarter`: $0.92 | — | `unique_engaged_hcps`, `start_quarter`, `NRx_lift`: < $0.2 each |

## 3. Findings

1. **Product type sets CPE.** Email Alert is $3, BrandSpot is $68, Condition Message is $11, and all other products are $39. A product-type lookup table predicts all 82 validation campaigns exactly.
2. **Compared with the most common price of $39 per engagement:** Email Alert costs 92% less, and BrandSpot costs 74% more.
3. **Linear regression, Ridge and XGBoost all find that product type is the main driver.** No model beats the lookup table.
   - Using only information available before a campaign, Ridge reaches MAE $1.19 and R² 0.994.
   - XGBoost has a lower average error ($0.35) but makes one large miss, and it still uses campaign results such as engagement rate.
4. **Without product type, accuracy drops sharply** (XGBoost R² 0.69, Ridge R² 0.63). The remaining signals mostly act as stand-ins for product type:
   - Engagement rate is 0.40 for $3 campaigns and 0.18 for $39 campaigns.
   - Prescriptions before the campaign are ≈44 for $3 campaigns, ≈71 for $39 campaigns and ≈94 for $68 campaigns.
5. **Some columns leak CPE.** Budget ÷ engagements equals CPE in 99% of rows. These columns were removed:
   - `total_budget_spent`, `log_total_budget_spent`
   - `budget_per_reach`, `budget_per_day`
   - `spend_near_zero`, `budget_anomaly`
   - `cpe_imputed`, `cpe_outlier`, `product_tier`
6. **Total spend is not yet predicted well.** Linear regression does worse than predicting the average, and it predicts negative spend for 20 of the 82 campaigns.
7. **Spend is the main area to work on next.** Modeling `log(total_spend)` with tree-based models, using only information available before the campaign, is the next step toward the targets of R² > 0.55 and MAPE < 30%.
