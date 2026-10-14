# Milestone 2, Task 2: Encoding and Train/Validation Split

## The goal

Models can only read numbers, not words. This task had two parts:

1. Change the text columns into number columns.
2. Split the data into a training set (for the models to learn from) and a validation set (for checking how well they learned).

## Starting file

`featured_df.csv` from Task 1. It has 417 campaigns (rows) and 60 columns. The column we want to predict is `cpe` (cost per engagement).

## Step 1: Split the data by date

- Sorted the campaigns from oldest to newest by start date.
- The oldest 80% went into training and the newest 20% went into validation.
- Campaigns that started on the same day were kept together in the same set.

Splitting by date means the models are checked on newer campaigns they have never seen, which is closer to how they would be used in real life.

| Set | Campaigns | Start dates |
|---|---|---|
| Training | 335 | Dec 17, 2020 to Mar 11, 2025 |
| Validation | 82 | Mar 12, 2025 to Apr 17, 2026 |

## Step 2: Change the text columns into numbers

There are 14 text columns, such as `country`, `company` and `media_type`. Each one was changed using one-hot encoding.

One-hot encoding gives every category its own column with a 1 or a 0. For example, the `media_type` column becomes three columns:

| media_type | media_type_editorial_videos | media_type_sponsored_videos | media_type_unknown |
|---|---|---|---|
| editorial_videos | 1 | 0 | 0 |
| sponsored_videos | 0 | 1 | 0 |
| unknown | 0 | 0 | 1 |

A few rules were followed:

- The encoding was learned from the training set only, then applied to both sets. This keeps the validation set fully unseen.
- Categories that show up fewer than 5 times in training were grouped into one shared "infrequent" column. This stops rare values from creating lots of nearly empty columns.
- If a category shows up only in validation, it goes into that "infrequent" column. If there is no "infrequent" column, the row gets all 0s for that column.

The 14 text columns became 117 number columns.

| Text column | Number of categories |
|---|---|
| brand_raw | 73 |
| company | 34 |
| product | 23 |
| profession | 14 |
| npi_specialty | 14 |
| sub_product_name | 11 |
| region | 9 |
| country | 9 |
| specialty | 9 |
| country_name | 9 |
| channel_tag | 8 |
| pillar | 5 |
| product_tier | 4 |
| media_type | 3 |

The 44 columns that were already numbers were left as they are. The `campaign_start_date` column was only used to make the split and is not in the final files. The year, quarter and month columns from Task 1 still carry that timing information.

## Step 3: Check the results

- No missing values
- Every column is a number
- Training and validation have the exact same columns
- The `cpe` column is kept separate from the other columns
- All 417 campaigns are accounted for

## Final files

Each set has 161 columns: 44 original number columns plus 117 new ones.

| File | What is inside |
|---|---|
| `notebooks/04_encoding_and_split.ipynb` | The code for this task |
| `data/train_val_split/X_train.csv` | Training data (335 rows, 161 columns) |
| `data/train_val_split/X_val.csv` | Validation data (82 rows, 161 columns) |
| `data/train_val_split/y_train.csv` | `cpe` values for training (335 rows) |
| `data/train_val_split/y_val.csv` | `cpe` values for validation (82 rows) |

## How to load the files

```python
import pandas as pd

S = "data/train_val_split/"
X_train = pd.read_csv(S + "X_train.csv")
X_val = pd.read_csv(S + "X_val.csv")
y_train = pd.read_csv(S + "y_train.csv")["cpe"]
y_val = pd.read_csv(S + "y_val.csv")["cpe"]
```

## Requirements

The notebook needs `pandas` and `scikit-learn`.
