# Club Event Radar: handoff notes

For my teammate, so you can keep building this without my laptop or my AI chat.

## What works now

- **Website:** https://tmoneymarini.github.io/BUS4498_InClass_Builds/event_radar/ (GitHub Pages, `main` branch, `/ (root)`)
- Club officers paste an announcement (and optionally a JPG or PNG flyer). The AI agent turns it into an event card, and the posting rules decide whether it's published, waits for a moderator, or isn't posted.
- The database refuses a second live post with the same date, start time, and place (duplicate check).
- Moderators sign in on the website to approve or reject waiting posts. New sign-ups are turned off.

## Where things live

- **Repository:** `TMoneymarini/BUS4498_InClass_Builds`, folder `event_radar/`
- **Supabase project:** `event-radar`, Project URL `https://uzhpkruvzprdxdmxdrmp.supabase.co`
- **Files that matter:**
  - `index.html`, `app.js`, `style.css`, `config.js`: the website (`config.js` holds only the Project URL and the publishable key)
  - `supabase/migrations/`: the database table and security rules, flyer photo storage, and moderator rules
  - `supabase/functions/submit-event/index.ts`: the AI agent (Edge Function, Verify JWT off)
  - `context/posting-rules.md`, `context/campus-places.md`: the rules and place codes the agent uses

## Secrets

- `ANTHROPIC_API_KEY` is set in Supabase → Edge Functions → Secrets. Only the agent can use it.
- You don't need its value, and no key value is written anywhere in this repository. Never commit the secret key (`sb_secret_…`), the database password, or an AI key.

## How to test

Posts are in `test/sample-announcements.md`.

- **Post A** (Board Game Guild game night): published, tagged free food.
- **Post A again, or Post C** (same event reworded): "already posted." The database stops the copy.
- **Post D** (house party, $10, BYOB, Venmo): waits for a moderator with several rule flags. Reject it.
- **Sample flyer** (`test/sample-flyer.png`): waits for a moderator. Approve it, and the clothing swap shows on the website with its photo.

## What's next

- Add a **"Free food only"** button that filters the list to events with `free_food = true`.
- Optional: the nightly clean-up job (builder Step 10) that archives past events.
- A free Supabase project pauses after about a week with no activity. If events stop loading, open Supabase and click **Restore project**.

## How we take turns

1. Tell each other in our group chat before you start working.
2. Download first: GitHub Desktop → **Fetch origin → Pull origin**.
3. Upload when you stop: **Commit to main → Push origin**, then say "pushed."
4. Never push to the same repository at the same time.
