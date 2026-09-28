# Send Confirmation Requests Task Specification

## Basic Information

- **Task ID:** T3
- **Task name:** Send Confirmation Requests
- **Task type:** Act
- **Task owner:** HackTrack; the CPVC event lead organizer owns the message wording and the sending account.
- **Automation level:** L1

## 1. Task Description

T3 sends each validated registrant one short, organizer-approved email asking whether they still plan to attend the AI Hackathon, with one-click Yes / Maybe / No links and an opt-out link. Registrant answers give T5 much better evidence than the raw registration total. The task follows fixed rules: on the first round, send to every registrant on the contact list who has not opted out; on the reminder round, send only to registrants T4 lists as non-responders. No registrant may ever receive more than two messages, and anyone who answered "No" or opted out receives nothing further. The message uses a fixed template; the task does not write custom text, and it never shares one registrant's information with another.

## 2. Inputs

### Input 1

- **Input name:** Registrant contact list
- **Contents and format:** Registrant ID, first name, Cal Poly email, and opt-out flag for each validated registrant.
- **Source:** T2: Validate Registration Data.

### Input 2

- **Input name:** Event details
- **Contents and format:** Event ID, event date, start time, run ID, and supply-order deadline.
- **Source:** T1: Retrieve Registration Records.

### Input 3

- **Input name:** Reminder request
- **Contents and format:** Round marker ("reminder"), the registrant IDs of non-responders, and the time T4 made the decision. Absent on the first round.
- **Source:** T4: Record Confirmation Responses.

### Input 4

- **Input name:** Message send log
- **Contents and format:** Per registrant ID: each message round already sent, send time, and delivery status. Used to enforce the two-message limit.
- **Source:** HackTrack message log written by this task on earlier rounds.

- **If a required input is missing or invalid:** If the contact list or event details are missing, send nothing and report the gap to H1: Organizer Reviews Data Issue. If the send log cannot be read, send nothing, because the two-message limit cannot be confirmed, and report the case to H1.

## 3. Outputs

### Output 1

- **Output name:** Confirmation requests
- **Contents and format:** One templated email per eligible registrant containing the event name, date, and time, unique response links (Yes, Maybe, No) tied to the registrant ID and round, and an opt-out link.
- **Next task or recipient:** Each eligible registrant; their replies go to T4: Record Confirmation Responses.
- **Complete when:** Every eligible registrant for this round has exactly one send record with an accepted status or a recorded failure.

### Output 2

- **Output name:** Message send log
- **Contents and format:** Per registrant ID: round, send time, message ID, and status (accepted, failed, or uncertain), plus round totals.
- **Next task or recipient:** T4: Record Confirmation Responses; a later T3 round.
- **Complete when:** The log totals equal the eligible count for the round, and every registrant ID has one status.

## 4. Planned Tools

### Tool 1

- **Tool name:** send_confirmation_requests
- **Input:** Registrant contact list; Event details; Reminder request; Message send log
- **Output:** Confirmation requests; Message send log
- **Implementation Route:** Web API calls to the CPVC club email account (Gmail API) using a fixed organizer-approved template.
- **Integration approach:** MCP integration
- **Role in this task:** Selects eligible registrants by rule, sends one templated email to each, and records each send. It cannot send to anyone outside the contact list, send a third message, or change the template.
- **Task timeout:** 10 minutes for one T3 round, including all sends and retries.
- **Maximum retries:** 2
- **Retry only when:** A send returns a temporary error (rate limit or 5xx) with no message ID. Wait 30 seconds, then check the send log and the sent folder for a message to that registrant with the same run ID and round. If one exists, record it as sent and do not resend. Each message carries a unique key (event ID + registrant ID + round), so a retry cannot send a duplicate.
- **On timeout, exhausted retries, or an error that cannot be retried:** Mark that registrant "failed" and do not retry further. If the outcome is uncertain (no confirmation and the sent-folder check fails), mark "uncertain" and never resend to that person in this round. Pass the send log to T4 with the failed and uncertain counts; if more than 10% of sends failed or are uncertain, T4 flags this for H2: Organizer Reviews Forecast. A failed send is never counted as a delivered request.
