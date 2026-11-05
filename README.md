# FX Bot: automated MT5 trading (strategy mode or TradingView mode)

Runs on a **Windows** machine or VPS (the MetaTrader5 Python package is Windows-only) next to a running MT5 terminal.

## Safety defaults
- `dry_run: true` logs what it would do and places nothing.
- It refuses to start on a **live** account unless `allow_live` is true.
- Every trade is checked against: max risk per trade (1% of balance), spread, free margin, max open trades, trading hours (UTC), a daily loss limit, a pause after 3 losses in a row, and a cooldown after each loss.
- With 0.5 lots and a 10-pip stop (about $50 on EURUSD), the 1% rule needs a balance of about $5,000. On a smaller account trades are refused until you lower `lots` or `stop_pips`.

## Setup
1. Install MT5 from your broker, log in to a **demo** account, and turn on **Algo Trading** (toolbar button).
2. Install 64-bit Python 3.10+, then in this folder: `pip install -r requirements.txt`
3. Edit `config.json`. Check your broker's symbol names (some add a suffix like `m` or `.pro`; put it in `symbol_suffix`). Gold needs `pip_sizes` set to 0.1.
4. Run `python bot.py` and watch `bot.log` for a few days in dry-run mode.
5. Set `dry_run` to false, still on demo, and leave it for several weeks before you think about a live account.

## Mode A: the bot generates signals
`python bot.py` checks the last closed candle on each pair. It enters on a fast/slow moving-average cross only if the trend average, RSI, slope and volatility filters agree. Stop is `stop_pips`, take profit is `take_profit_usd` (in account currency), and the stop moves to breakeven at `breakeven_at_usd`. To clear a loss-streak pause: `python bot.py --resume`.

## Mode B: TradingView generates signals
Run `python webhook_server.py` instead of the bot (never both at once).
1. Put the server behind HTTPS on port 443. Simplest: install Caddy on the VPS with a domain name pointing at it, and proxy to `localhost:8080`. Open ports 80 and 443 in the firewall.
2. Set a long random `webhook.secret` in `config.json`.
3. In TradingView create an alert, tick **Webhook URL**, and enter `https://your-domain/webhook`. Webhooks need a paid TradingView plan.
4. Alert message:
   `{"secret":"YOUR_SECRET","action":"buy","symbol":"EURUSD"}`
   Actions: `buy`, `sell`, `close`, `close_all`. Lots, stop and take profit always come from `config.json`, so an alert cannot change them.
5. Optional: paste TradingView's published webhook source IPs (see their help centre) into `webhook.allowed_ips`.

## Keep it running
Use Windows Task Scheduler or NSSM to start `python bot.py` (or the webhook server) at boot, and set MT5 to start automatically too.

## Limits
- Untested against a live MT5 terminal in the environment this was written in. Treat the first demo days as testing.
- No strategy is guaranteed to profit. A 10-pip stop with a $50 target is roughly 1:1, so after spread and commission it needs a win rate above 50% to make money.
- The daily-loss baseline is the balance at the bot's first run each UTC day.
