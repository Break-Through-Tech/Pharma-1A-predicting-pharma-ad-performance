# Total spend: tuning Linear + Random Forest
**Timothy Hsu** · notebook: `TimothyHsu_Spend_Tuned_Models.ipynb`

**Goal:** predict `total_spend` before a campaign runs. Brief: R² above 0.55, MAPE under 30%.

**Data:** Abhiram's `spend_train_val_split` -> 332 older campaigns to train, 82 newer ones to validate, 72 columns.

**Steps:** re-ran Abhiram's Ridge + Bakari's RF/XGBoost on the same split -> tuned the linear model and the RF with Optuna (time-ordered CV, training rows only) -> scored everything on val.

## Results (validation)

| Model | R² | MAPE | Within 2x | Takeaway |
|---|---|---|---|---|
| Median spend (baseline) | −0.14 | 767% | 16% | floor |
| Ridge (Abhiram) | 0.42 | 106.5% | 50% | starting point |
| RF (Bakari) | 0.56 | 107.0% | 54% | starting point |
| XGBoost (Bakari) | 0.42 | 124.7% | 59% | not tuned |
| **Linear, tuned** | **0.61** | 105.3% | 51% | almost pure Lasso on Abhiram's 14 columns |
| Linear + input search | −0.39 | 120.7% | 44% | best CV, worst val -> overfit |
| **RF, tuned** | **0.62** | **97.9%** | 54% | best overall; smaller trees |

## Takeaways
- **R² goal met:** both tuned models clear 0.55 (linear 0.61, RF 0.62). Up from 0.42 / 0.56.
- **MAPE goal not met:** best is 98% vs the 30% goal. Tuning only took 1–9 points off.
- **Why MAPE is stuck:** models are still ~1.9x off on a typical campaign. Target list + product + size tell you how *big* a campaign is, not why two same-product campaigns spend different amounts -> need new inputs, not more tuning.
- **Small campaigns = most of the MAPE:** 124–152% on $1k–$100M campaigns vs 40–63% on $1B+.
- **Tuned RF uses small trees:** depth 4, each tree sees 58% of rows (Bakari's: unlimited depth, all rows) -> overfits the older campaigns less.
- **Picking inputs by CV backfired:** the input search kept only target list × product -> over-predicted the two ~$7B campaigns by 2x+. 332 rows isn't enough to choose between 512 input combos.

## Issue I hit
- 5 training campaigns with $0–$362 spend were 96% of one CV fold's error -> tuning was basically fitting those 5 rows.
- Fix: CV only scores campaigns with spend >= $1k (same rule as MAPE). They're still used for training.

## Caveats
- This val set has been used across a bunch of our notebooks -> numbers are optimistic. Still need a real H2 test.
- Log-spend predictions aren't corrected when converted back to $ (they lean toward the median, not the mean).

## Next
- Handle the 6 training campaigns under $1k (drop them or model them separately)
- Try a log -> $ correction (Duan smearing)
- Look for inputs that explain spend within a product
