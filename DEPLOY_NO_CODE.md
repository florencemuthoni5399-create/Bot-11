# Deploying with zero command line (GitHub + Render)

~15 minutes, $7/month (Render's smallest always-on tier — the free tier
pauses after 15 minutes with no visitors, which would stop the bot too).

## Part 1 — Put the code on GitHub (just a file upload)

1. github.com → sign up if needed → **+** → **New repository**.
2. On the new repo page: **Add file → Upload files**.
3. Drag in: `bot.js`, `package.json`, `package-lock.json`, `.env.example`,
   `README.md`. **Do not upload a `.env` file** — secrets go into Render's
   dashboard, not GitHub.
4. **Commit changes.**

## Part 2 — Deploy it on Render

1. render.com → **Sign up with GitHub**.
2. **New +** → **Web Service** → pick your repo → **Connect**.
3. Fill in:
   - **Build Command**: `npm install`
   - **Start Command**: `node bot.js`
   - **Instance Type**: **Starter** ($7/month — not Free)
4. **Environment Variables** — add every key from `.env.example`, with your
   real `DERIV_APP_ID`, `DERIV_API_TOKEN`, and a random `DASHBOARD_TOKEN`.
   Leave `PORT` unset — Render assigns it automatically.
5. **Create Web Service.** Watch the Logs tab for "WebSocket authenticated".

## Checking on it

`https://your-app.onrender.com/?token=YOUR_DASHBOARD_TOKEN`

## Changing settings later

Render dashboard → your service → **Environment** tab → edit → **Save
Changes**. Render restarts automatically. Remember: changing anything while
Indicator Confluence is on resets its bar-collection warm-up.
