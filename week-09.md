# Week 9 (2026-09-28 ~ 2026-10-02)
---

## What I did this week
**09/28**
- RPA: Built a Python tool that extracts the contents of invoice PDFs and converts them into a new Excel file (one PDF = one invoice, with one Excel row per order contained in that invoice)
- RPA: Packaged the tool so users can run the conversion with a single click instead of typing commands like `python main.py`, and installed it on the user's machine
- VTP: Added a per-railcar sequence (order of locations passed) and visualized it
- VTP: Corrected data points that fell outside the expected path of each route

**09/29**
- VTP: Reviewed the dataset structure — the merged tracking data (source railroad, railcar ID, vehicle ID, city, province, coordinates, event date, ETA) and the railcar origin/destination table, where one railcar can have more than one origin–destination pair because it runs multiple trips
- VTP: Identified the limitation of the existing preprocessing — since I couldn't tell where one trip of a railcar ended and the next began, I had been spotting outliers on the map and fixing them by hand, which doesn't work for continuously incoming data
- VTP: Incorporated new trip-identifying columns received from 09/29 — a release date for CN, and a waybill number plus waybill date for CPKC — so that rows sharing the same railcar ID and these values can be treated as the same trip
- VTP: Redesigned the automation: Power Automate generates the merged data on SharePoint daily → Excel Power Query loads it from SharePoint → preprocessing and sequence numbering are calculated automatically
- VTP: Added a `TripKey` to distinguish new data (e.g. `CN|EquipmentId|ReleaseDate`) from historical data without trip identifiers (`Source|EquipmentId|Historical`), while keeping the existing single-pair railcar filter

**09/30**
- VTP: Visualized the sequences on a map, filterable by origin and destination so each pair's approximate path is drawn
- VTP: Started a Master Route table per origin–destination pair (route, origin code, destination code, route version, route sequence, city, province), using route versions to separate multiple paths between the same pair
- VTP: Completed the first three Master Routes out of Vancouver
- VTP: Connected the Master Route to Excel via Power Query, auto-preprocessed it, and matched latitude/longitude from the City Master
- VTP: Loaded both the tracking data and the Master Route into Power BI as separate tables for visualization

**10/01**
- VTP: Completed the Master Routes for all destinations out of Vancouver (Calgary, Regina, Edmonton, Toronto, Winnipeg, Quebec, Montreal, Halifax)
- VTP: Added three Power BI filters — origin, final destination, and source railroad (CN / CPKC)
- VTP: Checked each railroad's official network map and confirmed that CN and CPKC use completely different lines on the Toronto–Montreal segment
- VTP: Reflected these differences by splitting such pairs into multiple route versions, without tying a route version number to a specific railroad (since some segments are served by only one railroad)

**10/02**
- VTP: Reinforced the Vancouver-origin routes — data thins out the further east the trains go, so I back-estimated the eastern segments from the CN and CPKC official network pages and filled in intermediate locations more densely
- VTP: Wrote all Master Routes for trains originating from Monterrey using the same method
- VTP: Started reviewing how to calculate distances between route locations

## What I learned (skills/concepts)
- Data almost always needs additional fields once it is used for a specific purpose. The most important part of data architecture is designing so those additions can be made without breaking the existing process
- Preprocessing should be reversible. `TripKey` was created simply to separate historical from new data, but it turned out to be the key to undoing grouped results later — designing for reversibility is what keeps the raw data from being lost
- Instead of treating each origin–destination pair as a separate route, identifying one trunk route and the point where each destination branches off made Master Route writing much faster
- The same origin–destination pair can run on different railroads with different lines, so the railroad behind each drawn path has to be identified

## Challenges / problem-solving
- **Splitting multiple trips of the same railcar**: The old approach relied on manual map-based correction, which couldn't scale to daily incoming data. Solved by using the newly received trip-identifying columns and a `TripKey` that also keeps historical data distinguishable
- **Mixed CN / CPKC paths**: Some segments differ completely between the two railroads. Handled by checking official network maps and splitting pairs into separate route versions
- **Sparse data in the east**: Fewer tracking points toward eastern Canada made route estimation difficult. Filled the gaps by back-estimating from the railroads' official network pages
- **Distance between route locations (open)**: Latitude/longitude only give straight-line distance, which doesn't match actual rail distance. CPKC's data has a table with distances and location codes, but I haven't yet found the spacing between consecutive locations. A US railroad's site shows the distance to the nearest location for a selected point, but checking every location one by one isn't practical
- **Scalability and full automation (open)**: The data keeps growing, so I need an alternative before the Excel-based flow becomes too heavy. Also, since the Excel file still has to be opened and refreshed, the pipeline is effectively semi-automated

## Next week plan
- Verify the completed Master Routes against the official CN / CPKC network maps and finish any remaining origin–destination pairs
- Investigate CPKC's distance data structure further and decide on a method to calculate rail distance between route locations
- Explore storage/processing options for growing data volume and ways to remove the manual Excel refresh step

## Reflection
- I found the trip-splitting problem on Monday and completed the planned automation transition the next day
- I copied the existing Power Automate flow to avoid disrupting it. My own flow ran successfully, but I missed part of the original flow that also needed updating and had to spend time restoring it. When designing flows, every part — including the ones I'm not directly changing — needs to be checked carefully
- Visualization makes things easy to see at a glance, which also makes it easy to trust what I see without question. The CN / CPKC route mixing could have been avoided if I had checked the official network maps before building the Master Routes. Validation has to come before building, not after
