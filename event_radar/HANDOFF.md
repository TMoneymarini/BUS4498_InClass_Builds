# Club Event Radar: handoff notes

For my teammate, so you can keep building this without my laptop or my AI chat.

## What it is

Club officers paste an announcement (and optionally a flyer photo) on the website. An AI agent turns it into an event card, the posting rules decide whether it goes live or waits, and a moderator approves or rejects the waiting posts.

- **Website:** https://tmoneymarini.github.io/BUS4498_InClass_Builds/event_radar/ (GitHub Pages, `main` branch, `/ (root)`)
- **Backend:** Supabase project `event-radar` (Project URL in `config.js`)

## Where everything lives

| Part | Where | File in this folder |
|---|---|---|
| Website | GitHub Pages | `index.html`, `app.js`, `style.css`, `config.js` |
| Database (the agent's memory) | Supabase → Table Editor → `events` | `supabase/migrations/20261023000100_events_table.sql` |
| Flyer photos | Supabase → Storage → `flyers` bucket | `supabase/migrations/20261023000200_flyer_photos.sql` |
| Moderator rules | Supabase → Authentication | `supabase/migrations/20261023000300_moderator_sign_in.sql` |
| AI agent | Supabase → Edge Functions → `submit-event` | `supabase/functions/submit-event/index.ts` |
| Posting rules and campus places | read by the agent | `context/posting-rules.md`, `context/campus-places.md` |
| Test posts | builder test buttons | `test/sample-announcements.md`, `test/sample-flyer.png` |

## Keys: what's where

- `config.js` holds only the Project URL and the **publishable** key. Those are safe to be public.
- The AI key is saved in Supabase → Edge Functions → Secrets as `ANTHROPIC_API_KEY`. You never need it, and it is never in this repo.
- Never commit the secret key (`sb_secret_…`), the database password, or any AI key.

## Settings to keep

- Edge Function `submit-event`: **Verify JWT is off** (the website calls it with the publishable key).
- Authentication: **new sign-ups are off**. Any signed-in user can approve posts, so only the owner adds moderator accounts (Authentication → Users → Add user).
- Duplicate guard: the database allows one live post per date + start time + place, so a repeated event is refused even if the AI misses it.

## How we take turns

1. Say "working on it" in our group chat before you start.
2. In GitHub Desktop: **Fetch origin → Pull origin** first.
3. Make your change, then **Commit to main → Push origin**, and say "pushed."
4. Never push to the same repo at the same time.

If you change the database or the agent, add a new file in `supabase/migrations/` (or update `index.ts`) and run or deploy it in Supabase, so the repo always matches what's live.

## Next up

- Teammate task: add a **"Free food only"** button that filters the list to events with `free_food = true`.
- Optional: nightly clean-up job (builder Step 10) to archive past events.

## Keep it running

A free Supabase project pauses after about a week with no activity. If the website stops loading events, open Supabase and click **Restore project**.
