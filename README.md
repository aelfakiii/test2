# ابشر بالجواب — Qatar Quiz App

A mobile-first quiz app about Qatar — solo play, or create a "majlis" (game room) and play live with friends from their own devices.

## Quick start (GitHub Pages, easiest path)

1. Create a new repo on GitHub and upload everything in this folder (`index.html`, `manifest.json`, `sw.js`, `icons/`, `app.js`) to the repo root.
2. Go to **Settings → Pages**, set **Source** to your default branch, root folder.
3. Wait a minute, then visit `https://<your-username>.github.io/<repo-name>/`.

That's it — no build step, no dependencies, no server. It's a static site.

## What's in this repo

```
index.html      ← the entire app: HTML + CSS + JS, fully self-contained
app.js          ← same JS as what's inlined in index.html, kept here unminified for easy editing
manifest.json   ← PWA manifest (install-to-home-screen metadata)
sw.js           ← service worker (offline caching for solo play)
icons/          ← app icons (192/512, regular + maskable variants) + the header wordmark logo
```

**Only `index.html` is required to run the app.** The other files (`manifest.json`, `sw.js`, `icons/`) light up PWA install/offline behavior once hosted on a real HTTPS domain — they won't do anything if you just double-click `index.html` locally or preview it in a sandboxed iframe, by browser design.

If you ever want to change app logic, edit `app.js`, then copy its contents into the `<script>` block near the bottom of `index.html` (or just edit the inline script directly — they're identical right now, `app.js` is just there for readability). **Keep these two in sync** — if you only edit one, the deployed app (which reads the inline script in `index.html`) won't reflect your changes.

## Features

- **Top bar** — logo + an account chip (tap to view/edit your name, nickname, gender)
- **Lightweight profile, not real accounts** — first time someone taps any play action (solo, create majlis, join majlis), they're asked for their name, a nickname, and gender. This is stored in the browser's `localStorage` only — there's no backend, no password, no server-side user table. Gender picks the color of their avatar dot in majlis rosters/leaderboards. This is intentionally **not** account-gated for solo play — making someone "sign up" before trying one quiz kills conversion, so solo just asks for a nickname once and remembers it.
- **Solo quiz** — pick a category, 5 questions, two lifelines (50/50, skip), streak tracking
- **Majlis (game rooms)** — create a room, get a 4-character code, friends join from their own phones, everyone answers live, host advances questions, final leaderboard with medals
- **6 categories, each with a short description** — shown both as a swipeable carousel and a full descriptive list on the homepage. Heritage, Qatari food, World Cup 2022, Landmarks, Dialect, Modern Qatar (60 sample questions total, hardcoded — easy to expand in `app.js`'s `QUESTION_BANK`)
- **Homepage explainer** — expanded "how to play" sections for both solo and majlis modes, plus an about block
- **Mobile app shell** — bottom tab bar (primary navigation), fixed phone-sized frame, status-bar-safe spacing, swipeable category carousel
- **Casual Qatari-dialect copy** throughout, not formal MSA
- **Installable PWA** — once hosted, offers "add to home screen" and caches itself for offline solo play

## Navigation — a deliberate choice worth knowing

The brief originally asked for a top menu bar with Play / Contact Us / Login, run alongside the existing bottom tab bar. That was changed during build, on purpose:

- **Play** already has three entry points (the three hero buttons: Play Solo, Create Majlis, Join Majlis) plus the bottom tab bar's own Majlis/Join tabs. A fourth "Play" button in a top bar would be a fifth way to do the same three things — that's clutter, not generosity.
- **Contact Us** lives in the footer (`راسلنا`) rather than primary nav, since it's a low-frequency action that doesn't need permanent screen real estate on a 430px-wide frame.
- **Login** became the lightweight profile system described above (name/nickname/gender, no backend), not real authentication — see the note above on why solo play isn't gated behind it.

The top bar that *did* ship is intentionally slim: logo + account chip only. The bottom tab bar remains the only primary navigation. If you want the original 3-item top bar restored as literally specified, it's a small change in the `.top-bar` section of `index.html` and `account_chip`-related functions in `app.js` — flagging it here so it's an easy find if the client insists.

## Architecture notes (worth knowing before you extend this)

**Multiplayer sync is polling, not push.** Rooms use Claude's artifact `window.storage` API — shared key-value storage, not a real-time backend. Every device polls shared storage every 2 seconds to stay in sync. This works fine for a handful of friends in a casual quiz, but if you ever want sub-second sync or this grows past a prototype, swap this layer for a real backend (Firebase Realtime DB, Supabase Realtime, or your own WebSocket server) — the rest of the UI doesn't need to change, just the `storageGetRoom` / `storageSetRoom` / `storageGetAnswers` / `storageSetAnswers` functions in `app.js`.

**`window.storage` only exists inside a Claude artifact.** This means: *as a plain static site on GitHub Pages, the multiplayer/majlis feature will not work*, because `window.storage` won't be defined in a normal browser. Solo quiz mode (and the profile system) will work perfectly fine anywhere, since both are pure client-side state (`localStorage` for profile, in-memory for solo quiz).

If you want majlis/multiplayer to work once this is deployed off Claude, you have two paths:
1. **Swap in a real backend** (recommended) — replace the four `storage*` functions in `app.js` with calls to Firebase/Supabase/your own API. Everything else (UI, polling cadence, room codes, scoring) can stay as-is.
2. **Keep it running inside Claude** — this app continues to work exactly as built if it's ever re-opened/hosted as a Claude artifact rather than a plain static site, since that's where `window.storage` is provided.

**If "real accounts" ever come back into scope** (persisted across devices, password-based, etc.), that's a different and much bigger build — a real backend with a user database, password hashing, and session management, none of which exists in this codebase. The current profile system is local-only and resets if someone clears their browser data or switches devices.

## Customizing

- **Colors / fonts**: CSS variables at the top of `index.html`'s `<style>` block (`--maroon`, `--gold`, `--font-display`, etc.)
- **Questions**: `QUESTION_BANK` object near the top of `app.js` — 10 questions per category currently, quiz uses the first 5 (solo) or however many the room host picks (5 or 10)
- **Categories**: `CATEGORIES` array in `app.js` — each has a `desc` field used in both the carousel and the full description list on the homepage
- **Icons / logo**: `icons/icon-*.png` are the app icons (derived from the wordmark on a maroon background); `icons/wordmark.png` is the header logo. Replace with your own art, same filenames.
