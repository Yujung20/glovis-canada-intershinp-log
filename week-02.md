# Week 2

## What I Did

**8/10**
- Continued building a Power Automate flow for a cross-team document automation request (extracting data → populating a Word template → converting to PDF); reached ~90% completion.
- Prepared for the weekly Thursday briefing, studying the MS365 ecosystem and Active Directory basics.

**8/11**
- Completed the document automation flow from earlier in the week.
- Started merging two related datasets, validating that the shared key column was unique and consistent before joining.
- Supported a separate cross-team automation flow that takes a user-uploaded file, scrapes required values from a linked source, and fills them in automatically.
- Sat in on a walkthrough of a basic security check procedure (running a local script via PowerShell with admin rights, exporting results to JSON).
- Continued studying MS365 ahead of Wednesday's briefing.

**8/12**
- Delivered the weekly briefing on the MS365 ecosystem and reviewed the feedback afterward.
- Helped set up the file-upload/scraping automation flow on a teammate's machine.
- Worked on resolving a file-access permissions issue affecting the document automation flow.
- Continued the dataset cleanup: validating that location names matched their coordinates, and that the coordinates corresponded to real infrastructure.

**8/13**
- Finished the document automation flow end-to-end after resolving a template-path issue in the "Word Online (Business)" step.
- Attended training on a new settlement/accounting-related program, introduced because an upstream data source had recently started providing the input it needed.

**8/14**
- Completed the first pass of the dataset cleanup, applying a staged set of validation rules (name-to-coordinate match, real-world infrastructure match, region column for disambiguation, and manual review of edge cases).
- Started learning the basics of what SAP is, what accounting teams do, and how to read a public company's disclosure filings.

## What I Learned

- **Word template automation**: To use the "Populate a Microsoft Word Template" action, the template's fields need to be set up as Plain Text Content Controls via the Developer tab, with properties configured in advance to define how each field will be mapped.
- **Splitting flows improves reliability**: I originally tried to build the "generate Word doc" and "convert to PDF" logic as a single flow, which produced frequent errors. Splitting it into two separate flows (Word generation, then PDF conversion) significantly reduced failures. This taught me that breaking a complex flow into smaller, single-purpose flows is often more robust than one large flow.
- **Design with the end user in mind**: I initially planned to give users only the final PDF, but was advised that providing both the Word and PDF versions is more useful, since users often need to edit the document later. This reinforced the importance of interpreting requirements from the user's perspective, not just the technical spec.
- **Environment matters, not just logic that works locally**: A flow that worked fine on my own machine failed for the actual end user because a "Word Online (Business)" step couldn't resolve the template path. I debugged this by testing the path resolution step by step from the root folder down, which helped me isolate exactly where it failed.
- **SharePoint paths inside Power Automate are not "what you see"**: Power Automate resolves SharePoint locations through its own internal referencing system rather than the path visible in the UI. If underlying paths change over time, old references can become stale or point to the wrong place — this is part of why understanding the platform's internals (not just the individual actions) matters.
- **Communicating technical work to a non-technical audience**: When presenting on MS365 to the team, I learned to explain things in terms of what the tool is for and why the company built/adopted it, rather than diving into feature details first. Starting from purpose makes the technical parts easier to follow.
- **Careful, staged data cleaning**: For the dataset merge work, instead of deleting/altering data ad hoc, I applied validation in stages — first checking that names matched coordinates, then checking that coordinates matched real-world infrastructure, then adding a region/state column to disambiguate duplicate names, and finally reviewing edge cases (same coordinates but different naming conventions, or similar names referring to genuinely different locations). Treating this as a prioritized, staged process felt like the right approach.

## Challenges

- Debugging the "Word Online (Business)" path issue took a lot of trial and error — I didn't yet have enough understanding of how the platform resolves SharePoint paths internally, so my first instinct (just fixing the visible path) wasn't enough on its own.
- Data cleaning surfaced far more edge cases than my initial rule set accounted for, requiring manual, case-by-case review rather than a single automated pass.
- I still feel I don't have a strong grasp of how new systems (like the accounting/settlement tool or SAP) fit into the bigger picture — I tend to learn the tool in isolation before connecting it to why the company/department needs it.

## Next Week Plan

- Study the basics of accounting workflows, SAP, and how different departments' roles connect to them.
- Learn more about EDI (Electronic Data Interchange), which came up this week as a prerequisite for understanding a settlement process.
- Use Power BI to visualize the cleaned location dataset on a map, and compare it against real-world infrastructure paths to validate the cleaning work.

## Reflection

This week's biggest lesson was about thinking beyond "does it work" toward "does it work for the actual user, in the actual environment." The Word/PDF automation succeeded on my machine early on, but real deployment surfaced problems I hadn't anticipated — mainly because I didn't yet understand how the underlying platform (SharePoint referencing, template resolution) actually works. I also noticed a pattern across the week: whether it was designing a flow, presenting to teammates, or learning a new business system, I get more out of it when I start by asking "what is this actually for, and who is it for" before diving into the mechanics. That's something I want to keep applying more deliberately going forward, rather than defaulting to it only after running into friction.
