# Week 11 (2026-10-13 ~ 2026-10-16)
---

## What I did this week
**10/13**
- Finished calculating hub-to-hub distances for the standard rail routes: 36 routes in total, with distances computed between every hub on each route
- Used Excel Power Query to combine all per-route distance files in a folder into a single sheet, producing a `Master_Route_Distance` table
- Built visualizations on top of `Master_Route_Distance`
- Studied an overview document of the company website's architecture

**10/14**
- Resolved the last route whose hub distances could not be calculated
  - Root cause: CPKC's own track does not extend to that eastern destination; CPKC operates over a partner railway's track for that segment
  - Used the partner railway's GeoJSON data to calculate the remaining distances
  - Distance calculation is now complete for every route and every route version
- Started discussing the ETA model: transit time will be calculated from the first departure date to the arrival date, defined as `Delivered − Ship out` in the internal transportation management system

**10/15**
- Clarified the scope of the request to build the prediction model with Copilot Studio — two possible interpretations:
  1. Use an LLM (GPT / Anthropic API, etc.) to perform the ETA inference itself
  2. Build the ETA prediction with ML or rule-based logic, and use the AI agent only to answer questions (e.g. "What is the ETA of freight train X departing Vancouver?") by returning the model's results
- Confirmed that both approaches are feasible in Copilot Studio; a follow-up meeting with my senior is needed to decide
- Extracted all records from 2025-01-01 to 2026-10-13 from the internal system and calculated lead time (`Delivered − Ship out`, stored as `delivery days`)
- Removed outliers, including negative values, using the IQR method
- Visualized average lead time and differences by season with scatter plots and bar charts

**10/16**
- Received feedback from the team lead: extend the seasonal analysis to lead time by **route × season**
  - Core work: linking three datasets — the master route table, the merged rail event data, and vehicle movement records (extracted 10/15, covering 2025-01-01 ~ 2026-10-13)
  - Per the team lead, this is to be done with Power BI data modeling rather than Power Query (in progress)
- Received a new request: ideas for optimizing a client's warehouse layout using reinforcement learning; data will be shared on 10/19 for review

## What I learned (skills/concepts)
- **Spring Boot request flow**
  - `DispatcherServlet` receives every HTTP request (Front Controller pattern) and routes it to the right controller; Spring configures it automatically
  - `@RestController` with `@GetMapping` / `@PostMapping` maps URLs and HTTP methods to endpoints, and returned objects are serialized to JSON automatically
  - `@Service` holds the business logic, talks to the database through the Repository layer, and manages transactions with `@Transactional`
  - Spring Security handles authentication and authorization (login, JWT validation, permission checks) through a Filter Chain that intercepts every request
- **Copilot Studio**: supports both "LLM as the predictor" and "agent as an interface to an existing model"; websites can be added as knowledge sources when building an agent
- **Power BI data modeling vs. Power Query**
  - Better for multi-dimensional analysis like route × season: with date (season) and route tables as dimensions, a single DAX measure recalculates automatically as slicers (origin, destination, source, season) change, instead of pre-aggregating every combination into a large flat table
  - Better for data that grows daily: the merged table is refreshed every day via Power Automate, and repeatedly merging large tables slows refreshes and complicates the Power Query chain; with relationships in the model, each table refreshes independently and maintenance stays simple
- **Rail network ownership**: a carrier's routes don't always run on its own track — trackage over a partner railway has to be accounted for when calculating distances

## Challenges / problem-solving
- **Missing distances for one route** → traced the cause to track ownership, found that the segment runs over a partner railway, and used that railway's GeoJSON to complete the calculation
- **Ambiguous AI requirement for the ETA model** → broke it down into two concrete interpretations, verified both are possible in Copilot Studio, and prepared to confirm the direction with my senior
- **Choosing knowledge sources for the Copilot Studio agent** → still need to research which websites are reliable enough to add
- **Why data modeling instead of Power Query** → worked through the reasoning (multi-axis analysis and daily-growing data), which made the team lead's instruction make sense
- **Unfamiliar territory for the warehouse task** → need to understand the core idea of reinforcement learning (maximizing cumulative reward) before the data arrives
- **Frontend gap** → need to study the differences between Vue.js and React.js to better understand the company's system

## Next week plan
- Complete the route × season lead time visualization using Power BI data modeling
- Meet with my senior to decide which Copilot Studio approach to take for the ETA model
- Research reliable websites to use as knowledge sources for the Copilot Studio agent
- Review the warehouse data on 10/19 and propose reinforcement learning ideas
- Practice reinforcement learning on public datasets (e.g. Kaggle) to get a working understanding of reward maximization
- Study the differences between Vue.js and React.js

## Reflection
- With distance calculation finished for every route, the VTP data preparation phase is essentially complete, and the work is shifting toward actual ETA modeling
- When a requirement was vague, splitting it into concrete interpretations and checking feasibility first made the conversation with my senior much easier — a habit worth keeping
- The team lead's push toward data modeling showed me that tool choice should follow how the data grows and how it will be analyzed, not just what gets the result fastest
