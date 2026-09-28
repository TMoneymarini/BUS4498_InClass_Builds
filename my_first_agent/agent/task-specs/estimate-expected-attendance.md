# Estimate Expected Attendance Task Specification

> This task is Level 3 because intermediate findings determine what the agent does next: it may summarize responses, choose which historical show-up rates apply, examine a data anomaly, incorporate an organizer note, or form the forecast. Those choices are bounded, and unresolved cases go to a CPVC organizer.

```yaml
# BASIC INFORMATION
task_id: "T5"
task_name: "Estimate Expected Attendance"
task_owner: "HackTrack; the CPVC event lead organizer approves the final planning headcount"

# Agent Inference Configuration
Provider: Groq
Model: "openai/gpt-oss-120b"
Role: Interpret the response summary and attendance history, select the next permitted subtask, explain data anomalies, and write the plain-language forecast explanation. The model does not calculate the headcount; the calculate_attendance_range tool does.
Maximum inference requests per task run: 6
On inference failure or exhausted limits: Record the unresolved status and hand the case to the CPVC event lead organizer.
```

## 1. Task Goal

- **Objective:** Produce an evidence-backed expected attendance headcount for the AI Hackathon with a low–high range and a short confidence note, based on the validated roster, confirmation responses, and CPVC's prior attendance history. The forecast must explain which show-up rates were used and why, so an organizer can approve or adjust the planning headcount in H2. If the evidence is too thin or contradictory for a supported number, explain what is missing instead of inventing an estimate.

## 2. Inbound Inputs

### Input 1

- **Input name:** Clean registration roster
- **What it contains:** One record per validated registrant with registrant ID, registration date, registration type (individual or team), team size when stated, and validation status. Duplicate and test registrations are already removed. Contact details are not needed by this task and are not passed to it.
- **Source:** T2: Validate Registration Data

### Input 2

- **Input name:** Confirmation response log
- **What it contains:** The latest confirmation response per registrant ID (Yes, Maybe, No, or no response), the response timestamp, which message round it answered (first request or reminder), and opt-outs. Includes the overall response rate and the time the log was closed.
- **Source:** T4: Record Confirmation Responses

### Input 3

- **Input name:** Prior event attendance history
- **What it contains:** For each past CPVC build event: registrations, check-ins, and, when available, check-ins broken out by confirmation response (Yes, Maybe, No, no response). The last build event shows roughly 40% overall attendance-to-registration. A valid history may lack the response breakdown; that is different from an unreadable file.
- **Source:** T1: Retrieve Registration Records

### Input 4

- **Input name:** Event details
- **What it contains:** Event ID, date, start time, venue capacity, supply-order deadline, run ID, and forecast version number.
- **Source:** T1: Retrieve Registration Records

### Input 5

- **Input name:** Organizer forecast note
- **What it contains:** An optional note from H2 when an organizer requests a re-forecast, such as a known class conflict, a partner club bringing a group, or a request to explain a specific number. This input is absent on the first forecast.
- **Source:** H2: Organizer Reviews Forecast

## 3. Tool Permissions and Boundaries

*Name each planned tool and specify its permitted use. Use verb-object names, such as **`retrieve_records`**, usually matching the task or permitted subtask it supports. Tool name identifies the capability; tool type identifies the proposed implementation. No scripts or working integrations are required.*

### Task-Wide Limits

- **Total task timeout:** 5 minutes of elapsed time for one task run, including all tool calls, retries, waiting, and inference requests. A retry does not restart this clock.
- **Maximum tool calls:** 10 calls across all tools during one task run; retries count toward this total. Inference requests are counted separately under the Agent Inference Configuration but share the same 5-minute budget.

Tools may use only the supplied inputs for this event. They may not message registrants, change roster or response records, view contact information, place supply orders, or approve a planning headcount. The only write this task may perform is saving its own forecast record.

### Tool 1

- **Tool name:** `retrieve_roster_and_responses`
- **Input:** Clean registration roster; Confirmation response log
- **Output:** Response counts by category (Yes, Maybe, No, no response), team-size adjustments, and response rate for the Evidence summary; missing, stale, or conflicting records for Unresolved issues.
- **Implementation Route:** Database queries (read-only) against the event's roster and response tables, keyed by event ID.
- **Integration approach:** Direct integration
- **Role in this task:** Supports Summarize Confirmation Responses and Examine Data Anomalies.
- **Task timeout:** Each call may take at most 15 seconds or the remaining task time, whichever is shorter.
- **Maximum retries:** 1
- **Retry only when:** A temporary connection or read error occurs. Wait 5 seconds and retry once if enough time and call budget remain. Do not retry denied access or a missing table. This tool is read-only, so a retry cannot create duplicate records or messages.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record the failed read, error category, and attempts in Subtasks performed and Unresolved issues. Set Status to "Escalated to organizer — operational error," set Result to "undetermined," and hand the case to the CPVC event lead organizer. Do not treat unreadable responses as "no response."

