# Organizer Reviews Data Issue Task Specification

## Basic Information

- **Task ID:** H1
- **Task name:** Organizer Reviews Data Issue
- **Task type:** Verify
- **Task owner:** CPVC event lead organizer.
- **Automation level:** L0

## 1. Task Description

H1 is the human checkpoint for data problems HackTrack cannot fix by rule, such as a missing or unreadable registration export, missing event details, a roster with too many records lacking a Cal Poly email, or a message log that cannot be read. The organizer reads the Data issue report, looks at the source, and decides how to fix it: re-export the registrations, correct the event details, mark test rows, or confirm that the run should wait. The judgment is the organizer's; HackTrack only displays the report and records the decision. After a fix, the organizer starts a new run with **Start Attendance Forecast**; H1 itself never messages registrants or forecasts attendance.

## 2. Inputs

### Input 1

- **Input name:** Data issue report
- **Contents and format:** Run ID, event ID, the failed input or check, the error, affected row count, and example row numbers (no contact details).
- **Source:** T1: Retrieve Registration Records, T2: Validate Registration Data, T3: Send Confirmation Requests, or T4: Record Confirmation Responses.

### Input 2

- **Input name:** Organizer data decision
- **Contents and format:** Human response: fixed and ready to rerun, cannot fix yet, or cancel the forecast for this event; with a short note on what changed.
- **Source:** CPVC event lead organizer.

- **If a required input is missing or invalid:** If the report is incomplete, the organizer reviews the source files directly. Without an organizer decision, the run stays "awaiting organizer."

## 3. Outputs

### Output 1

- **Output name:** Data issue resolution record
- **Contents and format:** Run ID, event ID, the issue, the organizer's decision and note, reviewer name, and decision time.
- **Next task or recipient:** The event planning sheet; T1: Retrieve Registration Records reads the fixed sources when the organizer starts a new run.
- **Complete when:** The organizer's explicit decision is saved and linked to the run ID.

## 4. Planned Tools

### Tool 1

- **Tool name:** record_organizer_decision
- **Input:** Data issue report; Organizer data decision
- **Output:** Data issue resolution record
- **Implementation Route:** Web API calls that show the report in the HackTrack organizer page and append the decision to the event planning sheet (Google Sheets API). The same tool is used in H2 and H3.
- **Integration approach:** MCP integration
- **Role in this task:** Displays the issue and records the organizer's decision. It does not fix data, restart the run, or make the decision.
- **Task timeout:** Human response deadline: 24 hours after the report is created, or 4 days before the supply-order deadline, whichever comes first.
- **Maximum retries:** Not applicable — manual task.
- **Retry only when:** Not applicable.
- **On timeout, exhausted retries, or an error that cannot be retried:** If the deadline passes, mark the issue "overdue" and send it to the CPVC club president as backup organizer. The run stays awaiting organizer; a missed deadline is not approval and HackTrack does not continue with bad data. If saving the decision fails, show the error to the organizer and keep the run paused until the saved record is confirmed.
