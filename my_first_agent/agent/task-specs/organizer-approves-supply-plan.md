# Organizer Approves Supply Plan Task Specification

## Basic Information

- **Task ID:** H3
- **Task name:** Organizer Approves Supply Plan
- **Task type:** Decide
- **Task owner:** CPVC event lead organizer (with the CPVC treasurer when the plan exceeds the event budget).
- **Automation level:** L0

## 1. Task Description

H3 is the spending checkpoint. The organizer reviews T6's recommended supply plan against the approved headcount, budget, vendor details, and any flags, and decides whether it is ready. They either approve a specific plan version or request changes with comments, which sends the plan back to T6. If the plan is over budget, the CPVC treasurer must also approve. Approval means the club may order from this plan; the organizer places the orders themselves. HackTrack never places orders or spends money, and approval of one version does not approve a later version.

## 2. Inputs

### Input 1

- **Input name:** Recommended supply plan
- **Contents and format:** Item table with quantities, buffer, costs, total vs. budget, explanation, flags, and plan version.
- **Source:** T6: Recommend Supply Quantities.

### Input 2

- **Input name:** Approved planning headcount
- **Contents and format:** Approved headcount, forecast version, and adjustment reason.
- **Source:** H2: Organizer Reviews Forecast.

### Input 3

- **Input name:** Organizer supply decision
- **Contents and format:** Human response: approve (with the plan version) or request changes (with comments); treasurer approval when over budget.
- **Source:** CPVC event lead organizer and, when needed, the CPVC treasurer.

- **If a required input is missing or invalid:** If the plan was not calculated or not saved, the organizer may write a manual plan, recorded as "manual plan, no HackTrack recommendation." An approval without a plan version, or an over-budget approval without the treasurer, is not valid and the plan stays pending.

## 3. Outputs

### Output 1

- **Output name:** Approved supply plan
- **Contents and format:** Event ID, run ID, approved plan version, approver(s), approval time, and any organizer notes.
- **Next task or recipient:** CPVC organizers for purchasing (done by people, outside HackTrack); the event planning sheet; the run then waits for the event and T7: Record Actual Attendance.
- **Complete when:** The organizer's approval is saved and linked to one plan version, with treasurer approval recorded when required.

### Output 2

- **Output name:** Supply plan feedback
- **Contents and format:** Plan version reviewed and the organizer's requested changes.
- **Next task or recipient:** T6: Recommend Supply Quantities.
- **Complete when:** The comments are saved and linked to the reviewed version.

## 4. Planned Tools

### Tool 1

- **Tool name:** record_organizer_decision
- **Input:** Recommended supply plan; Approved planning headcount; Organizer supply decision
- **Output:** Approved supply plan; Supply plan feedback
- **Implementation Route:** Web API calls that display the plan on the HackTrack organizer page and append the decision to the event planning sheet (Google Sheets API).
- **Integration approach:** MCP integration
- **Role in this task:** Shows the plan and records the decision. It does not approve, change quantities, contact vendors, or place orders.
- **Task timeout:** Human response deadline: 24 hours after the plan is saved, or 2 days before the supply-order deadline, whichever comes first.
- **Maximum retries:** Not applicable — manual task.
- **Retry only when:** Not applicable.
- **On timeout, exhausted retries, or an error that cannot be retried:** If the deadline passes, mark the plan "approval overdue" and send it to the CPVC club president as backup organizer. A missed deadline is not approval, and no plan is marked approved. If saving the decision fails, read the sheet to check; if the decision is not there, show the error and keep the plan pending until the organizer resubmits.
