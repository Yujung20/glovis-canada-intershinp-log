# Week 15 (2026-11-09 ~ 2026-11-13)
---

## What I did this week
**11/09**
- Updated how Rail ETA connects with the Vessel system
  - Logistics flow: Ship → Port → Rail → Compound
  - Rail ETA (my part): estimates the compound arrival ETA from rail segment data
  - Vessel: estimates when and how many vehicles move from the ship to each compound
- Updated the INTransit page
  - Expanded the scope: from trips leaving one specific origin only → trips from all origins
  - Changed "ETA Unavailable" to 0: all unavailable rows are now filled with the average transit time (`Ave_T_time`)
- Found two causes of "ETA Unavailable"
  - **Data update timing:** VIN Movement data is updated daily with the previous day's data (e.g., on 11/8 it adds 11/7). If EDI arrives late and 11/7 data comes in later, the next update (11/9) only adds 11/8 data, so the late rows are missed
  - **Route key mismatch:** the merged rail carrier data has a terminal event, but VIN Movement does not. For example, a trip from origin A comes in as `A|B`, and there is no `B|C` data yet because it has not reached B. The lookup uses `A|C`, so the route does not match and no ETA is estimated
  - This logic will move into the company system later. Because that system runs in real time, I expect these issues to go away
- Updated the VIN Map: clicking a location circle now shows the VIN list for that location on the page

**11/10**
- Added the Compound Forecast feature
  - Purpose: Rail ETA shows when and how many vehicles arrive at each compound, and the Vessel side estimates the same. Compound Forecast combines both into a table of November arrivals by compound and by date
- Organized the overall structure of the integrated ETA Portal (based on PPT screenshots)
  - The Home screen shows three systems at once, in this order: Vessel → Port, Port → Compound, Rail → Dealer

| System | Role | Main menus |
|---|---|---|
| Vessel ETA system | Predicts vessel arrival | Dashboard, Port Map, Vessels, Arrival Forecast, Customers, Schedule Sheets, Reports, Model & Validation, Settings |
| VTP Rail ETA | Predicts rail ETA (my part) | Dashboard, Data Management, Pipeline, Model Management, Validation, ETA Prediction, INTransit, Route Map, VIN Map, Analytics, Model Matrix, Reports, Settings |
| Compound Forecast (new) | Predicts arrivals per compound from vessel ETA × compound ratio | Compound Forecast, Plan & Manual Input, Distribution Model, History Statistics, Data, Model Settings |

- Reviewed the Compound Forecast screens
  - **Compound Forecast:** shows expected volume per compound as of a chosen date
    - Input mode: Vessel plan, Manual input, or Both
    - KPI cards: Input Qty, At Compounds by As-of, On the Way, Vessel Not Landed, Next 7 Days, Not Distributed
    - A status chart per compound (At compound / On the way / Vessel not landed) and a table by departure port
  - **Plan & Manual Input:** enter the vessel plan (vessel, port, vessel ETA, quantity). Manual input per port is possible, and the port-to-compound ratio and lead time can be changed by hand
  - **History Statistics:** shows the past statistics the model uses — lead time per port, port → compound ratio, and lead time distribution. History currently goes up to 2026-04-29
- Reviewed the other Rail ETA screens in the PPT
  - **INTransit:** includes an Excel export example; one route is planned for the demo
  - **Model Matrix:** predicted remaining days by origin → destination and by progress range (ShipOut, 0–25%, 25–50%, 50–75%, 75–100%)
  - **VIN Map:** clicking a location shows the VIN list on the right (from the 11/09 work)
  - **Dashboard:** engine validation 112/112 PASS

**11/11**
- Reviewed the integrated ETA Portal: listed the parts I did not understand, and checked the system and its accuracy
- Reviewed the accuracy of Compound Forecast
  - Compound Forecast shows how many vehicles go to which compound on each date
  - My question: how accurate is this result?
  - Conclusion: with the current structure, accuracy **cannot be validated**
    - No actual distribution data: we do not know how many vehicles really go from the vessel to each compound
    - The ratio is only a past statistic: e.g., if 1,000 vehicles arrive at a port, 45% go to Compound A and 4.5% to Compound B — this is not a fixed number
    - The ratio changes over time
  - My hypothesis: if we can get the actual number distributed from each port to each compound, we can validate the accuracy and also improve it
