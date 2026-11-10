# Training dataset pipeline

Source: `sql/stolencard_dataset_pipeline.sql` (DataWorks, ODPS SQL).
Output: `BaseHub_mx_dev.stolencard_dataset_all_tables` (one row per card transaction).

## Population
Card payment events from `BaseHub_mx.dwd_risk_event_payment_details_di`
(`pay_option LIKE '%CARD%'`), deduplicated per `(user_id, trans_id)` (latest row).
Each transaction is tagged `sample_type`: `training` or `oot` (out-of-time) by its UTC timestamp.
Users present in training are removed from OOT so no user appears in both samples.

## Label (`fraud_label` = 1 if any of):
- **Dispute / chargeback** with reason code 68 or 41
  (`dwd_controversy_dispute_miclip_match_df`)
- **Dispute amount > 10%** of the user's running card amount (user-level flag)
- **Blacklists** (`ods_risk_variable_black_list`), only if the transaction happened
  after the blacklist time: `CS_Black_Userid` (stolen card), `ATO_Black_UserId`
  (account takeover), `Suspicious_userId` with amount > 2000 (high severity)

## Features (point-in-time joins)
Each block takes the latest snapshot strictly **before** the transaction date
(`transaction_mexico_time > dt`) to avoid leakage.

| # | Block | Source |
|---|-------|--------|
| 1 | Demographics | `dws_risk_mbr_role_user_base_info_dd` |
| 2 | Wallet business scenarios | `dws_wallet_txn_basic_feature_user_di` |
| 3 | Login | `dws_wallet_login_basic_feature_user_di` |
| 4 | Merchant details | `ods_cu_role_merchant` |
| 5 | Merchant transactions | `dws_risk_merchant_mcc_txn_basic_dd` |
| 6 | Online features | `app_risk_profile_variable_profile_df` |
| 7 | Risk community (graph) | `dwd_risk_card_stolen_riskgroup_feature_v01` |
| 9 | Card BIN profile | `dwd_risk_card_stolen_cardbin_profile_payment_v01` |

Block 8 (card classification) is commented out in the SQL. Blocks 10-13 (user attributes,
transaction preference, home location, business scenario) were prototyped and disabled
in the original DataWorks node and are not included in this file.

## Known issues / to review
- Block 6 joins on `MAX_PT(...)` (latest partition), not a point-in-time snapshot:
  possible leakage for historical samples.
- The `INSERT` column order does not match the `CREATE TABLE` order for the date
  columns (`mexico_city_time` is selected before `transaction_time`).
- Output includes `name` (PII); consider dropping it before exporting data.
