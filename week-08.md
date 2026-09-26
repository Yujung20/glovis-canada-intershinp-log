# Week 8 (2026-09-21 ~ 2026-09-25)
---

## What I did this week
**09/21**
- VTP: Identified a key weakness in the existing data flow — whenever new data arrived, preprocessing had to be redone manually from scratch
- Redesigned the flow so that newly arriving data is preprocessed and transformed exactly the same way as the existing data, with no extra manual steps
- Rebuilt the full preprocessing → merge → transformation process in Power Query so it runs automatically whenever new data comes in, feeding directly into Power BI

**09/22**
- VTP: Planned to attach each rail location's unique code to every freight railcar record
- Since no internal dataset mapped these codes, began checking whether the railroads themselves provide this data
- Compared CN and CPKC reference data against my existing rail location data and listed the city names not yet covered
- Attempted to add each railcar's origin and final destination, but lacked access permission — submitted an access request

**09/23**
- Combined CN/CPKC reference data with my rail location (latitude/longitude) data to identify every city that appears in CN/CPKC but has no known coordinates
- Began adding location information for those missing cities

**09/24**
- Used CN's official data to fill in location information (latitude, longitude, etc.) for all cities missing from the City Master
- Verified each coordinate through a three-step process: official data check → AI cross-check → map-based verification

**09/25**
- VTP: Added origin and final destination codes to each railcar and visualized the results
- RPA: For the task of entering invoice contents into Excel, Power Automate would run into credit limits, so started building the automation in Python instead
- The current version extracts text from the invoice PDFs but also picks up unnecessary information; working on filtering it out

## What I learned (skills/concepts)
- A data pipeline should be designed not only for the data that exists today but for the data that will arrive later — the way data is processed must stay the same even if the owner or the situation changes
- Rebuilding the pipeline made the strength of the MS365 / Power Platform ecosystem very clear: preprocessing, merging, transformation, and visualization can be chained together without manual touch points, which would require far more hands-on work on other platforms
- Power Query + Power BI can handle a large data volume end-to-end, in cases where manual work — and even Power Automate — would be too slow
- Location names in rail data are not always standard city names. The same city can appear under different labels (e.g., "Toronto" vs. "Toronyard") that actually refer to different locations, so each one must be verified individually
- Even data from official or provided sources needs validation; combining official data, AI cross-checking, and map verification gives much more reliable coordinates than relying on AI or manual lookup alone
- Filtering by origin and final destination codes alone was enough to see firsthand how many different routes a railcar can take

## Challenges / problem-solving
- The newly added data was many times larger than the previous dataset — impossible to process manually, and the existing Power Automate approach would have been too slow to meet the deadline for ongoing work. Solved by moving the entire preprocessing, transformation, and visualization process into Power Query and Power BI
- Needed a source for new location data that is both reliable and covers both CN and CPKC, and then had to plot the coordinates on a map to confirm they actually sit on the rail network. Resolved by using CN's official data together with the three-step verification process
- Adding origin/destination data was blocked by access permissions — submitted an access request and continued with other work in the meantime
- Power Automate's credit limits made it unsuitable for the invoice-to-Excel task, so switched to Python; the extracted output still contains noise that needs to be filtered
- Open issue: when the same railcar number has two or more departure–destination code pairs, I need evidence to tell whether the railcar genuinely ran different routes or was stored in a compound and later released

## Next week plan
- Define criteria and find supporting evidence to distinguish multiple departure–destination pairs per railcar (separate trips vs. compound storage)
- Finish cleaning up the PDF extraction in the Python invoice automation so that only the required fields are written to Excel
- Study the roles and purposes of Power Apps, Power Query, Power BI, and Power Automate, and how they fit together

## Reflection
- I need deeper knowledge of MS365. In particular, I should clearly understand the role and purpose of each of the four Power Platform tools — Power Apps, Power Query, Power BI, and Power Automate
- When asking someone for an explanation, I should communicate exactly which parts I already understand and where I need more detail — my requirements should be organized clearly before I ask
- I was satisfied that I didn't give up just because the data I needed wasn't in my current dataset. I searched the web and the railroads' systems for it, and also thought about how the extra data I found could be useful later, even if not right away
- Rather than defaulting to AI or doing everything by hand, I tried to make maximum use of available official data first — and still validated it, which is a habit I want to keep