- Tire warehouse slotting system (v1.1 ~ v1.4)
  - Overview
    - Purpose: a system to manage inbound, outbound, and stock locations in a partner's tire warehouse. The goal is to follow FIFO and reduce forklift travel, and also to study applying reinforcement learning (RL) in practice
    - Tech: Python (FastAPI), Vue 3, Three.js, reportlab (PDF), MaskablePPO (RL)
    - Warehouse model: 4 zones × 25 cells = 100 cells, 20 tires per cell (5 levels × 4), one dock
  - v1.1 — 3D simulator and RL (earlier work)
    - Upload inbound/outbound files → the system places tires by rule and replays 4 forklifts in 3D
    - Greedy placement rule: one size per cell, same-size cells first, otherwise the empty cell closest to the dock. Outbound follows FIFO
    - Built an RL environment, trained for 500K steps, and tested on 30 unseen scenarios
    - Result: RL performed the same as greedy statistically (−0.08%, 93.7% same cell choice)
    - Cause: with no relocation, greedy is already close to optimal, so looking ahead gives almost no room to improve
  - v1.2 — 2D floor plan screen (11/09)
    - A top-down floor plan to handle inbound and outbound directly
    - Current and expected stock at the top
    - Inbound tab: upload a list → assigned cells are calculated in advance; clicking a row highlights the cell
    - Outbound tab: clicking a cell shows a checklist (SKU + DOT + qty); searching a SKU makes its cell blink
    - Post-outbound optimization tab (outgoing cells in orange, incoming cells in blue)
    - 2D/3D toggle, and confirmed work replays with 3D forklifts
    - Decisions:
      - Feedback said there were too many colors → cells not related to the task show only a gray bar (fill level)
      - Hover shows a preview, click pins it (so the popup does not disappear when moving the mouse to check items)
      - If moving costs more than it gains, do not move
  - v1.3 — Redesigned stock location optimization (11/11)
    - Background: the SKU format changed, so I redefined the optimization criteria from scratch
    - Changed the SKU format to `205/55R16-TOUR-A` (only size changes → one SKU = one size)
    - Warehouse size: 1.5m × 1.5m cells, tires laid flat and stacked 2 × 2, 4.5m between zones (based on an average tire diameter of 0.7m)
    - 6 optimization criteria (turnover was removed because it needs outbound history; a move-cost limit was removed because it is the same as minimum travel):
      1. Maximize empty cells
      2. Move filled cells near the dock
      3. Keep FIFO
      4. Reduce mixed DOTs in a cell
      5. Group the same size
      6. Keep empty cells near the dock
    - Up to 4 scenarios, weights in 10% steps that add up to 100%. Two opposite criteria cannot be selected together
    - Comparison table: number of moves, travel distance (relative, "current = 100"), warehouse status metrics, and the floor plan after relocation
    - Flow: check data → scenario → compare → 3D preview (stock does not change, can undo) → confirm → move list PDF
    - Next-job data: use list automatically → ask once → last 30 days of history → sample
    - Inbound/outbound history is saved to a file; removed the forced post-outbound optimization rule
    - Decisions:
      - Stop rule is automatic ("until the score stops improving") instead of a user setting. Time and distance limits were left out because I do not know the real warehouse scale
      - No "recommended" label. Users choose by looking at the numbers in the table
      - RL recommendation is on hold. Criteria must be fixed first to design the reward, and it is hard to explain why RL recommends something
  - v1.4 — Korean/English toggle and easier launch (11/11)
    - Korean/English button (Korean by default, remembered by the browser). Error messages, PDF, and Excel follow the same language
    - Double-click `run_server.bat` → server starts with no window and the browser opens. `stop_server.bat` to stop
    - Split docs into Korean/English: README_KO/EN, RL_GUIDE_KO/EN
  - Validation
    - 36 backend tests passed (each criterion, weight check, preview does not change stock, reset when inbound/outbound happens before confirm, 30-day history, PDF, language)
    - Ran the full flow in the browser for each version and checked with screenshots
    - Auto-check for untranslated Korean on the English screen: 0 found
    - Did not run the `.bat` files on Windows directly (Linux environment, so only `launcher.py` was tested)
  - Findings
    - Comparison period problem: comparing only the next job often makes "no move" look best because relocation cost is high. The benefit of relocation builds up over many inbound/outbound jobs
    - Criteria conflict: grouping the same size cut scattering by 76%, but FIFO violations went from 0 to 61
    - Moves vs distance: the scenario with the fewest moves and the one with the shortest distance can be different → decided to show both
    - RL lesson: measure the headroom (room to improve) before applying RL
  - Remaining tasks
    - Extend the comparison period from the next job to multiple days
    - Retrain the RL model (cell 1.2m → 1.5m, new SKU format)
    - Review applying RL to relocation plans (current greedy plan is the baseline)
    - Test the launcher files on Windows

**11/12**
- Visited the tire warehouse in person
  - Heard how inbound and outbound actually work, and updated the stock location optimization system based on it
- Tire slotting updates from v1.4 to v1.6.3
  - UI changes
  - Added voice recognition: saying "show cell A01" shows the tires and quantities in that cell

