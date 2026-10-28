# BGP Hijacking 90-Day Prefix-Origination Dataset (Indexed)

Every record in this directory is identified by a single integer `id`. The
original identifiers (AS number and window start/end) have been mapped to
sequential IDs, so no AS numbers or dates appear in the data files.

## Overview

| | `hijacker` | `legal` |
|---|---|---|
| Images | 11,758 | 6,240 |
| Feature rows | 13,123 | 5,903 |
| Records with both image and row | 11,406 | 5,821 |

## Layout

```
.
├── hijacker_charts/          # 11,758 files, named <id>.png
├── legal_charts/             #  6,240 files, named <id>.png
├── hijack_struc.csv          # 13,123 rows × 30 columns
├── legal_struc.csv           #  5,903 rows × 30 columns
├── README.md                 # this file
└── README.zh-CN.md           # 简体中文
```

## ID rules

- `id` is an integer from `1` to `19797`, unique across both classes;
- image files use the zero-padded six-digit `id`, e.g. `000001.png`, matching
  the `id` column in the CSVs;
- 17,227 records have both a CSV row and an image; 1,799 rows have no image;
  771 images have no matching row;
- CSVs are sorted by `id`.

## Schema

Both CSVs share the same 30 columns: `id` plus 29 features. Count-like columns
are stored as `ln(1 + x)`.

### Activity and churn

| Column | Description |
|---|---|
| `appear_share` | share of all prefix-appearance records in the window |
| `insert_ratio` | announcements ÷ (announcements + withdrawals) |
| `remove_ratio` | withdrawals ÷ (announcements + withdrawals) |
| `turnover` | overall churn volume, `ln(1 + insert_cnt + remove_cnt)` |
| `prefix_coverage_ratio_for_as` | prefixes as a share of all prefixes observed |
| `adj_change_ratio` | share of changes that involve adjacent prefixes |

### Position shift

| Column | Description |
|---|---|
| `pos_shift_mean` | mean signed shift in position/rank over the window |
| `pos_shift_std` | standard deviation of that shift |

### Short-lived announcements

| Column | Description |
|---|---|
| `short_lived_insert_ratio` | share of announcements classified as short-lived |
| `short_lived_remove_ratio` | share of withdrawals classified as short-lived |

### Counts (`ln(1 + x)`)

| Column | Description |
|---|---|
| `appear_cnt` | prefix-appearance records |
| `insert_cnt` | prefix announcements |
| `remove_cnt` | prefix withdrawals |
| `persist_cnt` | prefixes present throughout the window |
| `short_ins_cnt` | short-lived announcements |
| `short_rem_cnt` | short-lived withdrawals |
| `total_events` | all BGP events observed in the window |
| `total_prefix_cnt` | all distinct prefixes observed in the window |

### MOAS features (multi-origin AS)

| Column | Description |
|---|---|
| `k_used` | `k` parameter used for the MOAS features (constant `3`) |
| `moas_event_cnt` | MOAS-related events, `ln(1 + x)` |
| `moas_prefix_cnt` | prefixes involved in MOAS, `ln(1 + x)` |
| `moas_days_active` | days with MOAS activity, `ln(1 + x)` |
| `moas_days_active_ratio` | active days ÷ 90 |
| `moas_multi_origin_ratio_k` | fraction of prefixes with more than `k` origins |
| `moas_more_specific_ratio` | fraction of MOAS prefixes announced as more-specifics |
| `moas_origin_cnt_mean` | mean number of origins per MOAS prefix |
| `moas_prefix_run_max` | longest run of consecutive prefixes announced together |
| `moas_top_peer_share` | share of MOAS events from the dominant peer/collector |
| `moas_peer_entropy` | entropy of the peer/collector distribution |

## Loading

```python
import pandas as pd

df = pd.read_csv("hijack_struc.csv")
df["image_path"] = df["id"].map(lambda i: f"hijacker_charts/{i:06d}.png")
print(df.shape)
```

Not every row has an image; check that the file exists before using it.

## Restoring counts

Count columns are `ln(1 + x)`; use `numpy.expm1()` to recover the raw counts:

```python
import numpy as np
raw = np.expm1(df[["appear_cnt", "insert_cnt", "remove_cnt", "turnover"]])
```

## Suggested splits

IDs are sequential, so contiguous `id` ranges can be used for train/test
splits; for class balance, sample `id` ranges within each class.

## Data source

**TODO:** add the collectors and archives, label source, extraction period and
preprocessing code.

## License

**TODO:** add a `LICENSE` file.

## Citation

**TODO:** add the paper or dataset citation.
