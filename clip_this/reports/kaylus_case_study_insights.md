# Kaylus Case Study CSV Insights

## Source
- File: `data/01_raw/kaylus_case_study.csv`
- Snapshot date analyzed: 2026-03-31

## Executive Summary
The dataset appears to be an event-level table where each `document_id` represents a session and each row is likely a gesture event within that session. Session-level fields are repeated across rows, while gesture telemetry fields vary. Several score columns are constant and likely not useful as predictive features in this file.

## Dataset Profile
- Rows: 125
- Columns: 32
- Unique `document_id`: 26
- Unique `userID`: 4
- Timestamp range (`timestamp`): 2026-03-13 04:36:09.730191+00:00 to 2026-03-19 06:02:36.400895+00:00
- Rows per `document_id`: min 1, max 16, mean 4.81

## Header-Level Interpretation
The headers naturally split into four groups:

### 1) Session Identity and Context
- `document_id`, `timestamp`, `userID`
- `sessionStart`, `sessionDelta`
- `city`, `region`, `country`, `gps_lat`, `gps_lng`

Interpretation: session identifiers and context metadata. `timestamp` likely reflects ingest/session event time, while `sessionStart` is the canonical session start.

### 2) Risk/Heuristic Score Features
- `inquisitorScore`
- `plaid_score`, `imei_score`, `blacklist_score`, `duplicate_score`
- `manual_refresh_score`, `geo_checkin_score`, `offer_redeem_score`
- `gesture_complexity_score`

Interpretation: score-engine outputs and at least one derived/aggregate signal.

### 3) Behavioral Aggregates
- `dailyClippedOffers`, `dailyPotentialSavings`, `sessionClips`

Interpretation: user/session behavior counters suitable as aggregate-level features.

### 4) Gesture Telemetry (Event-Level)
- `gesture_timestamp`
- `gesture_vx`, `gesture_vy`, `gesture_combinedVelocity`
- `gestureState_moveX`, `gestureState_moveY`
- `gestureState_dx`, `gestureState_dy`
- `gestureState_vx`, `gestureState_vy`

Interpretation: per-event kinematic traces and transformed movement states.

## Naming Quality Notes
- Mixed naming style exists:
  - Snake case: most columns (good).
  - Mixed/camel fragments: `userID`, `gestureState_*`.
- Potentially ambiguous time fields:
  - `timestamp` vs `sessionStart` vs `gesture_timestamp` would benefit from explicit semantic suffixes (for example `_utc` or `_epoch_ms`).
- Potential overlap/redundancy:
  - `gesture_vx`/`gesture_vy` and `gestureState_vx`/`gestureState_vy` may represent different stages of processing, but naming alone does not make this obvious.

## Missingness and Data Quality
- Missing values:
  - `city`, `region`, `country`: 33 missing each (26.4%)
  - `sessionDelta`: 74 missing (59.2%)
  - `gestureState_moveX`, `gestureState_moveY`, `gestureState_dx`, `gestureState_dy`, `gestureState_vx`, `gestureState_vy`: 99 missing each (79.2%)
- One user appears to have systematically missing location fields, suggesting a collection-source or permission pattern rather than random missingness.

## Distribution and Coverage
- Row contribution by user (`userID`):
  - `PcUwDMTrEXS8q0IrRPSstSSwZs82`: 70
  - `0HCVK54voidwNOOUHnwTXBI7Asp2`: 39
  - `ZsUMsUrFKeXqMqYQaM4WuHVXjSl2`: 11
  - `XfULUQGqgBV7uQSNqS3zyZgpFDY2`: 5

Observation: user representation is highly imbalanced.

## Non-Informative or Derived Fields
### Constant columns (single value in all rows)
- `plaid_score`
- `imei_score`
- `blacklist_score`
- `duplicate_score`
- `manual_refresh_score`
- `geo_checkin_score`
- `offer_redeem_score`

These provide no variance within this extract.

### Deterministic relationship
`inquisitorScore` is effectively a linear transform of `gesture_complexity_score` in this dataset:

- Fitted slope: 0.05000000002209166
- Fitted intercept: 22.090909089873062
- R^2: 1.0

Equivalent practical rule:

`inquisitorScore ≈ 22.090909 + 0.05 * gesture_complexity_score`

Implication: keeping both in modeling can introduce direct redundancy/leakage of engineered signal.

## Practical Recommendations
1. Separate grain before analysis:
   - Session table keyed by `document_id`
   - Gesture-event table keyed by (`document_id`, event index or `gesture_timestamp`)
2. Exclude constant columns from baseline models.
3. Keep either `inquisitorScore` or `gesture_complexity_score` initially, not both.
4. Treat location and `sessionDelta` missingness as informative (missing-not-at-random candidate).
5. Standardize naming conventions before wider pipeline use:
   - unify `userID` to `user_id`
   - clarify `gestureState_*` naming intent
   - normalize timestamp suffix conventions

## Full Header List (as observed)
`document_id, timestamp, userID, inquisitorScore, sessionStart, sessionDelta, city, region, country, gps_lat, gps_lng, plaid_score, imei_score, blacklist_score, duplicate_score, manual_refresh_score, geo_checkin_score, gesture_complexity_score, offer_redeem_score, dailyClippedOffers, dailyPotentialSavings, sessionClips, gesture_timestamp, gesture_vx, gesture_vy, gesture_combinedVelocity, gestureState_moveX, gestureState_moveY, gestureState_dx, gestureState_dy, gestureState_vx, gestureState_vy`
