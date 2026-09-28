# Record Actual Attendance Task Specification

## Basic Information

- **Task ID:** T7
- **Task name:** Record Actual Attendance
- **Task type:** Remember
- **Task owner:** HackTrack; the CPVC event lead organizer uploads or confirms the check-in list.
- **Automation level:** L1

## 1. Task Description

After the hackathon, T7 closes the loop on the system goal. It counts the unique checked-in attendees from the event check-in list, compares that count with the organizer-approved planning headcount, and calculates the forecast error with a fixed formula: absolute percentage error = |approved headcount − actual check-ins| ÷ actual check-ins × 100. It also counts check-ins by each registrant's final confirmation response (Yes, Maybe, No, no response) and appends one row to the CPVC event history so T5 can use real show-up rates next time. Walk-ins who did not register are counted in actual attendance and listed separately. T7 stores only counts in the history, never names, and never identifies or penalizes registrants who did not attend.

## 2. Inputs

### Input 1

- **Input name:** Event check-in list
- **Contents and format:** CSV or sheet with one row per check-in: registrant ID or Cal Poly email, check-in time, and a walk-in flag for unregistered attendees.
- **Source:** CPVC event lead organizer, uploaded or confirmed after the event.

### Input 2

- **Input name:** Approved planning headcount
- **Contents and format:** Event ID, run ID, approved headcount, and forecast version.
- **Source:** H2: Organizer Reviews Forecast.

### Input 3

- **Input name:** Confirmation response log
- **Contents and format:** Final response per registrant ID.
- **Source:** T4: Record Confirmation Responses.

- **If a required input is missing or invalid:** If the check-in list is missing or unreadable, or no approved headcount exists, T7 does not calculate an error; the run stops operationally incomplete and the CPVC event lead organizer is asked to upload or confirm the missing item.

## 3. Outputs

### Output 1

- **Output name:** Forecast accuracy record
- **Contents and format:** Event ID, run ID, approved headcount, actual check-ins (registered and walk-ins), absolute percentage error, whether the 20% target was met, and the forecast version it measures.
- **Next task or recipient:** The CPVC event planning sheet and the CPVC event lead organizer.
- **Complete when:** The record is saved and confirmed by read-back; the run is then marked C1: Run Complete.

### Output 2

- **Output name:** Updated attendance history
- **Contents and format:** One new row in the CPVC event history: event ID, date, registrations, check-ins, and check-ins by confirmation response. Counts only, no names or emails.
- **Next task or recipient:** CPVC event history spreadsheet, read by T1: Retrieve Registration Records for the next event.
- **Complete when:** Exactly one row exists for this event ID and is confirmed by read-back.

## 4. Planned Tools

### Tool 1

- **Tool name:** calculate_attendance_error
- **Input:** Event check-in list; Approved planning headcount; Confirmation response log
- **Output:** Forecast accuracy record; Updated attendance history
- **Implementation Route:** Functions/scripts that remove duplicate check-ins, count attendees, match check-ins to response categories by registrant ID, and apply the error formula.
- **Integration approach:** Direct integration
- **Role in this task:** Produces the counts and error value. It does not change the approved headcount or any response.
- **Task timeout:** 2 minutes total for one T7 run across all tools; this tool may take at most 30 seconds.
- **Maximum retries:** 0
- **Retry only when:** Not applicable.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "error not calculated," stop the run as operationally incomplete, and notify the CPVC event lead organizer. Do not record a success or target result.

### Tool 2

- **Tool name:** save_attendance_results
- **Input:** Forecast accuracy record; Updated attendance history
- **Output:** Forecast accuracy record; Updated attendance history (saved and confirmed)
- **Implementation Route:** Web API calls to the CPVC event planning sheet and event history spreadsheet (Google Sheets API).
- **Integration approach:** MCP integration
- **Role in this task:** Writes the accuracy record and the one history row.
- **Task timeout:** 20 seconds per call, within the 2-minute task limit.
- **Maximum retries:** 2
- **Retry only when:** A temporary error occurs. Wait 10 seconds and read back rows for this event ID; if a matching row exists, do not write again. Event ID is the unique key, so a retry cannot duplicate the history row. If the outcome is uncertain and the read-back fails, stop retrying.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "results not saved," send the calculated values to the CPVC event lead organizer so they are not lost, and stop the run as operationally incomplete. The run is not marked complete until both saves are confirmed.
