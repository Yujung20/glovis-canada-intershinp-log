# Week 10 (2026-10-05 ~ 2026-10-09)
---

## What I did this week
**10/05**
- VTP: Drew the Master Routes for the GRH-origin routes (destinations: CHR, WOL, STL). These run mostly on UP, which covers mainly US segments, so the segments after the UP portion were drawn with CN / CPKC as the priority
- VTP: Drew the Master Routes for the TAC-origin routes to the same three destinations, splitting them into two railroad lines (AMTK and BNSF), each with two paths — four routes in total
- VTP: Where the lines broke off from Chicago onward, connected them through the railroads with the most continuous coverage (UP, CN, etc.) so that each route stayed clearly distinguishable, and completed the routes
- VTP: Continued reviewing how to calculate distances between route locations (carried over from 10/02)

**10/06**
- VTP: Continued the route work from the previous day
- RPA: Found that the flow converting invalid PODs had stopped working. The cause was a change in the incoming email format, so I updated the flow's conditions to match the new format
- RPA: Followed up on the invoice-to-Excel conversion tool. Its accuracy is not yet 100%, so I will judge whether the issue can be fixed before deciding whether to continue
- VTP: Discussed the approach for the ETA prediction model. The merged CN / CPKC data covers only about 90% of freight trains, which raised several design questions:
  - How to predict ETA for the remaining ~10% of trains
  - Running AI predictions for every tracked train would waste credits, so one idea is to use AI only after a train has passed 50% of its total distance and use a different method before that point — but what that method should be is still open
  - How to reflect seasonal effects on travel time
  - Accuracy after the 50% point matters much more than before it, so the post-50% model needs particular care

**10/07**
- VTP: Matched the operating railroad to every route location and saved it in `Master_Route.xlsx`
- VTP: Organized `Master_Route.xlsx` as the reference file for all standard routes:
  - `Master_Route` sheet: location order for each standard route
  - `Master_Route_Merged` sheet: Master_Route merged with City_Master on city + province, adding latitude/longitude for each location. Distances between locations will be added to this sheet
- VTP: Obtained rail network data from the North American Rail Network Lines dataset — downloaded shapefiles for BNSF, UP, CPKC, and CN, and extracted them into separate folders per railroad
- VTP: Wrote `test_snap_cpkc.py` to snap route locations onto the rail network. The network data identifies locations by code rather than by name, so I compared each Master Route location (name + coordinates) with the network's locations (code + coordinates) and matched each name to the nearest code on the CPKC network
- VTP: Generated a snap-check result file that verifies connectivity and coordinate errors and classifies each location as Normal / Warning / Critical
- VTP: Set up a correction flow for Critical locations:
  1. Run a per-city candidate script (`check_{cityname}_candidates.py`) to extract nearby rail line candidates around the problem location
  2. Re-verify the most likely candidate on a map
  3. Fix the coordinates in `City_Master.xlsx`
  4. Refresh All in `Master_Route.xlsx` to update every sheet
  5. Re-run `test_snap_cpkc.py` and confirm the city no longer appears as Critical

**10/08**
- Day off

**10/09**
- VTP: Ran the snap-check code from 10/07 for each route and route version, and corrected the coordinates of Critical and Warning locations
- VTP: Wrote code to calculate, for each route, the total route distance, the cumulative distance from the origin to each location, and the progress rate (%) relative to the total distance
- VTP: Completed distance calculation for the WWL-origin and MOK-origin routes
- VTP: Handled railroad interchange segments — GRH-origin routes switch from UP to CPKC at Chicago, and TAC-origin routes switch from BNSF to CPKC at Chicago. Only the segment where the railroad changes is calculated as a straight-line distance
- VTP: Corrected the TAC-origin routes — I had initially drawn them as two lines (AMTK and BNSF), but on re-checking, they use the BNSF line only

## What I learned (skills/concepts)
- Even when trains enter through US ports, the railroad they use determines whether the route crosses the middle of the US or runs along the north. In Canada, most freight enters through the Port of Vancouver, so CN and CPKC routes end up fairly similar. In the US the routes diverge clearly, so confirming them takes much more verification
- Even without location codes, I could make the most of the data I already had: by comparing location name + coordinates against the rail network's code + coordinates, I could infer each location's code in reverse and snap it onto the actual rail line
- Turning "where is the train" into "how far along is the train" (cumulative distance and progress rate) is what makes a distance-based threshold such as the 50% point usable in the ETA model

## Challenges / problem-solving
- **Rail distance vs. straight-line distance**: Calculating distances from latitude/longitude alone only gives straight-line distance, which does not match actual rail distance. Solved by snapping each location onto the rail network shapefiles and calculating distance along the lines, with Normal / Warning / Critical checks and a correction flow for misplaced coordinates. Straight-line distance is used only at railroad interchange points
- **Sparse data in eastern Canada (open)**: There are fewer data points toward the east, so the current snapping approach may not work as well there. I need to find another method for those segments
- **ETA for trains not covered by the data (open)**: The ~10% of trains outside the CN / CPKC data have only their origin, final destination, and departure date. My first idea is to estimate their ETA from trains that departed on the same date on the same route
- **ETA before the 50% point and seasonality (open)**: I still need a method for estimating ETA before a train reaches 50% of its distance, and a way to reflect seasonal effects
- **Email format change breaking a flow**: The invalid-POD flow stopped because its conditions depended on the email format. Fixed by updating the conditions to the new format
- **Invoice-to-Excel accuracy (open)**: Accuracy is not yet 100%. Whether to continue depends on whether this can be solved

## Next week plan
- Finish distance calculation for the remaining routes, including the GRH- and TAC-origin routes with railroad interchanges
- Find a method for the eastern Canada segments where data is sparse
- Develop the ETA approach further: the method for trains outside the data, the pre-50% estimation method, and how to reflect seasonality
- Decide whether to continue the invoice-to-Excel conversion tool based on whether the accuracy issue can be resolved

## Reflection
- I drew the TAC-origin routes as two separate lines on Monday and only found out on Friday that they use one. Like last week's CN / CPKC case, this is a reminder that route structure has to be confirmed against official sources before building on top of it
- Instead of giving up when location codes were missing, I worked backward from the data I had. Building the check-and-correct loop (snap check → candidate search → coordinate fix → re-check) made it possible to fix problem locations systematically rather than one by one by hand
