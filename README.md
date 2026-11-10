# Stolen Card Model

Anti-fraud predictive model to identify **stolen card transactions** (card top-ups)
in the Mexico wallet. The training dataset is built in DataWorks (MaxCompute) and
explored/modeled in notebooks.

## Repository layout

| Path | What it is |
|------|------------|
| `sql/stolencard_dataset_pipeline.sql` | DataWorks ODPS SQL that builds `BaseHub_mx_dev.stolencard_dataset_all_tables` |
| `notebooks/stolencard_txn_samples.ipynb` | Sampling / inspection of labeled transactions |
| `notebooks/stolencard_dataset.ipynb` | Exploration of the training dataset |
| `notebooks/lag_dispute.ipynb` | Lag between transaction and dispute/chargeback |
| `docs/pipeline.md` | Pipeline documentation (labels, windows, feature blocks) |

## Setup

Notebooks read MaxCompute credentials from a local `.env` (not committed):

```
ODPS_ACCESS_ID=...
ODPS_ACCESS_KEY=...
ODPS_PROJECT=...
ODPS_ENDPOINT=...
```

Notebooks are committed **without outputs** (they can contain customer data).
Clear outputs before committing.

## Building the dataset

Run `sql/stolencard_dataset_pipeline.sql` in DataWorks with these parameters:

| Parameter | Format | Example |
|-----------|--------|---------|
| `train_start` / `train_end` | `yyyy-MM-dd` | 2026-07-01 / 2026-08-28 |
| `oot_start` / `oot_end` | `yyyy-MM-dd` | 2026-08-28 / 2026-10-01 |
| `lookback_dt` | partition bound, must be <= earliest window start | 2026-05-01 |

See `docs/pipeline.md` for details.
