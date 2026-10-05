# Cleaning Decision Log

Dataset: BreakThroughTech_training_data_1.xlsx (417 rows, 28 columns)

Sources: data quality notebooks for variables E to H and J to N, Milestone1.ipynb, BTT.ipynb, and TimothyHsu_Data_Quality_Report.ipynb.

Status key: Applied means the fix already runs in code on df_clean. Agreed means the fix is proposed in a notebook and still needs to be added to the cleaning script. No change means we checked the column and left it as is on purpose.

## Dropped columns

- campaign_end_date: Drop the column and do not derive campaign duration from it. 414 of 417 rows hold the same placeholder (2099-12-31 23:59:59 UTC) and the other 3 are null, so 0 rows have a real end date. A naive end minus start duration averages about 27,583 days, which is noise from the placeholder. (Agreed)
- specialty: Drop the column and use npi_specialty instead. Only 54 rows (12.9%) hold a usable value. The rest are "undefined" (295), null (57), or unexplained numeric codes (11). npi_specialty has 0 nulls and 13 standardized categories after the fix below. (Agreed)

## Missing values

- campaign_start_date (3 nulls, 0.7%): Drop the 3 rows. Too few to impute reliably. The same rows (170, 183, 282) are also null in campaign_end_date and total_spend, so one drop clears three columns. (Agreed)
- media_type (80 nulls, 19.2%): Recode null to "UNKNOWN". The gaps are not random: 56 of the 80 rows are also missing profession, specialty, country and pillar, which points to a batch that never captured targeting metadata. Filling with the mode would invent targeting information. (Agreed)
- profession (56 nulls, 13.4%): Recode null to "UNKNOWN". Same 56-row metadata block. An explicit level lets the model learn whether that block behaves differently. (Agreed)
- Reach, engagement, budget, NRx and spend columns: No change. All are numeric with no nulls, except total_spend (3 nulls, cleared by the row drop above) and cpe (93 nulls, see open questions). (No change)

## Data types and encoding

- profession: Treat numeric codes as categorical, not ordinal. Keep each code (8, 99, 10 to 23) as its own level, separate from the text labels, and cast the column to string before one-hot encoding. There is no natural ordering between codes and no codebook maps them to names. 296 rows use a numeric code and 65 use free text, so merging them now would be a guess. Casting to string stops 8 from being read as a number next to "psychologist". (Agreed)
- campaign_start_date: Convert from text to datetime64 (UTC). The format is consistent and 0 non-null values fail to parse. A real date type allows campaign age and month, quarter and year features. (Agreed)
- npi_specialty: Recode "3320 - Anesthesiologists" to "Anesthesiology" (1 row, index 136). Every other value is a plain specialty name, so the coded label would form its own one-row category. (Applied)
- region: Recode "West South Central" to "Southwest" (1 row, index 136). It is the only row with that label, while the other 9 regions have 35 to 64 rows each. Southwest is the closest existing category. (Applied)

## Invalid values, outliers and duplicates

- post_avg_NRx: Cap 20 values above Q3 + 1.5 x IQR at 319.31. Keeps the rows while stopping a few extreme values from dominating the fit. Dropping them would cost about 5% of the data. (Applied)
- pre_NRx_avg: Cap 14 values above Q3 + 1.5 x IQR at 232.60. Same reasoning as post_avg_NRx. (Applied)
- NRx_lift: Recompute as post_NRx minus pre_NRx, rounded to 2 decimals. 106 rows differed from that formula by one cent, and a derived column should match its definition exactly. (Applied)
- total_budget_spent: Keep the 2 zero-spend rows (146 Janssen Biotech, 173 Lilly) and flag them. Do not impute. Both show real reach and hundreds of engagements, so zero spend may be an entry error, but we cannot confirm it yet. The zero also carries into total_spend, actual_spend_per_engagement and spend_per_NRx_lift for those rows. (Agreed)
- unique_reach, unique_engaged_hcps, total_engagements, total_budget_spent: Apply log1p for linear models and keep all rows. All four are strongly right-skewed. log1p handles the skew and works on the two zero budgets. (Agreed)
- NRx_lift_pct, spend_per_NRx_lift: No change. IQR flags 8 and 49 outliers, but there are no negative, infinite or out-of-range values. The high values look like real large campaigns, not errors. (No change)
- engagement_rate, engagement_frequency: No change. Both match their formulas (engaged HCPs over reach, engagements over engaged HCPs) with 0 mismatches at stored precision. (No change)
- All rows: No deduplication. There are 0 fully duplicated rows. (No change)

## Open questions

These need a team decision or input from the Challenge Advisor before the cleaning script is final. The last four came from a full pass over the raw file and are not yet in any notebook.

- Profession codebook: Ask the Challenge Advisor for a mapping of codes 8, 99 and 10 to 23. If one exists, merge codes with the matching text labels.
- Zero-spend rows: Confirm whether rows 146 and 173 really had $0 spend. If not, set spend to null and decide whether to drop or impute.
- 56-row metadata block: Keep it with "UNKNOWN" levels, or drop it? Dropping removes 13.4% of rows.
- country and pillar nulls: Both have 56 nulls in the same block. Recoding to "UNKNOWN" would match media_type and profession.
- cpe nulls (93): cpe is a fixed price per sub_product_name (Brand Alert 39, Email Alert 3, Brandspot 68, Condition Message 11) and is null for 7 other product types. The null likely means "no fixed price", not missing data. Options are to drop cpe since it duplicates sub_product_name, or fill with 0 and add a flag.
- brand labels: 73 raw labels include the same company spelled several ways (BMS and BRISTOL-MYERS SQUIBB, BOEHRINGER INGELHEIM and BOEHRINGER-INGELHEIM) and product suffixes (LILLY_MOUNJARO_MEDSCAPE). Standardize to the parent company?
- pre_avg_NRx vs pre_NRx_avg: Two columns with near-identical names that never match and correlate at 0.78. Which definition is correct, and should one be dropped?
- total_spend scale: Median total_spend is about 1,800 times total_budget_spent. Check the units before using it or the two columns derived from it.
