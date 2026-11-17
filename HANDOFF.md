# HANDOFF: The KJ Total Package App

Start here if you are a new AI session picking this up. Update this file in the same commit as every change.

## How Steve works
- Steve owns Stemo Enterprises LLC, which sells this app to KJs ($199/month after a 14-day free trial). He also runs The Dive on 495's own karaoke app, which is a separate repo, server and database.
- Messages are short, often voice-typed from his phone. Reply terse and in plain language: what changed, what it means for a KJ, and the reason.
- His UI rules (every screen you touch):
  - On phones, use wrapped cards, drop-down pickers and wrapped fields. No sideways-scrolling tables.
  - Table headers sit over their data: numbers right-aligned, text left-aligned.
  - Summary numbers and tiles drill down to the detail behind them, with a back button.
- This app stays completely separate from The Dive:
  - Pages carry no Dive on 495 branding or wording. Each KJ sees their own name, logo and colors.
  - There are no bar-owner features: no Bar Owner app, owner list, live view, staff links or website/menu.
- Keep this HANDOFF.md updated in the same commit as every change.
- Never put PINs, keys or secrets in this repo, in commit messages or in this file.

## The app
- Node/Express `server.js` plus static HTML pages. Postgres via `DATABASE_URL`; with no `DATABASE_URL` it uses in-memory storage, which resets on restart. Settings are a key/value store (`db.getSetting/setSetting`), scoped per KJ account.
- Multi-tenant. A KJ signs up at `/start` and gets their own apps under `/b/<their-name>/`:
  - singer app `/` (index.html)
  - KJ page `/kj` (kj.html)
  - `/tv`, `/wheel`, `/ads`, `/history`, `/stats` (analytics), `/wall`
  - `/setup` (logo, colors, owner PIN, plan)
  - `/poster` and `/tent`
