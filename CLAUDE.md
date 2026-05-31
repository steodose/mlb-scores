# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page static site showing MLB box scores and standings. The entire app is one file: `index.html` (HTML + CSS + vanilla JS, no build step, no dependencies).

## Running locally

No build or install. Open `index.html` in a browser, or serve the directory:

```
python3 -m http.server 8000
```

There are no tests, linters, or package manager.

## Architecture

All logic lives in the IIFE `<script>` at the bottom of `index.html`. Key pieces:

- **Data source**: MLB StatsAPI (`https://statsapi.mlb.com`), called directly from the browser with `fetch`. No API key. Two endpoints in use:
  - `/api/v1/schedule?sportId=1&date=YYYY-MM-DD&hydrate=linescore,team` — list of games + linescore for the selected date.
  - `/api/v1/game/{gamePk}/boxscore` — lazy-loaded when a user expands a game.
  - `/api/v1/standings?leagueId=103,104&season=YYYY&standingsTypes=regularSeason&hydrate=team` — standings tab.
- **Team logos**: `https://www.mlbstatic.com/team-logos/{teamId}.svg` via `logoUrl(id)`.
- **Tabs**: `setTab("scores" | "standings")` swaps the main view and toggles the date controls in the header. Scores is the default.
- **Scores view**: `load(dateStr)` → `renderGame(g)` per game. Status logic branches on `status.abstractGameState` (`Preview` / `Live` / `Final`) to format the header line (start time, inning/outs, or final). Linescore innings come from `linescore.innings[]`; R/H/E totals from `linescore.teams[home|away]`. Box scores are fetched lazily on click in the delegated `app.addEventListener("click", ...)` handler and cached via `box.dataset.loaded`.
- **Standings view**: Fetched once into `standingsData`, rendered into two sub-views (`division` / `overall`) selected by `standingsView`. Division order is hard-coded in `DIV_ORDER` / `DIVISIONS` (AL East, Central, West, then NL). Overall view recomputes GB from the leader.
- **Theming**: CSS custom properties on `:root` with a `prefers-color-scheme: dark` override. Accent color (`--accent`) is used for live status, winning team R, and division leaders.

## Conventions

- Keep everything in `index.html` — there is intentionally no bundler, framework, or external JS.
- Be defensive about StatsAPI fields (`?.` / `??` / `|| ""`) — many fields are missing for preview games or doubleheaders.
- Use `font-variant-numeric: tabular-nums` on any new numeric tables to keep columns aligned.
