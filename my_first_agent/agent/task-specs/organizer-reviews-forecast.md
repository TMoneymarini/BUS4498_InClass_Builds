# Organizer Reviews Forecast Task Specification

## Basic Information

- **Task ID:** H2
- **Task name:** Organizer Reviews Forecast
- **Task type:** Decide
- **Task owner:** CPVC event lead organizer.
- **Automation level:** L0

## 1. Task Description

H2 is where a person turns HackTrack's forecast into the planning headcount the club will actually buy for. The organizer reviews T5's expected headcount, low–high range, confidence level, evidence summary, and any unresolved issues or escalation, along with the validation summary and send/response problems. Using their own judgment and anything HackTrack does not know (for example, a class exam that night), the organizer chooses one decision: approve a planning headcount (the forecast number or an adjusted number with a reason), request one re-forecast before the supply-order deadline with a note, or record that they cannot approve yet. Only an approved number moves to T6. The planning headcount is what T7 later compares to actual check-ins, so the goal's error metric is measured against the organizer's decision, not the raw forecast.

## 2. Inputs

### Input 1

- **Input name:** Attendance forecast
- **Contents and format:** T5's Outbound Deliverable: status, run ID, event ID, forecast version, expected headcount, low–high range, confidence, evidence summary, subtasks performed, unresolved issues, and handoff note.
- **Source:** T5: Estimate Expected Attendance.

### Input 2

- **Input name:** Validation summary
- **Contents and format:** Counts of raw, kept, removed, and flagged registrations with reasons.
- **Source:** T2: Validate Registration Data.

### Input 3

- **Input name:** Organizer forecast decision
- **Contents and format:** Human response: approve (with planning headcount and reason if it differs from the forecast), request re-forecast (with Organizer forecast note), or cannot approve yet (with reason).
- **Source:** CPVC event lead organizer.

- **If a required input is missing or invalid:** If the forecast was not saved or is marked operational error, the organizer may still set a planning headcount manually, but the record must say it was set without a supported forecast. Without an organizer decision, the run stays awaiting organizer.

## 3. Outputs

### Output 1

- **Output name:** Approved planning headcount
- **Contents and format:** Event ID, run ID, forecast version reviewed, approved headcount (whole number), whether it was adjusted and why, reviewer, and approval time.
- **Next task or recipient:** T6: Recommend Supply Quantities and T7: Record Actual Attendance.
- **Complete when:** A whole-number headcount is saved with the organizer's name and linked to a specific forecast version.

### Output 2

- **Output name:** Organizer forecast note
- **Contents and format:** Re-forecast request with the reason and any stated adjustment (for example, "partner club bringing 15").
- **Next task or recipient:** T1: Retrieve Registration Records starts the re-forecast run, and T5: Estimate Expected Attendance applies the note.
- **Complete when:** Produced only on a re-forecast request, at most once per event, and only while the supply-order deadline has not passed.

## 4. Planned Tools

### Tool 1

- **Tool name:** record_organizer_decision
- **Input:** Attendance forecast; Validation summary; Organizer forecast decision
- **Output:** Approved planning headcount; Organizer forecast note
- **Implementation Route:** Web API calls that display the forecast on the HackTrack organizer page and append the decision to the event planning sheet (Google Sheets API).
- **Integration approach:** MCP integration
- **Role in this task:** Shows the forecast and evidence and saves the organizer's decision. It never approves, adjusts, or chooses a headcount itself.
- **Task timeout:** Human response deadline: 24 hours after the forecast is saved, or 3 days before the supply-order deadline, whichever comes first.
- **Maximum retries:** Not applicable — manual task.
- **Retry only when:** Not applicable.
- **On timeout, exhausted retries, or an error that cannot be retried:** If the deadline passes, mark the forecast "review overdue" and send it to the CPVC club president as backup organizer. No planning headcount is approved by default and T6 does not start. If saving the decision fails or its outcome is uncertain, read the planning sheet for the decision; if it is not there, show the error and ask the organizer to resubmit before anything moves to T6.
