# Week 14 (2026-11-02 ~ 2026-11-06)
---

## What I did this week
**11/02**
- Reviewed VTP_ETA_code v1.1 (the demo system built with Vue.js + FastAPI) and updated it to v1.2.
- Company data cannot leave the company, so I made test data to check the full flow.
- Goal: keep the Route_Distance_Integrated file as is, and make the code work with any updated Merged and VIN Movement data.
- Fixed problems that broke when data was updated:
  - `AS_OF_DATE` was fixed to 2026-10-28, so 350 ShipOut rows and 497 Delivered rows after that date were dropped without any error. Now it is set automatically to the last date in the input data (dates later than the run date are ignored). It can still be fixed by hand with `P["AS_OF_DATE"]`.
  - A date typo (for example, the year 3172) stopped Step0. Now the error is only counted, and the pipeline keeps running.
- Fixed a mismatch between training and prediction:
  - When Step3 picked GBM, M2 was trained on a GBM-based R0, but the engine only uses B1. Now Step3 only records the comparison result and always passes B1 to the next step. `meta.json` saves `stage1_best` (comparison result) and `stage1_operational` (B1, used in production) separately.
  - The engine calculated B1 differently from Step3. I changed the engine to follow the same rule as Step3: last 60 days by arrival date, at least 3 trips, valid trips only. The values 60 days / 3 trips / default 14 days are now read from `eta_common.P` in one place.
- Included the fixes from 11/01:
  - The engine now uses the features selected by Step4-1 (7 candidates).
  - Fixed a duplicate `R0` column bug in Step4-2.
  - Added engine checks to `step5_validate_engine.py`.
- Changed 8 files in total: `eta_common.py`, `step0_structure_check.py`, `step1_clean.py`, `step3_stage1_model.py`, `step4-1_fe.py`, `step4-2_compare.py`, `eta_prediction_engine.py`, `step5_validate_engine.py`.
- Tested 3 data scenarios (same Route file, different VIN/Merged files) in the same order as the UI (Step0 → predict_input). All passed with no replacement files and no manual fixes.
- UI fixes:
  - A shared CSS rule (`.card + .card` margin) made side-by-side boxes different heights. Fixed it on the Dashboard, Data Management, Pipeline, Validation, and Settings pages.
  - Dashboard: changed the run history Duration unit to minutes (m).
  - Analytics: set 2 decimal places on the Model MAE vs Carrier MAE and Performance by Origin–Destination charts and tooltips, and removed the numbers in front of box titles.

**11/03**
- Checked how v1.2 works:
  - The real arrival date (the answer) is **Delivered** (the earliest VIN Delivered date in a trip).
  - Stage1 lead time = Delivered − ShipOut. Stage2 remaining days = Delivered − observation date.
  - The last event is not the answer. It is the reference point for prediction and an input for features.
  - If a trip has a destination event but no Delivered date, it is still an open trip and is predicted.
- Checked the B1 rule again: the median lead time of trips on the same OD that were delivered within the last 60 days (based on ShipOut).
  - Same OD = one group, no matter the RouteVersion.
  - Only valid completed trips, at least 3 trips.
  - Fallback order: same OD last 60 days → same OD all history → same origin last 60 days → all trips last 60 days → all history → fixed 14 days.
- Organized new requests:
  - Top 3 and bottom 3 routes (OD) by ETA accuracy
  - ETA accuracy by origin port (3 ports)
  - Monthly ETA accuracy by port
  - AsOfDate was always fixed to 10/29/2026 → change it to the current date
  - Compare the company's current method (average lead time per route) vs B1 vs the final model (B1 + M2_FINAL), and tell the story as "problems with current ETA → B1 baseline → M2_FINAL correction"
- Built v1.3 (same model, new screens and comparison targets):
  - Comparison: Average (company method) vs B1 + M2_FINAL vs Carrier. B1 is shown as a reference column in the number table.
  - AsOfDate default: the last Event Date in Merged, and the user can change it.
  - Analytics period: Test period only (monthly trends use the full period).
  - Analytics boxes: 10 → 13. New: top/bottom 3 routes, accuracy by port, monthly accuracy by port (Test), full-period monthly trend for Average and B1. Merged the Carrier distribution and model distribution boxes into one.
  - Chart changes: "Which ETA was closer?" donut → horizontal bar / two error distribution charts → one chart with 3 lines / Performance by OD bars → dots per route / Prediction method two axes → one MAE axis.
  - Added an accuracy formula: `100% − (sum of trip errors ÷ sum of actual lead times)` (per route and per port).
  - New input file: `input\Ave_T_time.xlsx`. If it is missing, only Average is removed and the page still works.
  - No need to rerun the pipeline. To apply a new AsOfDate, only `predict_input` needs to run again.
- Wrote the **VTP ETA System UI Guide** and a summary of the differences between v1.1, v1.2, and v1.3 (engine checks: 110 → 112 → 112).

