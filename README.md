# NFL Picks Dashboard

Self-hosted GitHub Pages dashboard for the NFL model, mirroring the soccer
Match Day Board's shape: three tabs (Top Edges / Games / Tracker) reading a
JSON blob baked into `index.html` at publish time.

## Files

- `template.html` -- the real template. Has one placeholder,
  `__GAMES_JSON__`, that `nfl-ingest/run/09_push_dashboard.py` replaces with
  live data from `nfl.v_future_games_model` to produce `index.html`.
- `preview_sample.html` -- **sample/synthetic data only**, for looking at the
  UI before your first real pipeline run. Never publish this as `index.html`
  -- it's fake games, not real edges.
- `index.html` -- generated. Do not hand-edit; it's overwritten on every
  push-script run. Not committed until the first real run produces it.

## One-time setup (do this on your machine)

1. Create an empty GitHub repo named `nfl-picks` (same account/org as your
   other dashboards) and push this local repo to it:
   ```
   git remote add origin <your-nfl-picks-repo-url>
   git branch -M main
   git push -u origin main
   ```
2. Enable GitHub Pages for the repo (Settings -> Pages -> Deploy from branch
   `main`, root).
3. Tracker sync backend: create a **new** Google Sheet for NFL picks, open
   Extensions -> Apps Script, and paste in the same `TrackerSync.gs` you're
   already using for Match Day Board (it's generic -- no soccer-specific
   logic). Deploy -> New deployment -> Web app -> Execute as Me -> Who has
   access: Anyone. Copy the resulting `/exec` URL.
4. Open `template.html` and set `TRACKER_SYNC_URL` (search for
   `PASTE_NEW_APPS_SCRIPT_WEB_APP_URL_HERE`) to that URL. Commit that change.
5. Set `NFL_PICKS_REPO_DIR` in your environment if this repo isn't at
   `~/Documents/nfl-picks` (that's the default `09_push_dashboard.py` uses).

## Running it

From `nfl-ingest/`, after your usual data-refresh scripts
(`run/07_odds.py`, `run/08_power_ratings_nfelo.py`) so the view has fresh
lines:

```
python run/09_push_dashboard.py
```

This reads `nfl.v_future_games_model` + team logos, writes `index.html` here,
and commits+pushes it (best-effort on the git step -- if push fails you'll
see a message and can push manually; the JSON build itself never silently
fails).

## Known v1 scope decisions

- No live tradable odds in the model view (only de-vigged probabilities and
  lines), so Tracker odds entry is manual-only -- no default like the soccer
  dashboard's live-price autofill. Revisit once `nfl.v_odds_effective_lines`
  or similar is wired in.
- No composite star/tier score -- "Top Edges" just ranks every market's raw
  |edge| across all games.
- No line-move sparkline -- just a static open -> now display.
