# Week 1 (2026-08-04 ~ 2026-08-07)

## What I did this week

**8/4 — Onboarding**

**8/5**
- Learned the basics of Power Automate — specifically how to build a flow that triggers on receiving an email and sends a reply email
- Explored the Power Automate basic web view
- Got an overview of Hyundai Glovis's internal systems

**8/6**
- More Power Automate fundamentals
- Realized that understanding Power Automate flows requires a solid grasp of MS365 itself (E3 license), including:
  - Teams
  - Outlook
  - OneDrive
  - SharePoint
  - Power Platform
- Studied the basics of each of these services

**8/7 (Work from home)**

*Morning*
- Organized an automation request from another team received the day before: fill a POD (Proof of Delivery) received by email into a Word template, then convert it to PDF
- Thought through how to automate this within MS365:
  - How to set up the trigger
  - How to extract table data from the email body
  - How to insert the extracted data into the Word template
  - How to convert the Word file to PDF
- Got a brief introduction to Power BI, for building dashboards for upcoming projects — including how to join and preprocess data, which means I'll need to study Power Query

*Afternoon*
- Met with the team to discuss the automation idea from the morning
- Learned that the **"Populate a Microsoft Word Template"** action can be used to fill data into a Word document
- Before adding that action to the flow, decided to think more carefully about how to extract the table data from the email body first

## What I learned (skills/concepts)
- Power Automate basics: triggers and actions, connecting an email trigger to an email-send action
- MS365 ecosystem overview (E3 license): Teams, Outlook, OneDrive, SharePoint, Power Platform
- What a POD (Proof of Delivery) is in logistics, and why it matters
- Power BI basics, and the need to learn Power Query for data joining/preprocessing
- The "Populate a Microsoft Word Template" action in Power Automate

## Challenges / problem-solving
- Still working out the best way to reliably extract table data from an email body
- Recognizing how much MS365 platform knowledge is a prerequisite before Power Automate flows really make sense

## Next week plan
- Continue researching how to extract table data from an email body
- Add the "Populate a Microsoft Word Template" action to the flow once the extraction approach is settled
- Study Power Query further to prepare for Power BI dashboard work

## Reflection
- I realized I tried to jump into actual tasks before really understanding the underlying systems
- I didn't know that MS365 is the standard/default toolset used in North America (US, Canada, etc.), which shows how little I actually understood about the systems I was working with
- Starting next week, I want to study more MS365 services in depth to build a stronger foundation before diving into automation work
