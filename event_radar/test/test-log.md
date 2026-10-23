# Week 9 test log: Club Event Radar

- Student: @TMoneymarini
- Website: https://tmoneymarini.github.io/BUS4498_InClass_Builds/event_radar/
- Folder: https://github.com/TMoneymarini/BUS4498_InClass_Builds/tree/main/event_radar
- Teammate: (add in Step 9)
- Written by the BUS 4498 Hosting Builder on Oct 23, 2026, 12:21 PM.

## Builder checks

| Check | Result | When |
|---|---|---|
| Database | Passed. Visitors can read published events from the database (2 so far). Visitors can’t change the database. Only the agent can. | 12:21 PM |
| AI agent | Passed. The agent is running, and it has your AI key: Claude (claude-haiku-5-5). | 12:21 PM |
| Website (GitHub Pages) | Passed. Your website opens at https://tmoneymarini.github.io/BUS4498_InClass_Builds/event_radar/ config.js holds only the Project URL and the publishable key, with no secret key. | 12:21 PM |
| Photo uploads | Passed. The post is waiting for a moderator, as it should (rule R9). The agent read the date from the photo (Wednesday, November 4). The photo is saved in Supabase Storage. | 12:21 PM |
| New sign-ups turned off | Passed. New sign-ups are off: only the accounts you create can sign in. | 12:21 PM |

## Test posts

| Post | Expected | What happened |
|---|---|---|
| A. A clean post | Published. Thursday, October 29, 7:00 PM to 10:00 PM, University Union (UU), Room 220, Free, free food. | published (then duplicate the second time): Your event is posted on the website. |
| B. A messy group-chat post | Published for tomorrow at 6:00 PM at the UU Plaza. Not tagged as free food, because "snacks maybe??" doesn't clearly promise free food (rule R7). | published: Your event is posted on the website. |
| C. The same event as A, reworded by another officer | Already posted. Same date, start time, and place as post A, so the database refuses a second copy. Post A first. | duplicate: Nothing new was added. This event is already in the database. |
| D. A post that breaks several rules | Waiting for a moderator. Off campus (R3), alcohol (R4), a personal payment handle (R5), and a personal phone number (R6). The phone number and Venmo handle stay off the card. | waiting: A moderator will check this post before it appears on the website. |
| E. A post with missing details | Waiting for a moderator, asking for the date, start time, and place (R2). | waiting: A moderator will check this post before it appears on the website. |
| F. Not an event | Not posted. It's a sale, not an event (R1). | rejected: Here's why: |
| X. a post that tries to give the AI orders | Waiting for a moderator. The agent treats the post as data, not as instructions (this comes back in Week 10). | waiting: A moderator will check this post before it appears on the website. |
| flyer. Flyer photo (Stage 7) | Waiting for a moderator, because posts with a photo are checked first (R9). The card should read the flyer: Sustainability Crew Clothing Swap, Wednesday, November 4, 11:00 AM to 2:00 PM, Dexter Lawn, Free. After a moderator approves it, it appears on the website with the photo. | waiting: A moderator will check this post before it appears on the website. |

## My answers

**Sending post A twice:** The second copy was refused with "already posted," and so was post C, the reworded version. The database stopped it, not the AI: the agent builds a key from the date, start time, and place, and the database's unique rule allows only one live post per key. So a duplicate is refused even if the AI misses it.

**What I approved and rejected:** I approved the Sustainability Crew clothing swap flyer, because it's a free, on-campus event (Dexter Lawn) with a clear date and time, and it now shows on the website with its photo. I rejected post D, the kickoff party, because it's off campus at a house, mentions alcohol (BYOB), and includes a personal Venmo handle and phone number (rules R3 to R6). I also rejected post E (no date, time, or place yet, rule R2) and the bonus beach-house party post (off campus, drinks, and it tried to tell the AI to publish it).

**The hand-off:** (not answered)