**11/04**
- Merged the accuracy analysis from 11/03 (by route, port, and month; Average vs B1 vs B1 + M2_FINAL) into the main code.
- Added a new **VIN Map** page to the navigation:
  - Purpose: show on a map where in-transit VINs are on each AsOfDate.
  - Date controls: date picker, previous/next day buttons, and a slider (dates outside the data range cannot be selected).
  - Filters: Origin, Destination, "Include no event yet" (include VINs still at the origin because they have no event), and customer (All / two customers).
  - Map: labels show VIN counts per location. When zoomed out, nearby locations group into a circle with the total; when zoomed in, they split again. Colors separate VINs at their latest event location from locations with only no-event (origin) VINs.
  - Clicking a location opens a popup: city/state, VIN count, VIN count by customer, railcar count, AsOfDate, and VIN/railcar counts by OD.
  - Right panel: total in-transit VINs, railcars, locations, no-event VINs, VINs with no coordinates (not drawn), and a VIN list by location (click to move to that location).
  - Rule: VINs on a railcar are removed from the in-transit count starting from the first Delivered date.
  - Even if map background tiles fail to load, VIN counts and locations still show. An internal tile server can be set with `VTP_MAP_TILE_URL`.
- Added the **Customer** column back to VIN Movement (it was removed in preprocessing before, but the VIN Map needs it). Columns went from 8 to 9.
- Added the customer filter (All / two customers) to the Dashboard, Route Map, VIN Map, and Analysis pages.
- Updated Settings: each sidebar menu can be turned on/off (saved in the browser). Hidden pages still open by link or URL, and at least one page is always shown.
- Updated data: Merged up to 11/04, VIN Movement up to 11/03 (same-day data keeps changing, so I use data up to the day before).

**11/05**
- Studied the company's current manual Excel process:
  - Pull only Rail **in-transit** shipments from the internal TMS.
  - For each VIN and railcar, use the ETA from the rail carrier if it exists. If not, estimate the ETA with the `Ave_T_time` value.
  - Build a table for the next 14 days (including today) with the number of arriving vehicles by compound and date, and by customer.
  - Everything is done with copy-paste and formulas inside Excel sheets.
- Request: build the same table using ETAs predicted by the M2_FINAL model.
- Added a new **INTransit** page to the navigation:
  - Left (VIN Tracking Upload): choose a VIN Tracking Excel file and click Run. The file name can be different, but the columns must follow the existing VIN Tracking structure. Period: today + 14 days, format: .xlsx.
  - Right (INTransit Preview): shows the result with **Download Excel** (e.g., `INTransit_ETA_20261108_131657.xlsx`). Counts are based on unique VIN numbers.
  - Table: rows = customer → destination compound, columns = Over ETA + each date.
  - Example result for one origin port (11/08 ~ 11/21): 6,177 VINs, 5,063 ETA Predicted, 863 Over ETA, 1,114 ETA Unavailable.

**11/06**
- The Rail ETA system and the Vessel program were combined into one program (switch between Vessel and Rail at the top of the sidebar).
  - The Vessel program estimates when and how many vehicles shipped by sea arrive at each compound.
  - Possible direction: connect the two into one system that covers ship → rail.
  - I don't understand how the two systems relate yet, so I plan to read the Vessel guide.
- Made some UI fixes to Rail ETA.
- VIN Map (in progress): circle size = volume. Working on showing the **VIN list** inside a circle when the user clicks it.

## What I learned (skills/concepts)
- In the Vue frontend, my changes to the JS files in `frontend\src\views` did not show up in the system. The running app uses the built files, not the source files. After running `npm run build`, the files in `frontend\dist` are rebuilt with my changes, and the system uses them.
- A reference date like `AS_OF_DATE` should come from the data, not be hard-coded. A fixed date silently dropped new rows without any error.
- Training and prediction must use the exact same rule. If the model is trained on one baseline (GBM-based R0) but the engine uses another (B1), the model's corrections no longer match. Keeping shared settings (60 days, 3 trips, 14 days) in one place (`eta_common.P`) prevents this.
- What counts as the "answer" (Delivered) and what is only an input (last event) must be clear before building metrics and pages.
- To show a model's value, comparing it with the company's current method (route average) tells a clearer story than comparing it only with the carrier ETA.

## Challenges / problem-solving
- I couldn't use company data outside the company, so I made test data and ran 3 scenarios in the same order as the UI to make sure the code works with any updated Merged/VIN Movement files.
- I had removed the Customer column in preprocessing, but the VIN Map needed it. I had to go back through the preprocessing steps to add it again.
- The VIN Map is drawn, but I'm not sure it shows what I intended. I need to check the code again and compare it with real data.
- The INTransit result still has "ETA Unavailable" VINs (1,114 in the example). Since the goal is to predict ETA, this should not happen. I need to find the cases where ETA cannot be estimated and decide on a fallback when the model has no value.

## Next week plan
- Finish the VIN Map feature that shows the VIN list when a circle is clicked.
- Check the VIN Map against real data to make sure it shows the right locations and counts.
- Find out why some VINs are "ETA Unavailable" on the INTransit page and add a fallback.
- Read the Vessel program guide and understand how it connects to the Rail ETA system.

## Reflection
- Because I split the preprocessing into clear steps in order, adding the Customer column back was not hard. I think this was a good decision.
- This week the work moved from fixing the model to building tools people can use (Analytics comparison, VIN Map, INTransit). The INTransit page showed me that a good model is not enough; the output also needs to cover every case the team works with.
