# Week 4 (2026-08-24 ~ 2026-08-28)
---

## What I did this week
**08/24**
- SDS POD_ver2: confirmed the flow succeeded on follow-up, and validated with the requester that each row took about 4 minutes to process before the flow was introduced.
- VTP (City Master merge prep): identified two issues to resolve before merging the CN/CP dataset with the City Master dataset — the CN/CP city names need to match the City Master city names, and any CN/CP records with no matching location data need to be checked for similar location or city names.

**08/25**
- Checked whether each CN/CP row's city name matched a city name in the City Master dataset, found mismatches, corrected them, and merged the two datasets.
- Attempted to visualize the merged data in Power BI using the Azure Maps visual, but the visual could not recognize the route order from the raw row structure (EquipmentId, City name, Latitude, Longitude, EventDate).
- Restructured the data so each row explicitly contains EquipmentId, origin/destination city name, origin/destination time, and origin/destination latitude/longitude, and added a sequence number per EquipmentId so route order could be recognized.

**08/26**
- Started adding intermediate nodes between routes so the map would resemble an actual railroad instead of straight lines between two points.
- Added nodes for one route as a test case.

**08/27**
- Added origin and final destination fields for each EquipmentId (truck/train unit).
- Grouped all data into 83 total route segments.
- Used AI to collect candidate node information for all 83 segments and checked the completeness of the collected nodes.

**08/28**
- Organized the finalized node list for the 83 routes and saved it to a dedicated Nodes file.
- Loaded the Nodes file into Power BI and ran a completeness check using Power Query, covering: route pair coverage (unique pairs, mismatches against the route pair summary), actual endpoint structure and coordinate checks, NodeSequence duplication/format checks, standardization of routing node names and coordinates, repeated coordinate checks, required/allowed value checks, and a final summary of items needing review (by basis, confidence, and physical node).

## What I learned (skills/concepts)
- After finishing a project or flow, doing a quick sanity check benefits both the builder and the requester — it's also a chance to confirm concretely how much time the automation saves.
- Power BI offers many visualization tools, but choosing the right one requires understanding how each one actually works and what shape of data it expects.
- Data processing is close to the entire job — how the data is structured determines whether a visualization tool like Azure Maps can even interpret it correctly.
- When restructuring data, it's necessary to personally verify the output, since transformations can change the data in unexpected ways.
- Route/node analysis requires deciding whether an intermediate point belongs on a route based on multiple criteria: whether it lies on the actual rail line, whether the line bends there, whether it's a major hub or yard, and whether omitting it would make the route deviate significantly from the real network.
- Gained a clearer sense of the relationship between Power BI and Power Query, and a better understanding of why the team lead emphasized studying the Power Platform.

## Challenges / problem-solving
- Comparing and reconciling city names between the CN/CP dataset and the City Master dataset was difficult due to many similar city names, requiring location checks to determine the correct province/state for each.
- Choosing the right visualization tool for the data was hard without first understanding how each tool works; this was also the first hands-on experience with Power Query in Power BI, which added to the difficulty.
- Determining how to source accurate location data between two known points on a route was an open problem, addressed by cross-referencing publicly available CN/CP network data.
- Deciding how far to take the completeness check was difficult — it wasn't obvious where "thorough enough" ends, so the approach taken was to complete about 80% of the systematic checks first and validate the remainder directly by plotting the data on a map.

## Next week plan
- Continue and finish the remaining ~20% of the node completeness check, including direct map-based validation.
- Draw and review the map with the added routing nodes to confirm it visually resembles the real rail network.
- Double-check basic fields (state/province and country) that were missed this week, and build a habit of verifying them early in the process.
- Continue building out state/country and location validation as part of the standard data-check routine going forward.

## Reflection
- I need to learn to look at the data more broadly and from multiple perspectives — right now I feel like I'm only seeing about 50% of the full picture, and I want to understand the structure of the data more completely.
- I also need a clearer understanding of table structure in Power BI and need to be more precise when building relationships between tables.
- I missed checking something as basic as whether the state and country values were correct, even while looking directly at the data — a reminder that fundamental checks shouldn't be skipped even when focused on more complex validation logic.
