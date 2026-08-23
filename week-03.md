# Week 3 (2026-08-17 ~ 2026-08-21)
---

## What I did this week
**08/17**
- Attended a training session on using AI within Power Automate, revisiting the earlier POD (Proof of Delivery) email table-extraction work: built a prompt to load email content, restructured the flow with Try-Catch for better error visibility, and explored sending a Teams notification on success.
- Received an explanation of EDI (Electronic Data Interchange).
- Reviewed feedback on the Week 1 briefing deliverable.
- Power BI dashboard: validated the City Master dataset against official US and Canada rail data, corrected City Master points that deviated from the actual rail lines, and saved the US and Canada files separately.
- Followed up on a workflow shared with the requesting team; still needs further follow-up.

**08/18**
- Finished the AI-connected POD workflow: completed the Try-Catch structure, used the "Run a Prompt" action, wrote a prompt to extract the email body, and studied how to send Teams notifications.
- Followed up on the previously automated POD workflow — an email had come in but the flow failed to trigger correctly; identified that the connection needed to be switched to the requester's account, and planned to keep monitoring this during the week.
- Continued City Master data work: updated city latitude/longitude values, checked US, Canada, and Mexico data, and removed points that fell outside the expected path.

**08/19**
- Wrapped up the Week 1 briefing materials and organized the feedback received.
- Followed up on the POD automation flow: it failed because the source email template had changed, so the requesting team asked for an alternative approach — started building a second version of the flow.
- For the new version, was advised to sketch out a flow architecture diagram before building, and to update it whenever a new idea came up. Identified three possible approaches (convert HTML to plain text, use regular expressions, or use a script-based method) and planned to test each one.

**08/20**
- Continued building the second version of the POD flow, coordinating with the original requester.
- Since the source email was written in HTML, needed a reliable way to convert it into a format Power Automate could parse, especially since the table's column order wasn't always consistent.
- Initial approach (converting HTML to plain text) proved too complex, since removing all the HTML table tags (`<table>`, `<tr>`, `<td>`, etc.) manually would make the flow unwieldy.
- Switched to an Excel-based approach: created an Excel file and wrote a script that extracts the `<table>` section, splits it into rows by `<tr>`, splits each row into cells by `<td>`, treats the first row as headers, and converts the remaining rows into JSON. This approach correctly parsed the data even when the column order changed.
- Worked on fixing an existing reporting program built by the IT team; had to first get it running properly on my own laptop before addressing the underlying issue.

**08/21**
- City Master data: added new columns to distinguish cities that share the same name — a country code, a province/state code, and a combined city key used as the merge key between the two datasets — then validated the data accordingly.
- Directed the AI to flag any address that looked uncertain when compared against the city name and location, then manually reviewed each flagged case.
- Followed up on the second version of the POD flow; it has been running successfully and consistently, and no longer needs close monitoring.

## What I learned (skills/concepts)
- Learned how to connect AI directly within a Power Automate flow using the built-in "Run a Prompt" action, rather than calling an external API separately.
- Learned that Try-Catch is available in Power Automate as an alternative to building error handling with Condition actions — a pattern worth reusing in future flow designs.
- Learned the differences between geographic data formats: shapefiles (a vector format that can precisely represent points, lines, and polygons — well suited to continuous paths like rail lines, but split across multiple files and typically distributed as a zip; coordinate systems can be explicitly set via a `.prj` file); GeoJSON (a single JSON-based text file combining geometry and attributes, human-readable, but can grow large for dense line data since coordinates are stored as raw numbers, and is generally fixed to WGS84); and CSV (plain rows/columns, which can only represent lines by embedding geometry as Well-Known Text in a text column).
- Learned the differences between Excel and CSV: file structure (CSV is plain comma-separated text, while Excel is an XML-based format that can also store formatting, formulas, charts, and other metadata), sheet structure (CSV has no concept of multiple sheets, while Excel supports several in one file), and formula handling (CSV loses formulas and cell/conditional formatting on save, while Excel retains them).
- When writing prompts for AI, instructions need to be explicit — missing detail can lead the AI to a different result than intended.
- When building automation workflows, parameters need to be clearly defined, and when an error occurs, it should be traced starting from the end of the flow backward.
- Learned that Excel has far more built-in scripting capability than expected, via the "Run Script" action used to parse HTML tables reliably.
- Discovered that duplicate city names exist between the US and Canada, which required adding a country code (not previously present in the dataset) to disambiguate them.
- Drawing out a flow's architecture before building it helps organize my thinking up front and gives me something to return to when I get confused partway through.

## Challenges / problem-solving
- The US rail dataset (about 275,000 rows) repeatedly crashed Power BI; resolved by splitting the US and Canada rail data into separate files.
- Connecting the City Master data properly within Power BI has been difficult: cargo location isn't tracked in real time, but a data point is captured each time cargo passes a rail station, so all the stations a shipment passes through need to be linked together in sequence — still researching how to do this in Power BI.
- The EDI training introduced a lot of unfamiliar terminology (SAP, SOAP, FTP, etc.) that needs separate, focused study.
- Comparing internal data against real-world rail paths was difficult without a clear reference, so official rail datasets from each country were used in Power BI for validation; this required first understanding the differences between the available geographic data formats to choose the right one.
- Judging whether cleaned data was actually correct was difficult without first-hand knowledge of Hyundai Glovis's real rail routes — some cases were uncertain enough that I had to check with the team lead, which highlighted the need to better understand how the logistics side of the business actually works.
- Version 2 of the POD flow was significantly more complex than version 1: Outlook emails are written in HTML, which Power Automate doesn't parse well by default, and the source table's column order isn't fixed, so a hard-coded extraction approach couldn't be reused. Since the table data still follows consistent per-column rules, the core challenge became matching each cell to the right field despite the reordering — solved with an Excel script rather than plain HTML-to-text conversion.

## Next week plan
- Continue following up on the POD automation flow to confirm it keeps running reliably.
- Study SAP, EDI, and related enterprise system terminology (SOAP, FTP, etc.) more deeply to close the knowledge gaps from this week's training.
- Work on linking sequential rail-station data points in Power BI so cargo movement can be traced along the actual rail path.
- Continue merging the two City Master datasets using the newly added city key, country code, and province code.
- Resolve the "On Water" reporting program issue after getting it running locally.

## Reflection
- Hearing a lot of unfamiliar computer science and networking terminology (SOAP, XML, etc.) without a solid grasp of what each one actually means was frustrating this week — I want to study computer science fundamentals alongside the day-to-day internship work going forward.
- It was hard to judge whether preprocessed data was actually valid without knowing Hyundai Glovis's real rail paths firsthand; I realized I need a better understanding of how the logistics side of the business works to make good judgment calls on this kind of data project.
- Learned to try solving a problem myself first, in multiple ways, before turning to AI or asking colleagues — research first, then use AI as a next step rather than a first resort.
- Sketching out a flow's architecture before building it turned out to be genuinely useful, both for organizing my own thinking and as something to fall back on when I got stuck mid-build.
