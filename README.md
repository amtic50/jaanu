# Jaanu 💗 — Prity's 3-Month Plan Tracker

A phone-first, installable tracker for the 3-month pregnancy-prep plan
(11 Nov 2026 – 20 Feb 2027). No medicine — gutka taper, nicotine gum,
walks, food, sleep and period tracking, with a playful streak/star game
layer. Works fully offline; syncs to Firebase when online.

## What's inside

| File | What it does |
|---|---|
| `index.html` | The whole app — date cards, check-ins, streaks/stars/levels, fertile-window estimate, TSH reminder. Single file, no build step. |
| `manifest.json` | Makes it installable to the home screen (PWA). App name **Jaanu**. |
| `sw.js` | Service worker — caches the app shell so it opens offline. |
| `icon-192.png` / `icon-512.png` | App icon (pink heart + leaf). `icon.svg` is the vector source. |
| `firestore-rules-snippet.txt` | **Complete, ready-to-paste** Firestore rules for the `jaanu_days` collection and the shared `jaanu_meta` doc + the exact console steps. |
| `README.md` | This file. |

## The plan baked into the app

- **Day 1 = 11 Nov 2026**, Day 102 = 20 Feb 2027.
- **Month 1 · Stabilise** (11 Nov – 10 Dec) — taper starts, TSH result collected.
- **Month 2 · Consolidate** (11 Dec – 10 Jan) — taper to zero, one gynaecologist visit (Dr Rajashree Paul, Medimax Polyclinic, Goalpara — Dr Brajen Das is **not** the follow-up doctor).
- **Month 3 · Try** (11 Jan – 12 Feb) — fertile window (cycle days ~10–17, estimated from logged period days) **only if** gutka and alcohol are zero and the checks are clear; otherwise Month 3 becomes a correction month.
- **Jaipur trip week** (13–20 Feb) is marked on the cards.
- **Gutka taper:** 6 → 5 → 4 → 4 → 3 → 3 → 2 → 2 → 1 → **0 from 11 Jan**. Nicotine gum (2 mg) is the craving helper — tap the gum counter instead of a packet.
- **Do not restart the thyroid tablet on her own** — the TSH result (sample given 11 Nov 2026) decides that, with a doctor.

## Publish to GitHub Pages (same as the leave portal)

```bash
cd ~/workspace/jaanu-app
git init
git add .
git commit -m "Jaanu — 3-month plan tracker"
git branch -M main
git remote add origin https://github.com/amtic50/jaanu.git
git push -u origin main
```

Then on github.com → repo **amtic50/jaanu** → **Settings → Pages** →
"Deploy from a branch" → branch `main`, folder `/ (root)` → Save.
The app will be at `https://amtic50.github.io/jaanu/`.

On the phone: open that link in Chrome → menu → **"Add to Home screen"**
(or "Install app"). It then opens full-screen like an app, and works offline.

## Firebase setup (do this once — parent does it in the console)

The app uses the **jaanu-ac156** project (its own Firebase project — not the leave portal's).
The Firebase web config is already in `index.html`.

1. **Anonymous auth** — the app signs in anonymously (no login for Prity):
   `console.firebase.google.com` → project **jaanu-ac156** →
   Build → **Authentication** → **Sign-in method** → **Anonymous** → Enable → Save.
2. **Authorized domain** — Build → **Authentication** → **Settings** →
   **Authorized domains** → make sure `amtic50.github.io` is listed
   (add it if it is missing). Without it,
   anonymous sign-in fails with `auth/unauthorized-domain`.
3. **Firestore rules** — Build → **Firestore Database** → **Rules** tab →
   paste the block from `firestore-rules-snippet.txt` **inside**
   `match /databases/{database}/documents { ... }` and **before** the
   catch-all deny line `match /{document=**} { allow read, write: if false; }`
   → **Publish**.

Until steps 1–3 are done, the app still works 100% on the phone — it
saves everything to the phone first and shows "saved on this phone —
will sync later". Nothing is lost; it syncs automatically once the
rules/auth are live.

## Data model

- Collection: **`jaanu_days`**, one document per day, id `YYYY-MM-DD`.
- Fields: `date`, `gutka` (0–6 or null), `gum` (0–24 or null),
  `alcohol` (true/false/null), `walk`, `food`, `period` (booleans),
  `mood` (happy/ok/meh/low/sad/null), `note` (≤120 chars), `updatedAt`,
  `uid` (anonymous user id).
- Shared doc: **`jaanu_meta/shared`** — cheers/events, rewards, streak
  freeze, period history (`hist`), reports, `why`, `pinHash`.
- Local first: `localStorage` keys `jaanu_days_v1`, `jaanu_settings_v1`,
  `jaanu_pending_v1`. Cloud merge is last-write-wins by `updatedAt`.
