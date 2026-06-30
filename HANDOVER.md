# World Cup '26 Sweep (shareable app) - Handover

**Owner:** Daniel Fainsinger (Strategy & Ops)
**Last updated:** 2026-06-30
**Status:** Active
**Repo:** https://github.com/f1atty/world-cup-26-sweep-app.git

## What it is

A free, self-hosted "last team standing" sweepstake app for the 2026 FIFA World Cup. A group enters its players, runs an animated draw to allocate the 48 teams, and the app tracks which players' teams are still alive as results come in. Whoever owns the eventual champion takes the pot. It is the genericised, shareable version of the original `world-cup-sweep`: anyone can copy it, set their own branding, and share a read-only link with friends. Vanilla HTML/CSS/JS, no build step, hosted free on GitHub Pages.

## Current status

- Live and working. Draw, group tables, knockout bracket, standings and Match Centre all functional.
- Results are fetched live from ESPN client-side. No server, no GitHub Action, no results cache in the repo.
- Recent fixes (late June 2026): openfootball dropped as a data source (ESPN is now single source); knockout bracket keeps finished ties in their correct slot; group-stage qualifiers derived from group tables.
- Drift risk with the original `world-cup-sweep`: the two are separate codebases, so a fix in one (e.g. a Match Centre layout change) may need applying to the other.

## How to run / access

This is a static site. No build or local server is needed to test, though it can be served with any static file server (e.g. `python -m http.server` from the project root, then open the printed URL).

To set up a fresh sweep for a new group (full plain-English steps are in the in-app **How to use** tab and in `Set-Up-Your-World-Cup-Sweep.docx`):

1. **Use this template** to create a new repo under the user's own account (repo must be flagged: Settings → General → Template repository).
2. Enable **GitHub Pages** (Settings → Pages → branch `main`, `/root`). Site lives at `<username>.github.io/<repo>`.
3. On the deployed site → **Settings**, paste a fine-grained PAT with **Contents: Read & write** on that repo (or a classic token with the `repo` scope). Badge reads "Admin - synced". The token is stored only in the browser (`localStorage`) and is never committed.
4. **Settings → Group Branding** to name it, then run the draw on **The Draw** tab and lock it - that commits the draw to `data.json`.

The PAT is only needed to author/save the draw. Results are fetched live from ESPN by every viewer; a copier without a token gets a read-only view.

## How it works

| Concern | How |
|---------|-----|
| Stack | `index.html` (shell + all views) + `style.css` + `app.js`. No framework, no build step. |
| Draw state | Lives in `data.json`, saved to GitHub via a PAT (admin only). Shared so every viewer sees the same draw/branding. |
| Live results | One ESPN scoreboard fetch on load and every 90s. `buildSchedule(espn)` builds the schedule (events numbered 1..104 by ascending `event.id`, stage from `season.slug`, group letter from `data.json`, teams/score/status from competitors). `refreshResults()` resolves the bracket and derives alive/out + champion. Never written back. Last-good results cached in `localStorage` (`wcs_results:<repo>`). |
| Branding | `DATA.meta.groupName`, `playerWord`, `playerWordPlural`, `subtitle` edited on Settings → Group Branding, stored in `data.json`. `applyBranding()` (from `renderAll`) pushes them into header, title, draw screens and hints. Helpers: `brandGroup()`, `pWord()`, `pWordPlural()`. |
| Repo auto-detection | `detectRepo()` reads `<owner>.github.io/<repo>` off the URL so a fresh copy points at its own repo with no code edit. Falls back to `DEFAULT_REPO` for local dev / custom domains. |
| localStorage namespacing | `APP_NS = location.pathname.split('/')[1]`, used in `CFG_KEY` and `VIEW_KEY`, so two sweeps on the same `*.github.io` origin do not clobber each other. |
| Views | Draw, Match Centre, Groups, Knockout, Standings, Teams (manual admin override), Settings, How to use. |

See `README.md` for the user-facing version of this.

## File / directory map

| Path | What it is |
|------|-----------|
| `index.html` | App shell and all views |
| `style.css` | Stadium-broadcast theme + guide styling |
| `app.js` | Draw, scoring, groups, bracket, Match Centre, branding, GitHub draw-sync, and the live-results engine (`refreshResults`/`buildSchedule`/`deriveStatus`/`bracketOrder`) |
| `data.json` | Teams (48), group letters, players + draw, branding. Ships as a clean slate. No live scores (fetched at runtime). Group letters also label ESPN events |
| `scripts/update_results.py` | No longer run or used (openfootball-based, dropped 2026-06-28). Kept only as historical reference |
| `Set-Up-Your-World-Cup-Sweep.docx` | Plain-English setup guide to send to a non-technical user |
| `README.md` | Public-facing overview and setup |

## Key decisions & gotchas

- **Repo is public** (required for free Pages). Anything in `data.json` (player names, draw, results) is publicly viewable. No tokens/credentials are ever committed.
- **Admin status is purely client-side**: holding a valid token in the browser = admin.
- **Kick-off times render in Sydney time** (the `SYD` constant in `app.js`).
- **openfootball dropped 2026-06-28.** Previously took fixture/bracket structure from openfootball and overlaid ESPN scores. openfootball's knockout wiring lagged group results by hours (left R32 slots as placeholders like "1I" or "3A/B/C/D/F"), wrongly marking qualified teams OUT. ESPN resolves knockout teams immediately, so it is now the only source. ESPN orders knockout matches differently, so bracket numbering is now ESPN's own. Removed helpers: `espnScoreIndex`/`overlayEspnScores`/`kickoffUtc`/`ofScores`.
- **Finished ties kept in their slot (fixed 2026-06-29).** ESPN drops a slot's "winner of match N" ref once its feeder finishes; `bracketOrder()` matches each resolved team back to the prior-round match it won so the tie stays in its correct column. Previously a tie vanished from its column the moment it was played.
- **Genericisation kept internal identifiers as-is.** User-facing "boy/boys" became "player/players", but internal names (`DATA.boys`, CSS classes, `addBoy`/`renderBoysSetup`) were left unchanged to avoid breakage. The leftover-teams pot is `const HOUSE = 'House'` (replaced "40th Trip").
- **Hide-the-Settings-tab toggle.** Shared flag `DATA.meta.hideSettingsTab` (in `data.json`) lets the admin hide Settings from the wider group. Logic: `settingsHidden()`, `settingsTabVisible()`, `adminHashOpen()`, toggle button `#toggleSettingsTab`, label/note synced by `updateSettingsTab()`. Admin always regains access via `#settings` (or `#admin`) on the URL, and the tab stays visible on their own token-holding browser.

## Open tasks / next steps

- [ ] Keep this repo and the original `world-cup-sweep` in sync where a shared fix applies to both.
- [ ] No outstanding feature work flagged. Monitor ESPN feed behaviour through the knockout stages.

## Dependencies, integrations & contacts

- **ESPN public scoreboard** - single live data source for results (no API key, CORS-open).
- **GitHub Pages** - hosting.
- **GitHub Contents API** - draw save/sync via a fine-grained PAT (admin only).
- **Related repo:** the original group-specific `world-cup-sweep` (`f1atty/world-cup-sweep`) this was copied from on 2026-06-09; the two will drift.
- **Contact:** Daniel Fainsinger.
