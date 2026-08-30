# Week 4 (2026-08-24 ~ 2026-08-28)
---

## What I did this week
**08/24**
- SDS POD_ver2 follow-up: confirmed with the requester that the flow succeeded; before this fix, the flow had been taking about 4 minutes per row.
- Started VTP work: before merging the two datasets, identified issues to resolve first — city names in the CN/CP dataset needed to match city names in the City Master dataset, and any CN/CP records with no clear location match needed to be checked against similar location or city names.

**08/25**
- Merged the CN/CP dataset with the City Master dataset after finding and reconciling city names that didn't match between the two.
- Attempted to visualize the merged data in Power BI using Azure Maps (the built-in free map visual).
- Found that Azure Maps could not recognize the route/path order from the merged data as originally structured (each row only had EquipmentId, city name, latitude, longitude, and event date).
- Reprocessed the data so each row represented a full route segment: added Origin/Destination city names, used latitude/longitude as Origin/Destination location, added an index field to represent route sequence, and split EventDate into origin and destination times.
- Also added a per-EquipmentId sequence number so route order could be recognized correctly when the same EquipmentId appeared multiple times.

**08/26**
- Started adding nodes between routes, since the map was only drawing straight lines between two points; the goal is to make the route look like a real railroad.
- Added nodes for one route as a test case.

**08/27**
- Added origin and final destination columns for each EquipmentId (vehicle).
- Grouped all route segments, resulting in 83 unique route pairs.
- Used AI assistance to collect candidate node information for all 83 route pairs, then ran a completeness check on the collected nodes.

**08/28**
- Organized the node list to be added across the 83 routes and saved it to a Nodes file.
- Loaded the file into Power BI and ran a completeness check using Power Query, covering: unique route pair counts and mismatches against the route pair summary; actual endpoint structure and coordinates (endpoint counts, first/last row checks, endpoint coordinate comparisons); NodeSequence duplicates and formatting; Routing Node name/coordinate standardization (same name with different coordinates, case/whitespace differences, same coordinate with different names); repeated coordinate checks; and required-value/range checks.
- Extracted and summarized the review targets, reasons for review, and consistency issues in the Basis and Confidence fields, then corrected them.

## What I learned (skills/concepts)
- After finishing a piece of work, doing a quick sanity check benefits both me and the requester — it's also a good way to confirm and quantify the time savings a new flow provides.
- Merging datasets on a shared key (like city name) requires validating that key carefully first; similar or duplicate city names across different provinces/states and countries make this trickier than expected.
- Power BI's Azure Maps visual needs data explicitly structured with origin/destination fields and a sequence number — it can't infer path order from a flat list of individual event locations.
- Data processing turned out to be nearly the entire job, more so than the visualization step itself.
- Got hands-on with Power Query in Power BI for the first time, and started to understand how it relates to the broader Power BI data model and table relationships.
- Deciding whether an intermediate node belongs on a route isn't a per-segment decision in isolation — it requires comparing all routes that share the same origin and destination to understand how many distinct paths exist between them.
- AI-assisted data collection (e.g., generating candidate route nodes) still needs a rigorous, structured completeness check before the results can be trusted.

## Challenges / problem-solving
- Reconciling city names between the CN/CP dataset and the City Master dataset was difficult due to many similar city names across different provinces and countries.
- Choosing the right Power BI visualization tool was hard without first understanding how each tool actually works.
- Power Query in Power BI was a first-time experience, which made the merging/reprocessing work more difficult than expected.
- Figuring out how to represent real event locations as a connected path (rather than straight lines) and identifying where additional nodes were structurally necessary.
- Working out clear criteria for when an intermediate node should be kept between two route endpoints — e.g., whether it sits on the actual rail alignment, whether the track bends near it, whether it's a major yard/junction, or whether removing it would cause the route to deviate significantly from the real network.
- Judging how far a data completeness check needs to go — settled on doing an automated pass to roughly 80% completeness, then manually verifying the rest by drawing the routes on a map.
- Missed a basic validation step (confirming province/state and country matched correctly) even while reviewing the data directly — an oversight that was avoidable just by looking more carefully.

## Next week plan
- Continue verifying and refining the node list, including manually confirming routes by drawing them on the map.
- Deepen understanding of table relationships in Power BI and how Power Query fits into the overall data model.
- Continue the route/node completeness validation work, including further Basis and Confidence consistency corrections identified during review.

## Reflection
- Now understand more concretely why my team lead recommended studying the Power Platform in depth — Power Query and table relationships turned out to be central to reliable data modeling work.
- Need to look at the data from a wider range of perspectives; currently feel like I'm only seeing about half of the full picture, and want to understand the data structure more thoroughly.
- A basic but important lesson from this week: don't skip the most obvious sanity checks (like confirming country/province correctness) even when reviewing data directly — it's easy to overlook something obvious while focused on more complex validation work.