### Tool 2

- **Tool name:** `retrieve_attendance_history`
- **Input:** Prior event attendance history; Event details
- **Output:** Historical show-up rates overall and by response category, with the source event for each rate, for the Evidence summary; missing breakdowns for Unresolved issues.
- **Implementation Route:** File operations (read-only) on the CPVC event history spreadsheet.
- **Integration approach:** Direct integration
- **Role in this task:** Supports Select Historical Show-Up Rates.
- **Task timeout:** Each call may take at most 15 seconds or the remaining task time, whichever is shorter.
- **Maximum retries:** 1
- **Retry only when:** A temporary file read error occurs. Wait 5 seconds and retry once if time and call budget remain. Do not retry a confirmed missing file. Read-only, so retries create no duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record the failure in Unresolved issues and continue only with the documented default rates in Section 4, clearly labeled as assumptions. If the history file is missing entirely, set Status to "Escalated to organizer" and ask the organizer to confirm the default rates before a forecast is used.

### Tool 3

- **Tool name:** `calculate_attendance_range`
- **Input:** Clean registration roster; Confirmation response log; Prior event attendance history
- **Output:** Expected headcount, low–high range, and the calculation behind each for the Result or recommendation and Evidence summary.
- **Implementation Route:** Functions/scripts that apply the fixed forecast rule in Section 4 to the response counts and selected show-up rates. The model supplies no numbers to this function other than the rates it selected and cited.
- **Integration approach:** Direct integration
- **Role in this task:** Supports Form Attendance Forecast.
- **Task timeout:** Each call may take at most 10 seconds or the remaining task time, whichever is shorter.
- **Maximum retries:** 0
- **Retry only when:** Not applicable. A recalculation after new evidence or an organizer note is a new invocation, not a retry, and must stay within Section 4's repetition limits and the task-wide budgets.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record the failed calculation and its inputs in Unresolved issues, set Status to "Escalated to organizer — operational error," and set Result to "undetermined." Do not estimate a number by hand or by the model.

### Tool 4

- **Tool name:** `save_forecast_record`
- **Input:** Event details (event ID, run ID, and forecast version) plus the completed forecast
- **Output:** Saved-record confirmation and record link for Status and Next task or recipient.
- **Implementation Route:** Web API calls to the CPVC event planning sheet (Google Sheets API) that append one forecast row.
- **Integration approach:** MCP integration
- **Role in this task:** Supports Form Attendance Forecast by persisting the result so H2 can review it.
- **Task timeout:** Each call may take at most 20 seconds or the remaining task time, whichever is shorter.
- **Maximum retries:** 2
- **Retry only when:** The save returns a temporary error (such as a rate limit or 5xx response). Before each retry, wait 10 seconds and read the sheet for a row with the same event ID, run ID, and forecast version. If that row already exists, treat the save as successful and do not write again. Each row uses this combination as a unique key, so a retry can never create a duplicate forecast. If the outcome is uncertain and the read-back fails, do not retry; hand off.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "Forecast not saved" with the attempts and error in Unresolved issues, set Status to "Escalated to organizer — operational error," and include the full forecast in the Handoff note so it is not lost. Do not report the forecast as saved or send it to T6.

## 4. How the Agent Should Reason

### Permitted Subtask 1

- **Subtask name:** Summarize Confirmation Responses
- **Subtask description:** Count validated registrants by latest response (Yes, Maybe, No, no response), apply stated team sizes, and report the response rate and how many answers came from the reminder round.
- **Subtask boundary:** Use only the Clean registration roster and Confirmation response log. Do not guess a response for anyone, re-contact registrants, or look at contact details.
- **Retry limits:** Perform once. Repeat once only if a re-forecast supplies an updated response log.

### Permitted Subtask 2

- **Subtask name:** Select Historical Show-Up Rates
- **Subtask description:** Choose the show-up rate for each response category from the most recent comparable CPVC event with a response breakdown. If no breakdown exists, use the documented defaults below and label them as assumptions.
- **Subtask boundary:** Use only Prior event attendance history and the defaults in this spec. Do not invent rates, use outside data, or change a rate to hit a preferred number.
- **Retry limits:** Perform once. Repeat once only after an organizer note gives a documented reason to change a rate.

