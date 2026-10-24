# The KJ Total Package App

Karaoke sign-up and show-running app for **KJs on the road** and **bars**, sold by **Stemo Enterprises LLC** ($199/month after a 14-day free trial).

Each KJ or bar signs up at `/start` and gets its own apps under `/b/<their-name>/`:

- Singer sign-up (QR code), wall of fame
- Host / KJ page (run the night, "I'm at a new venue tonight", tips, lock-your-spot)
- TV screen, spin wheel, daily ads, singer history, analytics
- Setup: logo, colors, owner PIN, plan

Separate from The Dive on 495's own karaoke app (its own repo, server and database).

## Run

`npm install` then `npm start`. Set `DATABASE_URL` (Postgres). Optional: `SESSION_SECRET`, `KJ_PIN`, Stripe (`STRIPE_SECRET_KEY`, `STRIPE_PRICE_ID`, `STRIPE_WEBHOOK_SECRET`), Twilio, `MYAPPS_URL` / `MYAPPS_SECRET`, `PUBLIC_URL`.

Never put PINs, keys or secrets in this repo.
