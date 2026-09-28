# Retrieve Registration Records Task Specification

## Basic Information

- **Task ID:** T1
- **Task name:** Retrieve Registration Records
- **Task type:** Retrieve
- **Task owner:** HackTrack; the CPVC event lead organizer maintains the event details and registration export connection.
- **Automation level:** L1

## 1. Task Description

T1 starts every planning run. It loads the current registration export, the event details entered by the organizer, and CPVC's prior event attendance history, then packages them for the rest of the workflow. It assigns the run ID and forecast version number (version 1 on the first run, increased by one on an organizer-requested re-forecast from H2). The task follows fixed rules: read the named sources, confirm each required file or field is present and readable, and record when each source was read. It does not clean, judge, or change any registration. If the registration export or required event details are missing or unreadable, the run goes to H1 instead of continuing with partial data.

## 2. Inputs

### Input 1

- **Input name:** Registration export
- **Contents and format:** CSV export from the CPVC registration form with one row per registration: registration timestamp, registrant name, Cal Poly email, registration type (individual or team), team size when stated, and any opt-out flag. Contact fields are read only so T2 can build the contact list.
- **Source:** CPVC registration platform export connected by the CPVC event lead organizer.

### Input 2

- **Input name:** Event details
- **Contents and format:** Structured record with event ID, event date, start time, venue capacity, and supply-order deadline.
- **Source:** CPVC event lead organizer, entered before selecting **Start Attendance Forecast**.

### Input 3

- **Input name:** Prior event attendance history
- **Contents and format:** CPVC event history spreadsheet with one row per past build event: event ID, date, registrations, check-ins, and, when available, check-ins by confirmation response (Yes, Maybe, No, no response).
- **Source:** CPVC event history spreadsheet, updated by T7: Record Actual Attendance after each event.

### Input 4

- **Input name:** Re-forecast request
- **Contents and format:** Optional record with the prior run ID, prior forecast version, and the Organizer forecast note. Absent on the first run.
- **Source:** H2: Organizer Reviews Forecast.

- **If a required input is missing or invalid:** If the registration export or any required event detail is missing or unreadable, record which item failed and send the case to H1: Organizer Reviews Data Issue; the run stops awaiting organizer. A missing or unreadable history file does not stop the run: T1 passes an empty history marked "history unavailable" so T5 can use its documented default rates and flag them as assumptions.

## 3. Outputs

### Output 1

- **Output name:** Raw registration records
- **Contents and format:** The unchanged registration rows with the export file name, row count, and read timestamp.
- **Next task or recipient:** T2: Validate Registration Data.
- **Complete when:** Every row in the export is read, the row count matches the file, and the read timestamp is recorded.

### Output 2

- **Output name:** Event details
- **Contents and format:** Event ID, date, start time, venue capacity, supply-order deadline, run ID, and forecast version number.
- **Next task or recipient:** T2: Validate Registration Data, T3: Send Confirmation Requests, T4: Record Confirmation Responses, T5: Estimate Expected Attendance, and T6: Recommend Supply Quantities.
- **Complete when:** All five organizer-entered fields are present and valid dates or numbers, and a unique run ID and forecast version are assigned.

### Output 3

- **Output name:** Prior event attendance history
- **Contents and format:** The history rows as read, or an empty table marked "history unavailable" with the reason.
- **Next task or recipient:** T5: Estimate Expected Attendance.
- **Complete when:** The history is passed as read or explicitly marked unavailable.

### Output 4

- **Output name:** Data issue report
- **Contents and format:** Run ID, event ID, which required input is missing or unreadable, the error message, and the time of the attempt.
- **Next task or recipient:** H1: Organizer Reviews Data Issue.
- **Complete when:** Produced only when a required input fails; it names each failed item.

## 4. Planned Tools

### Tool 1

- **Tool name:** retrieve_registration_export
- **Input:** Registration export
- **Output:** Raw registration records; Data issue report
- **Implementation Route:** File operations (read-only) on the CSV export in the CPVC organizer shared folder.
- **Integration approach:** Direct integration
- **Role in this task:** Reads every registration row and returns it unchanged with the row count and read timestamp. It cannot edit or delete registrations.
- **Task timeout:** 60 seconds total for one T1 run across all tools.
- **Maximum retries:** 1
- **Retry only when:** A temporary file read error occurs. Wait 5 seconds and retry once. Do not retry a missing file or denied access. The tool is read-only, so a retry cannot create duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record the failure in the Data issue report and send the case to H1: Organizer Reviews Data Issue. Do not pass partial records to T2.

### Tool 2

- **Tool name:** retrieve_event_details
- **Input:** Event details; Re-forecast request
- **Output:** Event details; Data issue report
- **Implementation Route:** Database queries (read-only) against the HackTrack event settings record, plus a function that assigns the run ID and forecast version.
- **Integration approach:** Direct integration
- **Role in this task:** Reads the organizer-entered event fields, checks that each is present and valid, and assigns the run ID and version.
- **Task timeout:** Included in the 60-second T1 limit.
- **Maximum retries:** 1
- **Retry only when:** A temporary connection error occurs. Wait 5 seconds and retry once. Read-only, so a retry creates no duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record the missing or invalid field in the Data issue report and send the case to H1: Organizer Reviews Data Issue.

### Tool 3

- **Tool name:** retrieve_attendance_history
- **Input:** Prior event attendance history
- **Output:** Prior event attendance history
- **Implementation Route:** File operations (read-only) on the CPVC event history spreadsheet. This is the same tool T5 uses.
- **Integration approach:** Direct integration
- **Role in this task:** Reads past event rows and passes them unchanged.
- **Task timeout:** Included in the 60-second T1 limit.
- **Maximum retries:** 1
- **Retry only when:** A temporary file read error occurs. Wait 5 seconds and retry once. Read-only, so a retry creates no duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Pass the history marked "history unavailable" with the error so T5 flags default rates as assumptions and H2 sees that the forecast has no historical basis. The run continues because history is not required to forecast.
