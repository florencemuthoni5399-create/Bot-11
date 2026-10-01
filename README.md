# SynthTrade Pro — headless server bot

Runs the Deriv connection, strategy, martingale, and risk logic as a plain
Node.js process — no browser tab, no throttling, no backgrounding.

**No terminal/SSH?** See `DEPLOY_NO_CODE.md` — same code, deployed via
GitHub's website + Render's dashboard, zero command line.

## Plain VPS setup

1. Get a small always-on server (~$4-6/month). A European datacenter keeps
   the network path to Deriv short.
2. Install Node.js:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
   sudo apt-get install -y nodejs
   ```
3. Upload this folder: `scp -r synthtrade-server root@YOUR_SERVER_IP:/root/`
4. Configure:
   ```bash
   cd /root/synthtrade-server
   npm install
   cp .env.example .env
   nano .env   # fill in DERIV_APP_ID, DERIV_API_TOKEN, DASHBOARD_TOKEN
   ```
5. Run it: `node bot.js` — watch for "WebSocket authenticated".
6. Keep it running with pm2:
   ```bash
   sudo npm install -g pm2
   pm2 start bot.js --name synthtrade
   pm2 save
   pm2 startup
   ```
7. Check on it: `http://YOUR_SERVER_IP:8787/?token=YOUR_DASHBOARD_TOKEN`

Every settled trade is also appended to `trades.log.jsonl`, independent of
the in-memory dashboard, surviving restarts.

## What this configuration does differently from earlier versions

- **1-minute contracts** (`DURATION_VALUE=1`, `DURATION_UNIT=m`) instead of
  tick-based duration.
- **1-minute indicator bars** (`CANDLE_MINUTES=1`) instead of 5-minute or
  tick-count bars — a much faster warm-up (candles needed × 1 minute each)
  while still using real OHLC bars for ADX/ATR, not raw tick noise.
- **Martingale capped at 2 levels** (was 5) — worst case per cycle at
  STAKE=0.5 is $0.50 + $1.00 = $1.50 before the cooldown kicks in.
- **600-second (10-minute) cooldown** (was 60s) after a martingale-exhaustion
  loss or hitting `MAX_CONSECUTIVE_LOSSES` — the circuit breaker auto-resumes
  on its own after the timer runs out, it does not require a manual restart.

## Safety features carried over

- **Buy rejections never hang the bot.** If Deriv refuses a buy (market
  closed, bad duration, stake limits), the lock is released and the bot backs
  off for `BUY_ERROR_BACKOFF_SECONDS` before trying again — it used to be
  possible for this to freeze the bot indefinitely; that's fixed.
- **Reconnect recovery.** If the connection drops mid-trade, any contract
  still open gets re-subscribed after reconnecting; any buy that was in
  flight (so its outcome is genuinely unknown) is marked for you to check
  manually on Deriv's own statement, rather than silently lost or assumed.
- **A startup contract check** asks Deriv what durations/stakes it actually
  allows for the configured asset, and warns (without blocking trading) if
  your configured duration or martingale ceiling falls outside that.
