---
name: "job-search"
description: "Finds current job and internship openings and filters them to the user's background, skills, location, and goals. Use when the user asks to find jobs, internships, or openings, asks \"what roles should I apply to,\" or wants a list of matching positions. Searches job boards and company career pages, drops roles that don't fit (wrong seniority, location, degree requirements, or expired postings), and returns a ranked shortlist with the title, company, location, pay if listed, deadline, a link, and one line on why each role fits. This Skill finds and ranks postings; it does not apply to jobs or contact recruiters."
---

# job-search

## User inputs
- Target role type or field (e.g., cybersecurity, data analytics, IT)
- Job type: internship, part-time, or full-time
- Preferred locations or remote
- Their resume or a short summary of skills, major, and graduation date
- Optional: target companies, minimum pay, start date, or roles to exclude

If the role type, job type, or location is missing, ask for it before continuing. If no resume or skills summary is given, ask for one, or ask whether to continue with a general search.

## Procedure
1. Read the user's inputs and resume. Pull out their key skills, degree, graduation date, experience level, and location limits.
2. Search job boards (LinkedIn, Indeed, Handshake) and company career pages for openings that match the role type, job type, and location.
3. Apply the filters. Remove postings that are expired or older than 30 days, above the user's experience level, in an excluded location, or that require a degree, clearance, or certification the user lacks.
   - Use [job-fit-rules.md](references/job-fit-rules.md): Before removing any posting, apply rules 1–7 (freshness, experience, degree timing, clearance and certifications, location, duplicates, red flags) and record each removal under its filtered-out reason.
4. Check each remaining role against the user's skills. Score it High, Medium, or Low fit based on how many listed requirements the user meets.
   - Use [job-fit-rules.md](references/job-fit-rules.md): Score each remaining role with the Fit scoring table exactly as written, and break ties using the rule below that table.
5. Check that each link works and goes to the original posting. Drop duplicates of the same role.
6. Produce a ranked shortlist of the top 10 roles, with the best fit first.
   - Use [job-shortlist-template.md](assets/job-shortlist-template.md): When producing the final shortlist, use this template's headings and table, replace every {{placeholder}} with the current run's details, and write "Not listed" for missing pay or deadlines.

## Output
Return a ranked shortlist of up to 10 roles, best fit first, as a table with: fit score (High/Medium/Low), title, company, location, pay (or "Not listed"), deadline (or "Rolling"), and a direct link to the original posting.
Under the table, give one line per role on why it fits the user's skills, major, or experience.
Then show a short "Filtered out" count by reason (expired, too senior, wrong location, missing requirement, duplicate) and one concrete next step.
The result should make it clear which roles to apply to first and why the others were dropped.

## Boundaries
Do not apply to jobs, submit applications, create accounts, or contact recruiters on the user's behalf.
Do not invent postings, pay, deadlines, or links. If a detail isn't listed, say "Not listed."
Do not guess the user's work authorization, clearance, or other personal details; ask if a posting depends on them.
Ask before widening the search (new locations, job types, or seniority) when fewer than 5 matches are found.
When a job board can't be searched or a link can't be verified, say which source was skipped and why instead of silently leaving it out.
