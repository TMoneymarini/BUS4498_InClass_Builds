# Validate Registration Data Task Specification

## Basic Information

- **Task ID:** T2
- **Task name:** Validate Registration Data
- **Task type:** Verify
- **Task owner:** HackTrack; the CPVC event lead organizer resolves data issues it cannot fix by rule.
- **Automation level:** L1

## 1. Task Description

T2 turns the raw registration rows into a clean roster that the rest of the workflow can trust. It applies a fixed checklist: remove exact duplicates (same Cal Poly email), remove test registrations (registrant names or emails containing "test" entered by an organizer account, or rows the organizer flagged as test), flag records missing a Cal Poly email, and carry opt-out flags forward. Each kept registrant gets a stable registrant ID. T2 then splits the result into two outputs: a Clean registration roster without contact details (for T5) and a Registrant contact list (only for T3 and T4), so contact information reaches only the tasks that need it.

The roster is not usable if the export is unreadable, if more than 25% of records lack a Cal Poly email, or if zero valid registrants remain. In those cases T2 sends the case to H1 instead of producing a roster. T2 never edits the original export and never guesses a missing email.

## 2. Inputs

### Input 1

- **Input name:** Raw registration records
- **Contents and format:** Unchanged registration rows with registration timestamp, name, Cal Poly email, registration type, team size, and opt-out flag, plus the export file name, row count, and read timestamp.
- **Source:** T1: Retrieve Registration Records.

### Input 2

- **Input name:** Event details
- **Contents and format:** Event ID, run ID, and forecast version number.
- **Source:** T1: Retrieve Registration Records.

- **If a required input is missing or invalid:** If raw records are missing, their row count does not match, or a validation threshold above fails, send a Data issue report to H1: Organizer Reviews Data Issue and stop the run awaiting organizer.

## 3. Outputs

### Output 1

- **Output name:** Clean registration roster
- **Contents and format:** One record per validated registrant: registrant ID, registration date, registration type (individual or team), team size when stated, and validation status. No names or contact details.
- **Next task or recipient:** T5: Estimate Expected Attendance and T4: Record Confirmation Responses.
- **Complete when:** Every raw row is accounted for as kept, removed as duplicate, removed as test, or flagged, and the kept count plus removed count equals the raw row count.

### Output 2

- **Output name:** Registrant contact list
- **Contents and format:** Registrant ID, first name, Cal Poly email, and opt-out flag for each validated registrant.
- **Next task or recipient:** T3: Send Confirmation Requests. Access is limited to T3 and T4 and the CPVC organizer team.
- **Complete when:** Every roster registrant ID has exactly one contact row, and opted-out registrants are flagged.

### Output 3

- **Output name:** Validation summary
- **Contents and format:** Counts of raw rows, kept, duplicates removed, tests removed, and records flagged, with the reason for each removal or flag.
- **Next task or recipient:** H2: Organizer Reviews Forecast (as supporting evidence) and the event planning sheet.
- **Complete when:** All counts are recorded and reconcile with the raw row count.

### Output 4

- **Output name:** Data issue report
- **Contents and format:** Run ID, event ID, the failed threshold or error, affected row count, and example row numbers (no contact details).
- **Next task or recipient:** H1: Organizer Reviews Data Issue.
- **Complete when:** Produced only when the roster cannot be validated; it names the specific failed check.

## 4. Planned Tools

### Tool 1

- **Tool name:** validate_registration_data
- **Input:** Raw registration records; Event details
- **Output:** Clean registration roster; Registrant contact list; Validation summary; Data issue report
- **Implementation Route:** Functions/scripts that apply the fixed duplicate, test, missing-email, and threshold checks and assign registrant IDs.
- **Integration approach:** Direct integration
- **Role in this task:** Applies the checklist to the supplied rows and returns the outputs above. It does not contact registrants, change the original export, or guess missing data.
- **Task timeout:** 30 seconds for one T2 run.
- **Maximum retries:** 0
- **Retry only when:** Not applicable.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "validation incomplete" with the error in a Data issue report and send the case to H1: Organizer Reviews Data Issue. Do not pass a partial roster or contact list to T3 or T5.

### Tool 2

- **Tool name:** save_clean_roster
- **Input:** Clean registration roster; Registrant contact list; Validation summary
- **Output:** Clean registration roster; Registrant contact list (saved and confirmed)
- **Implementation Route:** Database queries that write the roster and contact list to HackTrack's access-restricted event tables, keyed by event ID and run ID.
- **Integration approach:** Direct integration
- **Role in this task:** Saves the validated outputs so T3, T4, and T5 read the same version.
- **Task timeout:** 20 seconds per save, within the task's overall limit of 60 seconds.
- **Maximum retries:** 1
- **Retry only when:** A temporary write error occurs. Wait 5 seconds, then read back rows for this event ID and run ID. If they already exist and match, treat the save as successful; otherwise write once more. Rows are keyed by event ID, run ID, and registrant ID, so a retry cannot duplicate them.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "roster not saved" and send a Data issue report to H1: Organizer Reviews Data Issue. The run stops as operationally incomplete; T3 must not send messages from an unsaved roster.
