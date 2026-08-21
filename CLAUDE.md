# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A phone-first, single-file web app (`index.html` — inline CSS + vanilla JS, no dependencies, no build step) for running rolling substitutions at NZ miniball games (ages 5–8, 20-min running clock, 2 × 10-min halves). Deployed as a static page via GitHub Pages from `main`.

## Commands

There is no build, lint, or test tooling. To run locally, open `index.html` directly in a browser or serve the folder:

```
python3 -m http.server 8000   # then open http://localhost:8000
```

Verify changes by exercising the page in a browser (ideally a narrow/mobile viewport). Deploy = push to `main`.

## Architecture (all in `index.html`)

Everything hangs off a single global state object `S`, persisted to `localStorage` under `KEY` (`miniball-state-v3`). Every action follows the same pattern: mutate `S` → (usually) `buildWave()` → `logEv()` → `save()` → `render()`. `render()` rebuilds all DOM via `innerHTML` from `S`; there is no incremental DOM state.

Key pieces:

- **State shape**: `S.players[]` (`name, gender M/F/?, present, on, secs, onSince, offSince, adhoc?`), `clock` (game seconds), `running`, `half`, `courtSize`, `lastSwap`, `lastDue` (the scheduled slot the last swap fulfilled), `wave`, `log[]`. `load()` migrates older saved states by back-filling missing fields — when adding a field to `S`, add a default there rather than bumping `KEY` (bumping wipes users' in-progress games).
- **Clock**: `tick()` runs once per second via `setInterval` while `S.running`; increments `S.clock` and `secs` for every on-court player. Timers only accrue while the clock runs (pause at half-time).
- **Wave model** (`buildWave()`): the core logic. Computes `S.wave` = either `{due, pairs:[[courtIdx, benchIdx],...], final?}` or `{hold: "..."}`. Rules:
  - Waves are anchored to fixed clock times: `OPENING` (4:00, 7:00), then every `period()` = `GAME_SECS ÷ presentCount`, offset from the last opening wave. The next due slot is the first anchor after `S.lastDue` (the slot the last swap fulfilled), pushed to `lastSwap + MIN_GAP` if needed — so late taps neither drift the schedule nor skip the following slot.
  - Pairs = longest-on court players ↔ least-played bench players (sorted by `secs`, tie-broken by `onSince`/`offSince`).
  - `MIN_GAP` (60s) floors the next wave after any swap; last `FINAL_WIN` (90s) allows only catch-up swaps where the gap ≥ `FINAL_GAP` (120s) and exceeds time left; last `HOLD_SECS` (45s) is a hard hold.
  - `fairShare()` = `GAME_SECS × min(courtSize, present) ÷ present`; used for the green "reached fair share" highlight.
  - Tapping a suggested pair (`doSwap`) removes just that pair from `S.wave.pairs`; the wave is only rebuilt once all pairs are consumed or a manual sub happens. The wave is a *suggestion* — manual `subOn`/`subOff` always work and trigger a rebuild.
- **Game log**: `logEv(ev, detail)` appends `{t, h, w(wall time), ev, d, sug, snap}` to `S.log` (capped at 600). `logText()` renders it; `shareLog()` uses Web Share API → clipboard → `prompt()` fallback. `resetAll()` stashes the log under `LASTLOG_KEY` so the previous game can still be shared after a reset. `window.onerror` also logs into `S.log`.
- **Roster**: `DEFAULT_PLAYERS` is the hard-coded team (first names only); ad-hoc players added in the UI get `adhoc: true` and a remove button. A fresh game shuffles display order.

## Conventions

- Keep it a single dependency-free file; the whole point is it loads instantly on a phone at courtside.
- User-facing copy in the "How the game runs" section, the `paceline` text, and README.md all describe the wave rules — keep them in sync with `buildWave()` constants when changing pacing.
- Avoid adding `alert`/`confirm` beyond the existing destructive-action confirms; they're awkward on mobile.
