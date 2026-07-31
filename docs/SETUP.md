# 🛠 Setup Guide — GitHub Profile Project

Complete instructions for activating all automated features of this GitHub Profile.

---

## 📋 Prerequisites

- GitHub account: `ilhampryga`
- The special `ilhampryga/ilhampryga` repository (this repo)
- Repository must be **Public**

---

## 🔑 Required GitHub Secrets

Go to: **Settings → Secrets and variables → Actions → New repository secret**

| Secret Name | Description | Where To Get |
|---|---|---|
| `METRICS_TOKEN` | GitHub PAT for metrics | GitHub → Settings → Developer settings → PAT (Classic) |
| `GH_TOKEN` | GitHub PAT for WakaTime stats write | Same as above |
| `WAKATIME_API_KEY` | WakaTime API key | [wakatime.com/settings/api-key](https://wakatime.com/settings/api-key) |
| `SPOTIFY_CLIENT_ID` | Spotify app client ID | [developer.spotify.com/dashboard](https://developer.spotify.com/dashboard) |
| `SPOTIFY_CLIENT_SECRET` | Spotify app client secret | Same dashboard |
| `SPOTIFY_REFRESH_TOKEN` | Spotify OAuth refresh token | See Spotify setup below |

---

## 1️⃣ GitHub Personal Access Token (PAT)

1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. Click **Generate new token (classic)**
3. Name it: `METRICS_TOKEN`
4. Select scopes:
   - ✅ `repo` (full)
   - ✅ `read:user`
   - ✅ `read:org`
   - ✅ `user:email`
5. Set expiration: **No expiration** (or 1 year)
6. Copy the token and add it as the `METRICS_TOKEN` secret
7. Repeat for `GH_TOKEN` with `repo` + `workflow` scopes

---

## 2️⃣ WakaTime Setup

1. Sign up at [wakatime.com](https://wakatime.com)
2. Install the WakaTime plugin for your IDE (VS Code, PyCharm, etc.)
3. Go to [wakatime.com/settings/api-key](https://wakatime.com/settings/api-key)
4. Copy your API key
5. Add it as `WAKATIME_API_KEY` secret in this repo
6. The `waka.yml` workflow will automatically update the `<!--START_SECTION:waka-->` block in `README.md`

---

## 3️⃣ Spotify Setup

### Step 1: Create Spotify App

1. Go to [developer.spotify.com/dashboard](https://developer.spotify.com/dashboard)
2. Click **Create App**
3. Fill in:
   - App name: `GitHub Profile`
   - Redirect URI: `http://localhost:3000/callback`
4. Copy **Client ID** and **Client Secret**
5. Add them as `SPOTIFY_CLIENT_ID` and `SPOTIFY_CLIENT_SECRET` secrets

### Step 2: Get Refresh Token

Run this locally (requires Node.js):

```bash
# Install deps
npm install express axios

# Create auth server
cat << 'EOF' > spotify-auth.js
const express = require('express');
const axios = require('axios');
const app = express();

const CLIENT_ID = 'YOUR_CLIENT_ID';
const CLIENT_SECRET = 'YOUR_CLIENT_SECRET';
const REDIRECT_URI = 'http://localhost:3000/callback';

app.get('/login', (req, res) => {
  const scope = 'user-read-currently-playing user-read-recently-played';
  res.redirect(`https://accounts.spotify.com/authorize?client_id=${CLIENT_ID}&response_type=code&redirect_uri=${encodeURIComponent(REDIRECT_URI)}&scope=${encodeURIComponent(scope)}`);
});

app.get('/callback', async (req, res) => {
  const { code } = req.query;
  const creds = Buffer.from(`${CLIENT_ID}:${CLIENT_SECRET}`).toString('base64');
  const response = await axios.post(
    'https://accounts.spotify.com/api/token',
    `grant_type=authorization_code&code=${code}&redirect_uri=${encodeURIComponent(REDIRECT_URI)}`,
    { headers: { Authorization: `Basic ${creds}`, 'Content-Type': 'application/x-www-form-urlencoded' } }
  );
  console.log('REFRESH TOKEN:', response.data.refresh_token);
  res.send('Copy your refresh token from the terminal!');
  process.exit(0);
});

app.listen(3000, () => console.log('Go to: http://localhost:3000/login'));
EOF

node spotify-auth.js
```

3. Open `http://localhost:3000/login` in browser
4. Authorize the app
5. Copy the `REFRESH TOKEN` from terminal
6. Add it as `SPOTIFY_REFRESH_TOKEN` secret

---

## 4️⃣ Contribution Snake

The snake workflow is fully automated. It will:

1. Run every 12 hours (or on push to `main`)
2. Generate `github-contribution-grid-snake.svg` (light mode)
3. Generate `github-contribution-grid-snake-dark.svg` (dark mode, blue snake)
4. Push both files to the `output` branch

> **First Run:** Manually trigger the workflow from **Actions → 🐍 Contribution Snake → Run workflow**

---

## 5️⃣ Metrics

The metrics workflow generates a comprehensive `assets/metrics.svg` file daily.

**First-time setup:**
1. The workflow automatically falls back to `github.token` (built-in) if `METRICS_TOKEN` is not set.
   - **Without PAT:** Languages, calendar, topics, habits, people, achievements all work.
   - **With PAT** (`METRICS_TOKEN` with `repo`, `read:user`, `read:org`): unlocks traffic and additional data.
2. Manually run: **Actions → 📊 GitHub Metrics → Run workflow**
3. The SVG will be auto-committed to `assets/metrics.svg` via `output_action: commit`

> **Note:** `output_action: commit` means no manual `git push` step is needed — the action commits the SVG directly.

---

## 6️⃣ Enabling All Workflows

After adding all secrets, trigger each workflow once manually:

1. Go to **Actions** tab
2. For each workflow, click **Run workflow**
3. Check the logs for any errors

---

## 📁 File Structure

```
ilhampryga/
├── .github/
│   └── workflows/
│       ├── snake.yml       # Contribution snake (every 12h)
│       ├── metrics.yml     # GitHub metrics (daily)
│       ├── spotify.yml     # Spotify card (every 30m)
│       └── waka.yml        # WakaTime stats (daily)
├── assets/
│   ├── hero-dark.svg       # Hero banner — dark mode
│   ├── hero-light.svg      # Hero banner — light mode
│   ├── terminal-dark.svg   # Animated terminal — dark
│   ├── terminal-light.svg  # Animated terminal — light
│   ├── divider.svg         # Section divider
│   ├── footer-dark.svg     # Footer wave — dark
│   ├── footer-light.svg    # Footer wave — light
│   ├── particles.svg       # Background particles
│   ├── glass-bg.svg        # Glassmorphism BG card
│   ├── logo.svg            # Logo mark
│   ├── metrics.svg         # Auto-generated by metrics.yml
│   └── spotify.svg         # Auto-generated by spotify.yml
├── docs/
│   └── SETUP.md            # This file
└── README.md               # Main profile page
```

---

## 🚀 Quick Verification Checklist

- [ ] Repository is **Public**
- [ ] Repository name matches: `ilhampryga/ilhampryga`
- [ ] `METRICS_TOKEN` secret added
- [ ] `GH_TOKEN` secret added
- [ ] `WAKATIME_API_KEY` secret added
- [ ] `SPOTIFY_CLIENT_ID` secret added
- [ ] `SPOTIFY_CLIENT_SECRET` secret added
- [ ] `SPOTIFY_REFRESH_TOKEN` secret added
- [ ] Snake workflow triggered manually once
- [ ] Metrics workflow triggered manually once
- [ ] Spotify workflow triggered manually once
- [ ] WakaTime workflow triggered manually once
- [ ] `output` branch exists with snake SVGs
- [ ] `assets/metrics.svg` exists and renders
- [ ] `assets/spotify.svg` exists and renders

---

## 💡 Tips

- **Dark/Light mode** — GitHub automatically serves the correct SVG based on user's theme preference via `<picture>` + `<source media="(prefers-color-scheme: ...)">`
- **Caching** — GitHub CDN caches SVGs. If images don't update, append `?v=2` to URLs temporarily
- **Rate limits** — The metrics workflow uses `METRICS_TOKEN` to avoid API rate limits
- **Branch protection** — Do NOT add branch protection to `output` branch, the snake workflow needs to push there

---

*Made with ❤️ in Indonesia*
