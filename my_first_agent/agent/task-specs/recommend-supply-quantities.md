# Recommend Supply Quantities Task Specification

## Basic Information

- **Task ID:** T6
- **Task name:** Recommend Supply Quantities
- **Task type:** Reason
- **Task owner:** HackTrack; the CPVC event lead organizer approves the plan in H3.
- **Automation level:** L2

## 1. Task Description

T6 converts the organizer-approved planning headcount into a recommended supply plan for food, drinks, and swag. A fixed function applies CPVC's per-person ratios and a stated buffer: pizza at 3 slices per person (rounded up to whole pizzas of 8 slices), 2 drinks per person, 1 swag item per person, snacks at 1.5 servings per person, plus a 10% buffer on food and drinks (no buffer on swag). A language model then writes a short plain-language explanation of the plan and flags items that exceed the event budget or the venue capacity; the model does not change any quantity. T6 only recommends. It never places orders or spends club money; H3 approves the plan.

## 2. Inputs

### Input 1

- **Input name:** Approved planning headcount
- **Contents and format:** Event ID, run ID, forecast version, approved headcount (whole number), adjustment reason, and approver.
- **Source:** H2: Organizer Reviews Forecast.

### Input 2

- **Input name:** Supply ratios and budget
- **Contents and format:** CPVC's saved per-person ratios, buffer percentages, unit prices, and the event supply budget.
- **Source:** CPVC event planning sheet, maintained by the CPVC event lead organizer.

### Input 3

- **Input name:** Event details
- **Contents and format:** Event ID, date, venue capacity, and supply-order deadline.
- **Source:** T1: Retrieve Registration Records.

### Input 4

- **Input name:** Supply plan feedback
- **Contents and format:** Optional organizer comments requesting changes, such as a different pizza count or a vendor price change. Absent on the first pass.
- **Source:** H3: Organizer Approves Supply Plan.

- **If a required input is missing or invalid:** Without an approved headcount, T6 does not run; the case stays at H2. If ratios, prices, or budget are missing, T6 produces quantities only, marks cost "not calculated," and flags the gap for H3: Organizer Approves Supply Plan.

## 3. Outputs

### Output 1

- **Output name:** Recommended supply plan
- **Contents and format:** Table of item, per-person ratio, base quantity, buffer, recommended quantity, unit price, and estimated cost; total cost vs. budget; a short explanation; any flags (over budget, headcount above venue capacity, missing prices); plan version number.
- **Next task or recipient:** H3: Organizer Approves Supply Plan.
- **Complete when:** Every item has a whole-number quantity calculated by the function, the plan is saved to the event planning sheet, and the save is confirmed by read-back.

## 4. Planned Tools

### Tool 1

- **Tool name:** calculate_supply_quantities
- **Input:** Approved planning headcount; Supply ratios and budget; Supply plan feedback
- **Output:** Recommended supply plan
- **Implementation Route:** Functions/scripts applying the fixed ratios, buffers, rounding, and unit prices.
- **Integration approach:** Direct integration
- **Role in this task:** Calculates every quantity and cost. Organizer feedback can change a ratio or quantity only as the organizer stated.
- **Task timeout:** 2 minutes total for one T6 run across all tools and model calls; this tool may take at most 10 seconds.
- **Maximum retries:** 0
- **Retry only when:** Not applicable.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "supply plan not calculated" with the error and send it to H3: Organizer Approves Supply Plan so the organizer can plan manually. No quantities are guessed by the model.

### Tool 2

- **Tool name:** explain_supply_plan
- **Input:** Recommended supply plan; Event details
- **Output:** Recommended supply plan (explanation and flags added)
- **Implementation Route:** Web API calls to the Groq API (model `openai/gpt-oss-120b`, the same model as T5), limited to 2 inference requests per run.
- **Integration approach:** Direct integration
- **Role in this task:** Writes a 3–5 sentence explanation and flags budget or capacity concerns. It cannot change quantities, costs, or the headcount; the saved plan always uses the function's numbers.
- **Task timeout:** 30 seconds per request, within the 2-minute task limit.
- **Maximum retries:** 1
- **Retry only when:** A timeout or temporary API error occurs. Wait 5 seconds and retry once. The call does not change any record, so retries cannot create duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Save the plan with the explanation marked "not generated" and send it to H3. The quantities are still valid; the missing explanation is shown as an unresolved issue.

### Tool 3

- **Tool name:** save_supply_plan
- **Input:** Recommended supply plan
- **Output:** Recommended supply plan (saved and confirmed)
- **Implementation Route:** Web API calls to the CPVC event planning sheet (Google Sheets API) that write one plan version.
- **Integration approach:** MCP integration
- **Role in this task:** Saves the plan so H3 reviews a fixed version.
- **Task timeout:** 20 seconds per call, within the 2-minute task limit.
- **Maximum retries:** 2
- **Retry only when:** A temporary error occurs. Wait 10 seconds and read the sheet for the same event ID, run ID, and plan version; if the row exists, treat it as saved. This unique key prevents duplicate plans. If the outcome is uncertain and the read-back fails, stop retrying.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "supply plan not saved," include the full plan in the handoff to H3: Organizer Approves Supply Plan, and mark the run operationally incomplete until the save is confirmed.
