# Week 5 (2026-08-31 ~ 2026-09-04)
---

## What I did this week
**08/31**
- Organized the nodes to be added to the routing/geographic dataset
- Visualized the data on the map and checked whether the filter was working correctly
- Studied the data architecture needed to request authorization for the CPKC API (covering bulk train line-up, bulk train consist, carload track-and-trace equipment list, and related detail-level endpoints)
- Prepared for the upcoming internship presentation, including a Gantt chart of completed and in-progress work

**09/01**
- Continued preparing the internship presentation and drafted a PPT outline covering RPA, VTP, and the hackathon
- For RPA, built an As-Is vs. To-Be comparison showing a significant reduction in manual task time after automation
- For VTP, drafted an explanation of what the project is and its purpose
- Researched the hackathon host organization's motivations, judging criteria, existing assets, and what kind of solution they're looking for
- Received a new RPA-related task and organized its requirements

**09/02**
- RPA: organized requirements for automating tax-related invoice processing — emails arrive automatically in a shared inbox used by one person, need to be converted into Excel regardless of source format, and cover four tax types; designed an Excel output format (Invoice Number, Origin, Destination, Delivery Date, VIN, Tax, Tax Type, Total Cost)
- Hackathon: reviewed prototypes already built by the team lead and studied the released datasets
- Explored AI Builder in Power Automate for document extraction

**09/03**
- Attended the week's briefing to prepare for the intern's first presentation and received feedback on the overall flow:
  - RPA: keep the current page, but add an outline page explaining scope of responsibility and background (e.g., what POD is) before diving into detail
  - VTP: currently relying on vendor-provided ETA; the goal is to estimate ETA independently for full rail coverage, and eventually extend this to truck logistics as well
  - Hackathon: reframe the effort's name/framing to emphasize research rather than competition
- RPA: iterated on the AI Builder model after accuracy plateaued — traced the issue to overfitting on an invoice-type model, then rebuilt using a general-document model with single-field extraction (which improved accuracy), and finally combined multiple single fields into a table to extract multiple orders from one invoice

**09/04**
- Completed an initial AI model for the RPA task but hit a service limitation, requiring a redesign using a different document-processing connector
- Began exploring the new connector's text-extraction capability, which looks promising early on
- Received a new VTP dataset and started thinking through how to incorporate it into the existing visualization
- Ran into an issue where the attachment-type filter for the automation didn't fully exclude non-PDF files as intended

## What I learned (skills/concepts)
- When presenting to a non-technical audience, tailor explanations to what they can actually follow, keep the presentation concise, and go into detail only if asked
- Sketch out the content and structure of a presentation before building the slides
- When entering a research/competition-style initiative, first understand the judging criteria, sponsor motivations, and existing assets before shaping a concept
- AI Builder in Power Automate behaves similarly to supervised machine learning — it trains on sample data and applies the learned pattern to new documents; the choice of model type (e.g., invoice-specific vs. general document) meaningfully affects accuracy and overfitting risk
- How an initiative is framed and named can change how it's perceived, independent of the underlying work

## Challenges / problem-solving
- Getting authorization for the CPKC API is taking time; a request has been submitted and dataset validation is pending approval
- Coming up with a concept for a data-driven initiative was difficult due to limited domain knowledge in logistics, which was addressed by researching the released datasets
- An AI Builder model's accuracy plateaued; root-caused to overfitting from training an invoice-type model on too many self-generated values, resolved by switching to a general-document model
- An email-attachment filter intended to process only one file type let other formats through unexpectedly
- Hit a service limitation partway through building an AI model, requiring a redesign with a different tool

## Next week plan
- Continue building out the RPA automation using the new document-processing connector
- Fix the attachment-type filtering logic so only the intended file format is processed
- Incorporate the newly received dataset into the existing visualization
- Follow up on external API authorization and validate the dataset once access is granted
- Keep refining presentation materials based on this week's feedback

## Reflection
- Realized I had been executing tasks without fully understanding the project behind them, and should ask for context upfront rather than waiting until it's explained
- Only after receiving a direct explanation did the full purpose and expected benefits of the main visualization project really click — a reminder to clarify scope early rather than assuming it will become clear later
- Reminded myself not to put excessive pressure on presentation performance
