# Record Confirmation Responses Task Specification

## Basic Information

- **Task ID:** T4
- **Task name:** Record Confirmation Responses
- **Task type:** Remember
- **Task owner:** HackTrack; the CPVC event lead organizer reviews response problems at H2.
- **Automation level:** L1

## 1. Task Description

T4 records every registrant reply (Yes, Maybe, No, or opt-out) with a timestamp and the round it answered, keeping only the latest answer per registrant ID as current while preserving earlier answers. It also applies the workflow's fixed reminder rule: 48 hours after the first round, if fewer than 60% of registrants have responded, no reminder has been sent yet, and at least three days remain before the supply-order deadline, T4 sends a Reminder request for non-responders only back to T3. Otherwise it closes the log and passes the Confirmation response log to T5. The rule is fixed; the task does not interpret free-text replies or guess a response.

## 2. Inputs

### Input 1

- **Input name:** Registrant responses
- **Contents and format:** Each click on a response link: registrant ID, round, answer (Yes, Maybe, No, or opt-out), and click time. Replies received as free text are stored unmatched for organizer review, not interpreted.
- **Source:** Registrants, through the response links in T3's confirmation emails.

### Input 2

- **Input name:** Clean registration roster
- **Contents and format:** Registrant IDs and validation status for all validated registrants.
- **Source:** T2: Validate Registration Data.

### Input 3

- **Input name:** Message send log
- **Contents and format:** Per registrant ID: round, send time, and status (accepted, failed, or uncertain).
- **Source:** T3: Send Confirmation Requests.

### Input 4

- **Input name:** Event details
- **Contents and format:** Event ID, run ID, and supply-order deadline.
- **Source:** T1: Retrieve Registration Records.

- **If a required input is missing or invalid:** A response for an unknown registrant ID or an invalid link is logged as "unmatched" and excluded from counts. If the roster or send log cannot be read, T4 does not close the log; it reports the issue to H1: Organizer Reviews Data Issue.

## 3. Outputs

### Output 1

- **Output name:** Confirmation response log
- **Contents and format:** The latest response per registrant ID (Yes, Maybe, No, or no response), response timestamp, round answered (first request or reminder), and opt-outs; plus the overall response rate, failed/uncertain send counts, unmatched replies, and the time the log was closed.
- **Next task or recipient:** T5: Estimate Expected Attendance.
- **Complete when:** Every roster registrant ID has exactly one current status, the counts add up to the roster total, and the log is marked closed with a timestamp.

### Output 2

- **Output name:** Reminder request
- **Contents and format:** Round marker ("reminder"), the registrant IDs of non-responders who have not opted out, and the decision time.
- **Next task or recipient:** T3: Send Confirmation Requests.
- **Complete when:** Produced at most once per run, only when all three reminder conditions are met, and it lists only non-responders.

## 4. Planned Tools

### Tool 1

- **Tool name:** record_confirmation_responses
- **Input:** Registrant responses; Clean registration roster; Message send log; Event details
- **Output:** Confirmation response log
- **Implementation Route:** Database queries that write each response to HackTrack's access-restricted response table.
- **Integration approach:** Direct integration
- **Role in this task:** Stores each reply once and keeps the latest per registrant as current. It does not message registrants or change the roster.
- **Task timeout:** 10 seconds per response write; the collection window itself stays open until the reminder decision or the log close time.
- **Maximum retries:** 2
- **Retry only when:** A temporary write error occurs. Wait 5 seconds and read back the row keyed by registrant ID, round, and click time; if it exists, do not write again. The unique key prevents duplicate responses.
- **On timeout, exhausted retries, or an error that cannot be retried:** Keep the raw click in the response link service's own record, mark the response "not saved" in the log, and flag it for H2: Organizer Reviews Forecast. An unsaved response is never counted as "no response."

### Tool 2

- **Tool name:** decide_reminder_round
- **Input:** Confirmation response log; Message send log; Event details
- **Output:** Reminder request; Confirmation response log (closed)
- **Implementation Route:** Functions/scripts that apply the fixed 48-hour, 60%, one-reminder, and three-days-before-deadline rule.
- **Integration approach:** Direct integration
- **Role in this task:** Either issues one Reminder request to T3 or closes the log and passes it to T5.
- **Task timeout:** 10 seconds.
- **Maximum retries:** 0
- **Retry only when:** Not applicable.
- **On timeout, exhausted retries, or an error that cannot be retried:** Do not send a reminder. Close the log with the error noted in the Confirmation response log and pass it to T5, which will flag it in its Unresolved issues for H2: Organizer Reviews Forecast.
