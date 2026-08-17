# Miniball Subs — Game Day Helper

A single-page, phone-friendly helper for running rolling substitutions at NZ miniball games.

**Live page:** enable GitHub Pages (Settings → Pages → main branch, root) and open `https://<user>.github.io/miniball-subs/`

## How it works

- **Roster** — 8 players (first names only), each with a gender chip (tap to change) and a Playing/Absent toggle for game day.
- **Game format** — ages 5–8 (Years 1–4): 20-minute running clock, two 10-minute halves, 1-minute half-time, rolling subs. Court size adjustable (4 or 5).
- **Court time tracking** — start the game clock; every on-court player's minutes accumulate automatically. Pause at half-time.
- **Sub waves** — one wave every `20:00 ÷ players` (2:30 with 8 players): the whole bench comes on for whoever has been on longest. Waves are anchored to fixed clock times so late taps don't drift; the last 90s holds unless someone is well under fair share. Everyone finishes within ~30–45s of fair share.
- **Game log** — every action is logged in the page; **Share game log** sends it via the phone share sheet (or clipboard) so it can be reviewed afterwards. Last game's log survives a reset.
- **Gender balance** — the on-court boys/girls count is always visible.
- State persists in the browser (localStorage), so a page refresh won't lose the game.

## Game-day flow

1. Open the page on your phone.
2. Mark anyone absent in **Edit roster**.
3. Put the starting five on, hit **Start**.
4. Follow the suggested subs (or chat with Claude for live calls).
