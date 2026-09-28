# About the Agentic System

**A Hackathon Registration and Attendance Planning Agent**

> **Problem to be solved**: The Cal Poly Vibe Coding Club (CPVC) is planning a campus-wide AI Hackathon event for students. Participants register before the event, but not everyone who registers will actually attend. Some participants may change their plans without canceling, while others may remain unsure until shortly before the event. The attendance-to-registration rate of CPVC's last build event was roughly 40%. Because registration totals do not accurately represent actual attendance, CPVC organizers have difficulty deciding how much food, how many drinks, and how much event swag to prepare. Planning for too many attendees wastes the club's limited budget. Planning for too few may leave participants without adequate food or materials and negatively affect their event experience. CPVC currently relies mainly on the number of registrations and the organizers' personal judgment. The club needs a more reliable and manageable way to anticipate actual attendance while respecting participants' privacy and avoiding excessive communication.

### System Designer Name

Tyler Marini

### System Name

HackTrack

### System Goal

**For** CPVC event organizers planning the AI Hackathon, **improve** the accuracy of the headcount they use to buy food, drinks, and swag, **measured by** the absolute percentage error between the organizer-approved planning headcount and the actual number of checked-in attendees, **moving from** a baseline of about 150% error (planning from total registrations when only about 40% of registrants attended the last build event) **to** a target of 20% error or less for this event, **without** sending any registrant more than two attendance-confirmation messages, sharing registrant contact information or responses outside the CPVC organizer team, penalizing or publicly identifying registrants who do not attend, or placing orders and spending club money without an organizer's explicit approval. The planning headcount is the one an organizer approves after the final forecast run; the actual count comes from event check-in records.

### Who Is Better Off When This Works?

CPVC organizers are better off because they can buy supplies from a reviewed, evidence-based attendance forecast instead of the raw registration total, which protects the club's budget while making sure the students who show up have enough food, drinks, and swag.
