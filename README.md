# Panini 2026 — My Album

Personal sticker tracker for the Panini FIFA World Cup 2026 album (980 stickers).

## Features
- Mark stickers as collected, track duplicates for trading
- Sidebar grouped by confederation, with per-section progress
- Overall progress bar
- Auto-sync to your Dropbox (across phone + laptop)
- Mobile-friendly layout
- Local-first: works offline, syncs when online

## Running it

It's a single HTML file. You need to serve it over `http://` (not `file://`) so Dropbox OAuth works.

### Local
```
cd /Users/jesussanchez/Documents/BetConnections/panini
python3 -m http.server 8000
```
Open <http://localhost:8000>

### Phone access
Either:
- **Easiest**: drop `index.html` onto <https://app.netlify.com/drop> — get a free https URL you can use anywhere.
- Or push to a private GitHub repo and enable GitHub Pages.

Whatever URL you end up using, that's your **redirect URI** for the Dropbox setup below.

## Deploying to Render

This is a static site — Render serves it for free.

1. Push this folder to a GitHub repo:
   ```
   git init
   git add .
   git commit -m "Initial commit"
   gh repo create panini-album --private --source=. --push
   ```
   (or use the GitHub web UI to create the repo and push manually)

2. Go to <https://dashboard.render.com> → **New** → **Static Site** → connect the repo.

3. Render auto-detects `render.yaml` (already in the repo). Defaults are fine:
   - **Build command:** *(empty)*
   - **Publish directory:** `./`

4. Click **Create Static Site**. You'll get a URL like `https://panini-album.onrender.com`.

5. Add that exact URL to your Dropbox app's **OAuth 2 Redirect URIs** (Settings tab at <https://www.dropbox.com/developers/apps>). Without this step, the Connect button will fail with `invalid_redirect_uri`.

Auto-deploys: every push to `main` redeploys.

## Sharing this with friends

If you want to give the URL to friends so they can track their own collections, edit `index.html` and set:

```js
const SHARED_DBX_APP_KEY = "your_app_key_here";
```

(near the top of the `<script>` block). With that set, friends won't see the "create your own Dropbox app" instructions — they just click **Connect Dropbox** and authorize. Each friend's album lives in *their own* Dropbox app folder, so collections stay private.

Make sure the URL where you host the page is added to your Dropbox app's **OAuth 2 Redirect URIs** (Settings tab in the Dropbox app console). Whatever URL friends use must match exactly.

## Dropbox setup (one-time, ~2 min)

1. Go to <https://www.dropbox.com/developers/apps> → **Create app**
2. Choose:
   - **Scoped access**
   - **App folder** (only sees its own folder — safest)
3. Name it (e.g. `panini-album`) → Create
4. **Permissions** tab → tick `files.content.write` and `files.content.read` → **Submit**
5. **Settings** tab:
   - Copy the **App key**
   - Add your URL (e.g. `http://localhost:8000/` or `https://your-site.netlify.app/`) to **OAuth 2 — Redirect URIs**
6. Open the album, click ⚙ in the header, paste the App key, click **Connect Dropbox**
7. Authorize → you're synced

The album file ends up at `Apps/<your app name>/album.json` in your Dropbox.

## How sync works
- Every change is auto-saved locally and queued for upload (1.5s debounce)
- On open, pulls latest from Dropbox if remote is newer
- Last write wins (don't edit the JSON manually while two devices are open)

## Data shape
```json
{
  "version": 1,
  "lastUpdated": 1746633600000,
  "owned": { "23": true, "47": true },
  "duplicates": { "23": 2 }
}
```
`duplicates.N` is the number of EXTRAS beyond the first one (so 2 means you have 3 total of that sticker).

## Notes on the sticker list
- 980 base stickers, numbered 0–979 (no Coca-Cola exclusives or 1:100 ultra-rares).
- **Intro (0–19):** #0 Panini logo, #1–#2 FIFA Emblem (×2), #3 Mascots, #4 Slogan, #5 Ball, #6–#8 Hosts, #9–#19 FIFA Museum (11 past champions).
- **Teams (20–979):** 48 teams alphabetical, 20 stickers each. Per-team layout: position 1 = Crest, positions 2–12 = Players, position 13 = Team Photo, positions 14–20 = Players.
- Player labels are the team's **FIFA country code** (trigram) + position within team (e.g. `ARG 2`, `KSA 3`, `NED 14`, … — skipping 1 and 13 which are Crest and Team Photo). These are the codes Panini prints on the actual stickers (KSA, GER, NED, POR, SUI, RSA, URU, PAR, CRO, ALG, HAI…), *not* ISO 3166 codes.
- If the official album numbers teams in a different order (e.g. by group draw), rearrange the `TEAMS` array in `index.html` — sticker numbers stay tied to numbers, so existing progress is preserved.
