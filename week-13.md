# Week 13 (2026-10-26 ~ 2026-10-30)
---

## What I did this week
**10/26**
- Saw that only 43.5% of trips had rail events in Step 2 and decided the problem was in the data itself, not in the model
- Found that the rail car master table in the old merged structure (CN/CP merged data + City Master coordinates + rail car master) was not accurate, so I replaced it completely
- Changed the structure from rail-car-based to VIN-based
  - Took the first origin and final destination of each VIN from the internal VIN movement data
  - One rail car carries many VINs, so grouping VINs by rail car gives more accurate routes than before
- Merged routes that were split at an interchange hub into one route (e.g., A→Hub + Hub→B became A→B). Only one hub needed this rule
  - ShipOut = earliest ship-out date of the same VIN
  - Delivered = latest delivered date of the same VIN
- Set up `VIN Movement.xlsx` with two sheets
  - `Data`: raw export from the internal system
  - `VIN_Org_Dest`: cleaned route per VIN, with 8 fixed columns (VIN, EquipmentId, Origin, Destination, ShipOut, Delivered, ShipmentKey, EquipmentStatus)
  - EquipmentStatus is OK when the other 7 columns are filled, and Missing when any of them is null

**10/27**
- Organized `Merged.xlsx` into two sheets
  - `ver1CNCP_Raw`: daily CN/CP data collected by Power Automate
  - `ver1CNCP_Clean`: preprocessed Raw data + City Master latitude/longitude
- Tried to merge `ver1CNCP_Clean` with `VIN_Org_Dest` like the old structure, but the data was too large and the file overloaded
- Merging also links the two files, so they can no longer be kept as independent data → decided not to merge
- Fixed the final input for the ETA model as three independent files
  - `Merged.xlsx` [ver1CNCP_Clean]
  - `Route_Distance_Integrated.xlsx` (all sheets)
  - `VIN Movement.xlsx` (all sheets)