- The bare site (the "house" tenant) only serves `/start`; its app pages redirect there. `/bar`, `/owner`, `/staff`, `/watch` and `/links` redirect a KJ to their `/kj`.
- Branding happens at serve time (the KJ's own name/city/address go in as placeholders filled in last, so no rule rewrites them). The HTML is shared with The Dive's app and still says "The Dive" in the source. `brand()` in server.js rewrites paths to `/b/<name>/` and swaps "The Dive" / "The Dive on 495" for the KJ's short or full name before the page is sent. When porting Dive changes:
  - Keep new wording neutral where you can. Text that must name the show can say "The Dive"; `brand()` turns it into the KJ's name.
  - Check the served page (`curl /b/<name>/...`), not the file, for leftover Dive words.
- Identity is the PHONE: each sign-up row has `device` (the `dive_device` cookie, or `c<uid>` for a signed-in singer). Likes, posts, wall and duets hang off the sign-up row.
- `update.js` reloads phones on a new build. `install.js` is the Add-to-Home-Screen guide.
- Plans: a 14-day trial or a code through the My Apps console (`MYAPPS_URL` / `MYAPPS_SECRET`). Stripe card billing is optional (`STRIPE_*`). There are 2 host logins per account.
- Text-message (Twilio) sign-in codes were dropped on 2026-10-26 because they aren't needed. The Twilio code is still in server.js but stays off unless its environment variables are set.

## Ship
- Deploy = push to `main`. Render auto-deploys service srv-dbe3h6vb9u1s73c26vk0.
- The service was suspended on 2026-11-08 and resumed on 2026-11-16. Before relying on a push to go live, check in Render that the service isn't suspended.
- Render setup:
  - Free plan.
  - Free Postgres `kj-total-package-db`, which expires around 2026-11-22 (see Open items).
- Environment variables are set in Render, never in the repo: `DATABASE_URL`, `SESSION_SECRET`, `KJ_PIN`, `DEMO_PIN`, `STRIPE_SECRET_KEY`, `STRIPE_PRICE_ID`, `STRIPE_WEBHOOK_SECRET`, `MYAPPS_URL`, `MYAPPS_SECRET`, `PUBLIC_URL`.
- Optional environment variables:
  - `HOST_TEMP_PIN`: a starting PIN the owner can give new DJ logins.
  - `PLATFORM_KEY`: the key My Apps uses to list and manage accounts. It defaults to `MYAPPS_SECRET`.
  - `MYAPPS_SLUG`: this app's name in My Apps. It defaults to `karaoke`.
- Local test:
  1. Run `npm ci`, then `node --check server.js`.
  2. Start the server: `PORT=3101 DATABASE_URL=postgresql://user@127.0.0.1:<port>/<db> PGSSL=off node server.js`. Leave `DATABASE_URL` unset to use in-memory storage instead.
  3. Make a test KJ with `POST /api/start` (name, slug, email, pin, agree:true).
  4. Agree to the singer terms (`POST /b/<slug>/api/terms`), then sign up singers (`POST /b/<slug>/api/signup`).
  5. Run the KJ page APIs with header `x-kj-pin`.

## Change log (newest first)
- 2026-11-17 Follow-up: "Change my name" needs at least one song in this account (stops name-only sign-up tries from filling the KJ's undo log), and the KJ activity log's quick-rename line names the singer before the change ("Renamed Eve (Song) to Evie").
- 2026-11-17 Ported from The Dive: singers can change their own name ("Change my name" under the locked name): stays locked to their phone, past songs in this account renamed (duet partner kept), 3 a day, can't take a name in line on another phone, the KJ can undo; this account only.
- 2026-11-17 Ported from The Dive: Fix a singer's name (KJ page): rename a phone everywhere or move songs to another singer #; two-tap confirm, logged, one-tap undo (newest first per singer); scoped to the KJ's own account (phones are shown by an account-tied handle, never their cookie; phone-number sign-ins get a per-account name, never changed in other KJ accounts). The KJ page's quick rename is now account-only too, and the KJ activity log records actions again (it missed every `/b/<name>` request).
- 2026-11-17 Branding fix after re-review:
  - A KJ's own name, city and address are no longer rewritten. "Deep Dive Karaoke" used to come out as "Deep Deep Dive Karaoke Karaoke" and "Sky Dive KJ" as "Sky Sky Dive KJ KJ". `brand()` now turns "Dive KJ/Karaoke/..." into "The Dive ..." first and puts the KJ's values in as placeholders filled in last.
  - No doubled app word when the name already ends with it: "Get the Demo KJ app", not "Demo KJ KJ".
  - The share page (`/s/:id`) brands its template before adding the singer's name, song and artist, so singer text like "The Dive" or "McAllen, TX Band" shows as typed.
  - The install bar says "Get the app" when a page has no app name.
  - Pages for normal KJ names are byte-identical to before.
- 2026-11-17 Fixes after review:
  - No Dive or McAllen wording reaches a KJ's pages. The served `manifest.json` and the singer app's home-screen name showed "Dive Karaoke" and "McAllen TX"; `brand()` now covers "Dive Karaoke", every "McAllen TX" form (with no dangling comma or "in" when the KJ has no city) and "Bar + Stage". The install bar's fallback name is now "this app".
  - Spanish for step 4 of the iPhone guide.
  - Step 4 only shows when the guide is opened from "Buzz my phone", not from "Add to my phone" or after a backdrop tap.
  - The uncropped-logo rule matches only the logo file (`/logo.png`).
- 2026-11-17 Ported from The Dive: KJ page "Copy song" button on each waiting singer and "📋 Copy song" on the singer up now (copies "Song Artist" to paste into the karaoke software).
- 2026-11-17 Ported from The Dive: likes have no "Slow down" pop-up; limit raised to 150 a minute per phone, and taps on different singers are no longer dropped.
- 2026-11-17 Ported from The Dive: logo is never cropped in the logo avatars on the singer app and the KJ page (whole logo shows inside the circle).
- 2026-11-17 Ported from The Dive: the install banner's paragraph of steps is now an "📲 Add to my phone" button. Tapping it opens the iPhone steps or the phone's own install box. The Dive's website "Get the app" link (`?getapp`) was not ported because this app has no website.
- 2026-11-17 Ported from The Dive: on an iPhone browser, "Buzz my phone" is now a button that opens the step-by-step add-to-home-screen guide (with a step 4: open the app and tap Buzz my phone), instead of a paragraph of instructions.
- 2026-11-17 Added this HANDOFF.md.
- Through 2026-11-08: The Dive's changes up to Nov 8, 2026 were already ported (song and artist search, duets, time-to-sing banner, lock your spot, phone alerts, tonight's singers, double sign-up guard, add-to-home-screen guide fit, review fixes). See `git log`.
- 2026-10-26 Each KJ's own logo shows faint behind every screen. 9d20fa4 (KJ red by default, demo KJ account from `DEMO_PIN`, sign-up link on the not-found page) was committed marked WIP (see Open items).
- 2026-10-23 KJ app only: no bar/venue sign-up, Bar Owner app, owner list, live view or staff links; "KJ page" wording; analytics by bar (venue log).
- 2026-10-23 The KJ Total Package App created from The Dive's app: sign-up at `/start`, accounts under `/b/<name>/`.

## Decisions
- 2026-10-23 The app is separate from The Dive (own repo, server and database). Reason: it is sold to other KJs under Stemo Enterprises.
- 2026-10-23 KJ businesses only: no bar sign-up and no bar-owner features. Reason: the product is for KJs on the road. The KJ page is the KJ's control room.
- 2026-10-26 Text-message (Twilio) sign-in codes dropped. Reason: they aren't needed.
- 2026-11-17 The Dive's changes are ported here one by one, adapted to multi-tenant and with no Dive branding. Dive-only parts (website `site.html`, bar owner, store manager, food menu) are skipped. Reason: Steve asked for every Dive change made since this app was created to be brought over and deployed as it comes up.
- PINs (owner PIN, `KJ_PIN`, `DEMO_PIN`) live only in Render or hashed in the database, never in code. Reason: README rule.

## Open items
1. Free Postgres `kj-total-package-db` expires around 2026-11-22. Steve has to upgrade it, or move the data, before then or every KJ account is lost.
2. Commit 9d20fa4 is labelled "WIP (not tested, not on main)" but sits on `main` (red default theme, demo KJ account from `DEMO_PIN`, sign-up link on the not-found page). Check that it works on the live service, or fix or revert it.
3. The Twilio code is still in server.js though text codes were dropped. Remove it once Steve confirms.
4. Keep porting new Dive changes as they land (skip website, bar-owner, store-manager and menu features).
5. `/api/singers` (duet partner pick list) still lists phone-number sign-ins from every KJ account (shared `customers` table, not per account). Make it list only singers who have sung in this account.
