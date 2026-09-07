# TradingView Paper Trading Bot

An automated strategy that runs entirely inside TradingView and trades
against TradingView's built-in **Paper Trading** broker — no external
server, no broker API keys, no webhooks. Good for safely testing a
strategy's logic before ever risking real money.

## How this actually works

TradingView doesn't expose an API for its own Paper Trading account, but
it has a feature built for exactly this use case:

1. You write a Pine Script **strategy** (not just an indicator) that
   calls `strategy.entry()` / `strategy.exit()`.
2. You apply it to a chart and connect the chart's **Trading Panel** to
   the **Paper Trading** broker (TradingView's free simulated broker).
3. You flip on **Auto-Trading** for that strategy. From then on, every
   time the strategy's logic fires a signal, TradingView places the
   matching simulated order on your Paper Trading account automatically.

That's the whole "bot" — the automation lives in TradingView itself.

**Important limitation:** TradingView's auto-trading only fires while the
browser tab with that chart is open and TradingView is running (it's
evaluated client-side in real time). If you close the tab or your
computer sleeps, auto-trading pauses until you reopen it. For this kind
of local test that's usually fine; if you later want it running 24/7
unattended, the standard next step is switching from auto-trading to
**alert → webhook → external server → real broker API** (Alpaca, Binance
Testnet, OANDA, etc.) — happy to build that version too once you've
validated the strategy logic here.

## What's in this folder

- `strategies/ema_rsi_strategy.pine` — an EMA(9/21) crossover strategy,
  filtered by RSI so it skips longs when RSI is overbought and skips
  shorts when RSI is oversold, with ATR-based stop-loss and take-profit.
  All the numbers are exposed as inputs so you can tune them without
  touching code.

## Setup: get it running on Paper Trading

1. **Open TradingView** and go to any chart (pick the symbol/timeframe
   you want to test, e.g. `BTCUSD` 15m or `AAPL` daily).
2. Open the **Pine Editor** tab at the bottom of the screen.
3. Click **Open** → **New blank script**, delete the placeholder code,
   and paste in the contents of `strategies/ema_rsi_strategy.pine`.
4. Click **Add to Chart**. You should see the fast/slow EMA lines plot
   and (if there's trade history in view) triangles marking past
   entries/exits.
5. Open the **Strategy Tester** tab (bottom panel) and check the
   **Performance Summary** / **List of Trades** to sanity-check the
   logic on historical data before trusting it live. Adjust the inputs
   (gear icon on the strategy, or right-click → Settings) — EMA lengths,
   RSI thresholds, ATR multipliers, long/short toggles — until the
   backtest behaves the way you expect.
6. Open the **Trading Panel** (bottom of the chart, next to Strategy
   Tester — may be labeled "Trading").
7. Under the broker list, choose **Paper Trading** and click **Connect**.
   No account/API key is required — it's TradingView's own simulator.
   It starts you with a default cash balance you can reset anytime from
   the panel's settings.
8. Back in the **Strategy Tester** tab, find the **Auto-Trading** toggle
   (near the top of that panel, next to the strategy name) and switch it
   on, with Paper Trading selected as the target account. Confirm the
   order size / settings TradingView shows you.
9. Leave that chart's tab open. From here, every long/short signal the
   strategy generates is automatically sent as a simulated order to your
   Paper Trading account — you can watch fills happen in the Trading
   Panel in real time.

## Tuning the strategy

All key parameters are `input.*` calls at the top of the script, grouped
in the settings dialog:

- **Trend** — fast/slow EMA lengths (default 9/21).
- **Filter** — RSI length and the overbought/oversold thresholds used to
  skip weak-looking entries.
- **Direction** — allow longs, shorts, or both (shorts are off by
  default).
- **Risk** — ATR length and the stop-loss/take-profit multipliers.
- **Backtest window** — restrict the historical backtest to a date range
  without affecting live auto-trading.

Change these from the strategy's Settings dialog (no code edits needed)
and re-check the Strategy Tester results before re-enabling auto-trading.

## Next steps (optional, once you're happy with the logic)

- Add more filters (volume, higher-timeframe trend confirmation, session
  time filters) directly in the Pine script.
- Track performance over a few days/weeks of paper trading and compare
  it against the backtest to catch overfitting.
- If you want the bot to keep trading without a browser tab open, or to
  eventually go live on a real broker, migrate to a webhook-based setup:
  TradingView alert → your own small server → broker API (e.g. Alpaca
  paper/live). Ask and I'll scaffold that as a separate, opt-in upgrade.