### Permitted Subtask 3

- **Subtask name:** Examine Data Anomalies
- **Subtask description:** Check for issues that could make the forecast unreliable: a response rate under 30%, a late surge of registrations after messages went out, a large team registration, conflicting responses from the same registrant, or an expected headcount above venue capacity.
- **Subtask boundary:** Flag and explain anomalies only; do not delete records or change responses.
- **Retry limits:** Examine no more than two distinct anomalies before handing the case to the organizer.

### Permitted Subtask 4

- **Subtask name:** Incorporate Organizer Note
- **Subtask description:** When an Organizer forecast note is supplied, identify what it changes (for example, adding an expected group of 15 from a partner club) and apply it as a separately labeled adjustment.
- **Subtask boundary:** Apply only adjustments the organizer stated. Do not extend the note to other registrants or treat it as a registrant response.
- **Retry limits:** Incorporate the note once per task run. A further note starts a new re-forecast run through H2 (at most one re-forecast before the supply-order deadline).

### Permitted Subtask 5

- **Subtask name:** Form Attendance Forecast
- **Subtask description:** Use `calculate_attendance_range` to apply the forecast rule, then write a plain-language explanation of the expected headcount, range, confidence, and the biggest source of uncertainty.
- **Subtask boundary:** Do not choose a planning headcount for the organizer, recommend supply quantities (T6 does that), or report a probability for any individual registrant.
- **Retry limits:** Revise once only if another permitted subtask produces new evidence or reveals an internal conflict.

**Forecast rule:** Expected headcount = (Yes × Yes rate) + (Maybe × Maybe rate) + (No response × no-response rate) + any organizer-stated adjustment. "No" responses and opt-outs count as 0. Default rates when history lacks a breakdown: Yes 80%, Maybe 45%, no response 25%. The low end of the range lowers each rate by 10 percentage points; the high end raises each by 10 points. Round the expected headcount and range to whole people. Confidence is "High" when the response rate is 60% or more and history has a breakdown, "Medium" when the response rate is 30–59% or defaults were used, and "Low" otherwise. A forecast requires at least one validated registrant and a readable response log; otherwise report `Insufficient evidence to forecast` with the exact need.

- **Decision guidance:** After each subtask, use its findings to pick the permitted subtask most likely to resolve the most important remaining uncertainty. Do not follow a fixed sequence. If no permitted subtask can make useful progress, stop and hand the case to the organizer.

## 5. When to Stop or Hand Off to a Human

- **Stop successfully when:** The response counts, selected rates, anomalies checked, expected headcount, low–high range, and confidence level are all recorded with their sources, and the forecast record is saved and confirmed by read-back.
- **Hand off early when:** The response rate is under 30% and no reminder round remains; history and defaults conflict sharply with current responses; an anomaly limit is reached; the expected high end exceeds venue capacity; a tool or inference failure prevents a supported result; or the task-wide time or call limit is reached.
- **Hand off to:** The CPVC event lead organizer, who owns the planning headcount decision.

Stop at the first applicable budget limit or handoff condition. While awaiting review, take no further autonomous action.

## 6. Outbound Deliverable

- **Status:** Completed, Escalated to organizer, or Escalated to organizer — operational error. Include the run ID, event ID, and forecast version.
- **Result or recommendation:** The expected headcount, low–high range, and confidence level (for example, "Expected 62 attendees, range 51–73, Medium confidence"), or `Insufficient evidence to forecast` / `undetermined` when a supported number is not possible. This is a forecast, not an approved planning headcount.
- **Evidence summary:** A table of response counts by category, the show-up rate used for each and its source (past event or default), organizer adjustments, anomalies found, and how each piece contributes to the number and range.
- **Subtasks performed:** The permitted subtasks completed, including any repeats.
- **Unresolved issues:** Remaining missing, stale, or conflicting evidence and any tool failures. Write `none` only when the task completed with no unresolved issue.
- **Handoff note:** The reason for stopping and exactly what the organizer needs to decide or provide; write `Not applicable` for a completed task.
- **Next task or recipient:** Send every forecast, completed or escalated, to H2: Organizer Reviews Forecast. Only an organizer-approved planning headcount from H2 moves to T6: Recommend Supply Quantities. An operational error keeps the run operationally incomplete even if a partial number exists.
