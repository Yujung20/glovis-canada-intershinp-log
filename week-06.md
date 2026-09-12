# Week 6 (2026-09-08 ~ 2026-09-11)
---

## What I did this week
**09/08**
- Hackathon: organized the concept and purpose of the contest, and started trying to understand the ML model (XGBClassifier) behind it
- VTP: preprocessed data to prepare it for merging with the previous dataset
- RPA: explored using AI for the task, but found it would consume a lot of credit, so started looking for an alternative approach

**09/09**
- Prepared intern presentation materials
  - RPA: framed the presentation around why the work matters (contribution to the IT team's key tasks), gave an overview of what RPA is, explained what POD is and which fields are extracted, and reworked the As-Is vs. To-Be comparison — the task was reduced from 2 minutes 30 seconds to 0 seconds; the previously cited 25.8 seconds was actually the automated flow's execution time, not the time a person actually has to spend, so the 0-second framing was emphasized instead
  - RPA (second task): explained why it's being done, the expected results, and the planned approach
  - VTP: explained what VTP is, why it's being done, and included the data preprocessing work
- RPA: noted volume context for the presentation — TGT arrives at 200–600 cases per month, with every 200 cases taking about 30 minutes; also noted 91 PDFs received as of August

**09/11**
- Continued preparing the intern presentation: the overall structure is complete, and details are being refined based on feedback; added a short explanation to each page so its meaning is clear at a glance while staying brief; decided to calculate and add the annual time saved for the RPA task to better show the productivity improvement
- Hackathon: built the program from the given data using vibe coding, and identified the need to understand which features the model used, how it was built and used, and how the risk score was calculated

## What I learned (skills/concepts)
- XGBClassifier is a binary classification model whose raw output is 0 or 1, but generating an actual risk score requires `predict_proba()` instead of `predict()`, since it returns a continuous probability between 0 and 1
- The threshold a model uses isn't set manually — it's found automatically during training. Because XGBoost combines dozens to hundreds of trees through boosting, each splitting at different thresholds, a single tree behaves like a step function, but the combined output becomes a smooth, continuous probability
- This clarified the structure of the current pipeline: independent XGBClassifier models for features a, b, c, and d each output a probability via `predict_proba()`, and these are combined through a weighted sum into a final risk score — using fine-grained probabilities rather than 0/1 outputs produces a richer, more detailed score distribution
- In the hackathon, one of the four independent XGBClassifier models (used for HOS) returned an ROC of 1, meaning it had essentially memorized the training data rather than learned a generalizable pattern. As a result, HOS was excluded from the risk score calculation and used instead as a rule-based layer

## Challenges / problem-solving
- Finding a way to extract data from a PDF without relying on AI, and considering whether a Python-based system needs to be built instead
- Mapping Origin City and Destination City to their corresponding code numbers for VTP
- Handling VTP's CN/CP data, which arrives fresh each time rather than in a form that can be preprocessed the same way every time

## Next week plan
- Continue investigating PDF data extraction without AI, and decide whether a Python-based extraction system is needed
- Finish mapping Origin City / Destination City to code numbers for VTP
- Work out a consistent approach for handling CN/CP data that arrives fresh on each VTP update
- Finalize and deliver the intern presentation, incorporating the annual time-saved framing for the RPA task
- Keep building understanding of the hackathon model's features, construction, and risk score calculation

## Reflection
- Noticed a tendency to ask questions rather than work through problems independently — want to dig deeper into the model itself instead of defaulting to asking
- While working on VTP, should avoid creating too many query statements, and if one is created, should clearly know its exact purpose
