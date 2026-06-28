# World Cup '26 Sweep

A free, self-hosted sweepstake app for the 2026 FIFA World Cup. Run a "last team standing" sweep for your own group: enter the players, run an animated draw, and let results update themselves.

- **48 teams**, 12 groups of 4 (the real 2026 final draw).
- **Animated live draw** — shuffles the draw order, then deals every team out slot-machine style.
- **Last team standing** — whoever owns the eventual champion takes the pot.
- **Group stage + knockout bracket** tabs that fill in as the tournament plays out, each team tagged with the player who drew it.
- **Live results.** Every viewer's browser fetches live scores from ESPN's public scoreboard and derives the standings client-side. No server, no GitHub Action.
- **Shared draw state** via a `data.json` file synced to GitHub (admin uses a personal access token; everyone else just views).
- **Make it yours** — set your group name and what to call the players from the Settings tab. No code needed.

No build step. Vanilla HTML/CSS/JS.

## Files

| File | Purpose |
|------|---------|
| `index.html` | App shell and views |
| `style.css` | Stadium-broadcast theme |
| `app.js` | Draw, scoring, groups, bracket, GitHub draw-sync, and the live-results engine (fetches and parses ESPN) |
| `data.json` | Teams, group letters, players, draw, branding. No live scores (those are fetched at runtime from ESPN) |
| `scripts/update_results.py` | **No longer run or used.** It was openfootball-based; openfootball was dropped on 2026-06-28. Kept only as historical reference |

## How it works

1. **The Draw** — two steps: *Draw the order* shuffles the running order and locks it in, then *Draw the teams* deals the 48 teams out with animation. Every player gets an equal share; any leftover teams go to **House** (the pot). Both steps save automatically.
2. **Groups** — the 12 group tables, computed live from results, top two highlighted, plus fixtures. Each team is tagged with its owner.
3. **Knockout** — the full bracket, Round of 32 → Final (+ third-place), teams filling in as groups finish.
4. **Standings** — players ranked by how many of their teams are still alive.
5. **Teams** - manual override: the admin can mark teams out / crown the champion by hand (useful if the ESPN feed lags).

## Set this up for your group

1. **Create your own repo from this one** (use it as a template, or fork it) and enable **GitHub Pages** (repo Settings → Pages → Branch: `main`).
2. **Point the app at your repo.** Open `app.js` and set `DEFAULT_REPO.owner` to your GitHub username (and `repo` to your repo name if you renamed it). This is what lets your viewers load the draw data automatically.
3. **Add your admin token.** On the deployed site → **Settings** → fill in repo owner / name, then create a **fine-grained personal access token** with **Contents: Read & write** on your repo (or a classic token with the `repo` scope). Paste it in and *Save config*. The token is stored only in your browser (`localStorage`) and is never committed. No Actions or secrets are needed: results are fetched live from ESPN by every viewer's browser.
4. **Brand it.** Settings → **Group Branding**: set your group name, tagline, and the word for players (e.g. "player", "lad", "member"). Saved with the sweep so everyone sees the same wording.
5. **Run the draw** and share the Pages URL. No token = read-only view of the latest committed data.

## Live results

There is **no server, no GitHub Action and no results cache in the repo**. ESPN's public scoreboard is the **single data source**. On load and every 90s, every viewer's browser does **one fetch** of the ESPN scoreboard (no API key, CORS-open) and the live-results engine in `app.js`:

- builds the whole schedule from that one feed: events are numbered 1..104 by ascending `event.id`, each match's stage comes from `season.slug`, the group letter comes from `data.json`, and the teams/score/status come from the competitors,
- turns knockout slot labels like "Round of 32 3 Winner" into the bracket refs the app uses,
- resolves the bracket and derives each team's alive/out status and the champion.

This is **live**: latency is just how quickly ESPN updates after a match, plus the 90s refresh. If ESPN ever lags, the admin can set results by hand in the **Teams** tab. Last-good results are cached in each browser's `localStorage` for transient outages.

**openfootball was dropped on 2026-06-28.** The engine used to take fixture/bracket **structure** from [openfootball/worldcup.json](https://github.com/openfootball/worldcup.json) and overlay live **scores** from ESPN. openfootball's knockout bracket wiring lagged the group results by hours (after the group stage it left Round-of-32 slots as placeholders like "1I" or "3A/B/C/D/F"), which made the qualification logic wrongly mark **qualified** teams as **OUT**. ESPN resolves the knockout teams immediately, so it is now the only source. (ESPN orders knockout matches differently from openfootball, so the bracket numbering is now ESPN's own.)

## Notes

- Team list reflects the 2026 final draw as published. Any single team can be corrected directly in `data.json` (`name`, `group`, `flag`).
- Local backup: **Settings → Export / Import JSON**.
- All kick-off times display in Sydney time (AEST); adjust in `app.js` (`SYD` constant) if you want a different zone.