**11/13**
- Tire slotting update to v1.7
- Redesigned the outbound tab: the system now guides the cell based on the outbound list
  - Before (v1.6.3): the worker clicked a cell, checked the tires, entered the quantity, and added it to the outbound list by hand
  - After (v1.7): upload the outbound list file → each row shows which cell to pick from (e.g., `B-05 −6 · DOT 2531`)
  - Cell selection rules
    - Urgent rows first → older DOT first (FIFO)
    - If one cell is not enough, split across multiple cells
    - If DOTs are the same, pick the cell with fewer tires first (to create empty cells)
  - If there is an older DOT than requested, the system guides to the older DOT and shows "requested DOT → changed to older DOT" where the old ⚠ mark was
  - If stock is short, it guides only what is available and shows the shortage. The shortage stays in the list after confirm
  - "+ Add list" lets users upload several outbound lists in a row
  - Removed manual cell picking. A cell opened by SKU search is view-only and shows what will be picked from it
  - "Next outbound" in stock location optimization now uses the outbound tab list automatically
  - Backend: added `outbound_plan` with the same structure as inbound `inbound_plan`
- Fixed voice features
  - Infinite loop bug: when both cloud and on-device recognition failed, it kept switching between them (11 times in 4 seconds in testing). Now it tries each once, then stops auto-switching and wake-word mode
  - Fixed the recognition mode switch running twice
  - Added a 1-minute timeout for language pack download
  - On a `service-not-allowed` error, it checks first whether the mic is blocked
  - Added text in the voice panel explaining the difference between wake-word mode on and off
  - Added a "load sample outbound" command. "Read the task list" reads cell, SKU, and quantity, and also reads the message for rows with a changed DOT or a stock shortage
- Other
  - Put DOT 2611 in two sample outbound rows (OUT-006, OUT-009) so the DOT change message shows in the sample
  - Updated README (Korean/English) sections 3-3 and 3-6, API, and version history for v1.7
- Tests
  - Backend tests: 36 → 45, all passed
  - Voice command tests: 58 → 60, all passed

## What I learned (skills/concepts)
- **Find the cause before filling gaps:** for "ETA Unavailable," I first found why it happened (data update timing, route key mismatch), and only then filled it with the average. Knowing the cause also told me the issue should disappear once the logic runs in real time
- **To validate an output, first check if its input can be validated:** when asked "Can we validate Compound Forecast accuracy?", the right follow-up is "Can we even guarantee the vessel → compound accuracy?" That part has no ground truth — it is a statistical average from the past (e.g., 45% to Compound A, 4.5% to Compound B), and it can always change. Asking this question is what it means to fully understand the part
- **Measure headroom before applying RL:** RL matched greedy because greedy was already near optimal without relocation. Check how much room there is to improve before investing in RL
- **Optimization criteria can conflict:** grouping the same size reduced scattering a lot but broke FIFO. Fewest moves and shortest distance can also point to different scenarios, so showing both is better than picking one
- **The comparison window matters:** relocation looks bad if you only look at the next job, because its benefit builds up over many jobs
- **Voice issues are not always hardware issues:** voice recognition behaved differently on different PCs. I need to check step by step — is it a company security policy, is audio coming into the PC, or is audio fine but recognition failing — and fix it for each PC environment

## Challenges / problem-solving
- **Combining Vessel and Rail ETA:** needed to build a Home screen table that shows expected arrivals per compound by date, using both systems. Solved by adding Compound Forecast
- **Explaining each feature's purpose:** I had to understand and explain why each feature in the portal exists. I studied the full flow (shipping line → port → rail → compound) first, and then the project goals made sense
- **Compound Forecast cannot be validated:** there is no actual distribution data. I wrote down a hypothesis: getting real port → compound distribution counts would make validation possible
- **Understanding real warehouse rules:** I did not know the real inbound/outbound rules, so I visited the warehouse and updated the system based on what I heard
- **Voice recognition loop:** cloud and on-device recognition kept switching forever when both failed. Fixed by trying each once and then stopping

## Next week plan
- Check real mic, on-device recognition, and the 1-minute language pack timeout on a company PC
- Retrain the RL model with the new warehouse size (1.5m cells) and SKU format (currently 0.25% longer travel than greedy on average)
- Extend the optimization comparison period from the next job to multiple days
- Test the Windows launcher files on a real Windows PC

## Reflection
- I am glad I did not just fill "ETA Unavailable" with the average right away. I first found the cause and checked it in the system, and I also thought about how the future system would solve it
- Understanding the whole flow (shipping line → port → rail → compound) first helped me understand the purpose of each feature in the portal
- Since I did not yet know how tires should be placed in the warehouse, I did not force one rule. Instead, I let users choose their own scenario. I think this makes the system fit the user better
- Visiting the warehouse in person showed me that real operations should drive the design, and this changed how the outbound tab works (system-guided picking instead of manual picking)