**10/28**
- Ran the ETA pipeline from Step 0 to Step 4-3
- **Step 0 (input check):** kept VIN shipments, CN/CP events, and standard routes as separate inputs and joined them in Python. Kept leading zeros in EquipmentId, handled wrong dates, and set time cutoffs so future data does not leak into training
- **Step 1 (shipment units):** turned about 414K cleaned VIN rows into about 44K rail car shipments (key: EquipmentId + Origin + Destination + ShipOut)
  - Split by time into Train / Validation / Test, plus censored and still-open shipments
  - Kept route nodes with the same city name but a different NodeSequence as different locations (the same city appears on two carriers' segments)
- **Step 2 (events and progress):** linked CN/CP events to shipments and made one observation per day (progress %, current node, km traveled, km remaining, days since departure, number of valid events, carrier ETA, actual days left)
  - 94.9% of linked events matched a route node
  - Fixed backward progress (0.9% of observations) by keeping the previous position
  - Removed events at a repeated city when the city name alone could not tell which position it was
- **Step 3 (Stage 1, ETA at departure):** compared B1, B1_BIAS, and GBM → chose **B1**
  - B1 = median actual lead time of the same O-D in the last 60 days, with a 6-level fallback (same O-D all history → same origin → all routes → fixed 14 days)
  - 95.2% of shipments were predicted at the first level
  - Test: MAE 2.13 days, ±3 days hit rate 76.2%, bias −1.54 days (predicts arrival too early)
- **Step 4 (Stage 2, ETA in transit):** compared R0, R2, R2N, R3, GBM, and BEST_BY_BIN (best method per progress bin on validation)
  - BEST_BY_BIN was best on validation, but single R2 was more stable on test → went back to review the GBM features
- **Step 4-1 (feature search):** single features → ablation → promising sets → 511 combinations of 9 shortlisted features
  - Final M2_FINAL features: R0, n_match (valid matched events), remaining_km
  - Chose the simpler 3-feature model because it was only 0.0025 days (about 4 minutes) worse than the best one on validation
  - Test: MAE 1.56 days, bias almost 0, ±3 days 86.9%, ±5 days 96.2%
  - About 6.3 hours better than the old GBM and about 3.4 hours better than R2
- **Step 4-2 (operating rule):** set a minimum improvement of 0.10 days before switching methods by bin. R0 was only about 25 minutes better in the 0–25% bin on validation and much worse on test → use **M2_FINAL at every progress level when valid events exist**
- **Step 4-3 (vs. carrier ETA):** on the same test observations that have a carrier ETA
  - M2_FINAL MAE 1.44 days vs. carrier ETA 2.20 days (about 18 hours better)
  - M2_FINAL was closer in about 6,000 observations, the carrier in about 2,700
- Final ETA structure
  1. At departure: Initial ETA = ShipOut + Stage 1 (B1) lead time
  2. No valid events yet: R0 = max(Stage 1 lead time − days elapsed, 0) → keep the initial ETA
  3. Valid events: predict remaining days with M2_FINAL → Current ETA = observation date + remaining days

**10/29**
- Added data up to 2026-10-29 and re-ran everything from Step 0 to Step 4-3 with the same steps to decide the final model
- M2_FINAL features changed after retraining
  - 10/28: R0, n_match, remaining_km
  - 10/29 (final): progress_gap, carrier_vs_r0, expected_progress, remaining_km
- Built the Step 5 prediction engine and test code
  - `eta_prediction_engine.py`: one engine that combines Stage 1 and Stage 2
  - `step5_validate_engine.py`: checks that the engine works → passed
  - `step5_predict_eta.py`: predicts ETA for shipments with no Delivered date (real operation targets)
  - `step5_validate_case.py`: predicts ETA for delivered shipments and compares it with the actual Delivered date
- Input for both scripts: EquipmentId, Origin, Destination, ShipOut, AsOfDate
  - The engine only uses data between ShipOut and AsOfDate, and treats anything after AsOfDate as unknown

**10/30**
- Built a validation dataset from every case that `step5_validate_case.py` can handle (`validation_input`)
- Predicted ETA for all cases with the model (`validation_output`) and saved both to Excel
- Started building the system: Vue.js frontend + FastAPI backend that calls my Python prediction engine

## What I learned (skills/concepts)
- When the result looks wrong, check the data first. The low event coverage came from an inaccurate master table, not from the model
- Choosing the right data grain (VIN vs. rail car) changes how accurate the routes are
- Keeping input files independent and joining them in Python is lighter and easier to maintain than merging them in Excel
- Time-based splits and cutoffs to prevent data leakage, and how to handle censored and still-open shipments
- A fallback order makes a simple baseline usable even for routes with little history
- Feature selection with ablation and combination search, and choosing the simpler model when the gap is very small
- A minimum improvement threshold stops me from switching methods because of tiny validation differences that do not hold on test
- A fair comparison with the carrier ETA needs the same sample on both sides
- Point-in-time prediction (AsOfDate): the model should only see what was known on that date
- Connecting a Python model to a web system with FastAPI and Vue.js

## Challenges / problem-solving
- **Low event coverage (43.5%):** replaced the rail car master table with a VIN-based structure built from the internal VIN movement data
- **Routes split at an interchange hub:** merged the two legs into one route using the earliest ShipOut and latest Delivered of the same VIN
- **File overload when merging:** stopped merging and kept three independent input files
- **Same city name at different route positions:** kept nodes separate by NodeSequence, and removed events that could not be placed by city name alone
- **Backward progress:** kept the previous position instead of moving backward
- **BEST_BY_BIN won on validation but not on test:** went back to GBM feature engineering instead of using the per-bin mix, which led to M2_FINAL

## Next week plan
- Compare the Step 5 engine's predictions with the carrier ETA on the same cases
- Review `validation_output` to see where the model is far off
- Continue the Vue.js + FastAPI system that calls the prediction engine

## Reflection
- I did not stop at the first GBM result. I used feature engineering to find the best GBM, compared it again with R0, R2, R2N, and R3, and got M2_FINAL. Before feature engineering, BEST_BY_BIN used a different model for each progress bin; after it, one model worked for every bin
- Fixing the data structure first (rail car → VIN) made the rest of the pipeline possible. This week moved from cleaning data to a working prediction engine and the start of a real system
