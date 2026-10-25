# Week 12 (2026-10-19 ~ 2026-10-23)
---

## What I did this week
**10/19**
- Rebuilt the route × season lead-time analysis from scratch in Power BI, dropping the previous approach because merging tables in Power Query becomes inefficient as data grows
- Fixed the Merged pipeline's SharePoint connection issue
  - Before: Power Query read a manually copied version of the Merged Template, then preprocessed it and merged City Master data
  - After: Power Query connects directly to the Merged Template file that Power Automate generates, so preprocessing runs automatically from the source path
- Loaded three base tables separately (no query merges) and connected them through data modeling instead
  - `VIN_Lead_Time`: completed shipments with actual lead times (fact data)
  - `CNCP_Movement`: actual cities and sequence per EquipmentId (movement event data)
  - `Route_Master`: route, hub order, and distances per O-D RouteVersion (grain: one route node within a RouteVersion)
- Added a `Route_Pair` (Origin→Destination) column to both Route_Master and VIN_Lead_Time, plus a `Route_Version_Count` measure on Route_Master
- Built lead-time measures on VIN_Lead_Time: average / max / median / min delivery days, delivery-day range, shipment count, and multi-version shipment count (shipments that still need RouteVersion resolution)
- Created a `Dim_RoutePair` dimension table (union of Route_Pairs from both tables, deduplicated) and related it 1:* to Route_Master and VIN_Lead_Time; CNCP_Movement is loaded but not yet related
- Built a clustered column chart (Route_Pair × average delivery days, legend by season, tooltips for median / min / max / shipment count) and switched every route-based visual and slicer to `Dim_RoutePair[RoutePair]`

**10/20**
- Designed the ETA model structure: defined the purpose of the ETA and the possible prediction scenarios
- Split cases by whether intermediate hub events are reported (about 10% of shipments have none) and structured the progress-rate thresholds for building separate models
- Tried weighted regression, but the rail carrier only provides a ship-out ETA, not a delivered ETA, so another approach was needed

**10/21**
- (No log recorded)

**10/22**
- Ran the two-stage ETA model on real data

| Stage | Best method | Result |
| --- | --- | --- |
| Stage 1 (departure → total transit days) | B1 (recent route median) | MAE 1.78 days, predicts 0.88 days earlier than actual on average |
| Stage 2 (remaining-days correction in transit) | R2 (route × progress-rate median) | MAE 1.63 days |

- Confirmed that error decreases as progress rate increases, and judged the model more accurate than the rail carrier's ETA — a meaningful result on its own
- Validated stage 2 labels (min / median / max) against 5,148 comparison trips; all showed a mean difference of about 2 days
- Confirmed that the ETA Date in each CNCP_Movement row is the ETA to the final destination, not to that row's city, and that VIN_Lead_Time has no ETA field

**10/23**
- Finalized step 4-1 requirements: use stage 1 (B1) in early progress and switch to stage 2 after a threshold (e.g., 40%); goal is maximum accuracy with no runtime limit
- Ran step 4-1 feature engineering on the company PC; fixed a label-check error caused by mixed date formats in `delivered_median` (numbers before the error were valid, only the report export failed, so only step 4-1 needed a rerun)
- Step 4-1 results

| Method | Validation MAE | Test MAE |
| --- | --- | --- |
| M2_best (GBM) | 1.38 | 1.72 |
| R2N (hub-level historical remaining days, new) | 1.60 | **1.55** |
| R2 | – | 1.63 |
| R0 | – | 1.73 |

- GBM ranked first on validation but fell behind on test (short split: 3 weeks train / 1 week validation / 2 weeks test)
- By progress segment: 0–25% → R0 (B1) 1.78 beats R2N 1.95; 50–75% → R2 best at 1.31; 75–100% → R2N 1.14 (floor 0.87 days)
- On rows that have a carrier ETA: CARRIER 2.61 / CARRIER_ADJ (unloading-delay corrected) 2.13 / R2 1.85 / R2N 1.88
- Removing the carrier ETA from GBM features barely changed results (1.417 → 1.405)
- Switch point: switching at 20% gave 1.48 vs 1.51 at 40% (0.03-day difference)
- Shared step 2 coverage results: about 6,400 trips with a resolved RouteVersion (281 tied), event coverage among completed trips only 43.5%, event period 2026-04-05 ~ 2026-10-22
- Prepared a coverage diagnostic script that traces, by departure month, at which stage trips drop out (wagon exists in Merged → event linked → hub matched → trainable); pending run on real data

## What I learned (skills/concepts)
- **Power BI star schema**: a shared dimension table (`Dim_RoutePair`) can bridge two fact tables without merging them, but axes and slicers must come from the dimension so filters propagate to both facts
- **Defining grain per table**: being explicit about what one row means (a shipment, a movement event, a route node) made relationship design much clearer
- **Closing the pipeline loop**: pointing Power Query at the file Power Automate generates removes the manual copy step and keeps the pipeline automatic end to end
- **Validation vs. test reversal**: with short time-based splits, a complex model (GBM) can win on validation and lose on test, while simple median baselines stay robust
- **Segment-wise hybrid models**: no single method wins across all progress ranges, so choosing the method per progress segment is more accurate than one global model
- **Fair comparison and feature dependence**: checked whether a model that uses the carrier ETA as a feature can be fairly compared against the carrier ETA itself, and confirmed by ablation that the model barely depends on it
- **Understanding field semantics before modeling**: the ETA Date turned out to mean the final-destination rail arrival, not the arrival at each hub — this changes how it can be used

## Challenges / problem-solving
- **Segmented route mismatch**: our collected data records a route A→B→C as origin A and destination C, while the internal system splits it into A→B and B→C. Still need to decide how to reconcile the two
- **No delivered ETA from the carrier**: weighted regression was not feasible, so moved to median-based baselines (B1, R2, R2N) plus a GBM correction model
- **Is the M2 comparison fair?**: instead of stopping at the all-features GBM, tested feature combinations to find the best M2, then compared it again with R0 / R1 / R2 / CARRIER, and tested excluding the carrier ETA feature
- **Mixed date formats in labels**: fixed the label-check error and reran only step 4-1 instead of the whole pipeline
- **Low event coverage (43.5%) and late training start (8/8 departures)**: rail event data shows U.S.-origin cars moving through several hubs, while the internal shipment data records every U.S.-origin route with the same final destination. These legs need to be stitched together; a diagnostic script was prepared to locate where trips drop out

## Next week plan
- Run the coverage diagnostic on real data and identify the stage where most trips drop out
- Link U.S.-origin legs between the rail event data and the internal shipment data to raise coverage and extend the training window earlier than 8/8
- Decide how to handle the segmented route mismatch (A→C vs. A→B, B→C)
- Retrain and re-evaluate with a longer period, then finalize the switch point (20% vs. 40%) and the per-segment method (B1 / R2 / R2N)
- Define handling for the ~10% of shipments without intermediate hub events
- Connect CNCP_Movement to the Power BI data model

## Reflection
- Rebuilding the lead-time analysis instead of patching it was the right call — designing the model around table grain and a shared dimension made it easier to extend
- The biggest lesson of the week was that the simple baselines held up better than the complex model on unseen data. Before adding complexity, the data itself (coverage, period length, how routes are recorded) needs to be fixed, because that is what limits accuracy right now
