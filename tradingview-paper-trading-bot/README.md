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

- `strategies/ict_mtf_mgc_strategy.pine` — **primary strategy.** A
  multi-timeframe ICT/Smart-Money-Concepts style bot built for MGC
  (Micro Gold futures):
  - **Daily bias** — bullish/bearish filter from the 1D close vs. a
    daily EMA.
  - **4H Fair Value Gap (FVG)** — locates the most recent unmitigated
    3-candle imbalance on the 4H chart and treats it as the zone price
    needs to be trading in to qualify for an entry.
  - **Break of Structure (BOS)** — checks the 15m, 30m, and 1h
    timeframes for a close breaking the last confirmed swing
    high/low; by default any one of the three firing is enough to
    trigger, with a toggle to require all three.
  - **Execution** — a LONG fires when bias is bullish, price is inside
    the bullish 4H FVG zone, and BOS confirms; SHORT is the mirror
    image on the bearish side.
  - **Risk/sizing** — position size is capped at both a max contract
    count (default 3) and a max dollar risk (default $250), and the
    bot always uses whichever cap is more restrictive for that trade's
    stop distance. It skips the trade entirely if even 1 contract
    would exceed the dollar cap. The stop sits just beyond the
    triggering swing point (with a small ATR buffer); the target is a
    configurable reward:risk multiple (default 2R).
  - The `$ value per 1.00 price move per contract` input defaults to
    10, matching MGC's contract size — update it if you point this at
    a different instrument.
  - This script fetches its own Daily/4H/15m/30m/1h data via
    `request.security()`, so it works no matter what timeframe you
    actually have the chart open on.
  - **Session gate** — only looks for a setup inside the London
    (default 02:00–05:00 America/New_York) and/or NY AM (default
    08:30–11:30 America/New_York) kill zones, capped at one trade per
    zone (so up to 2 trades/day if both fire). Either zone can be
    switched off independently.
  - **No live/intrabar data** — `calc_on_every_tick = false` plus a
    `barstate.isconfirmed` gate mean entries only ever evaluate on a
    fully closed bar, never a live, still-forming one.

- `strategies/scalping_5pip_strategy.pine` — a fast scalper: EMA(5/13)
  crossover filtered by RSI(7), with a **fixed 5-pip stop-loss as 1R**
  and a take-profit at a configurable RR multiple of that. Deliberately
  has **no session gate and no trade cap** — it takes every valid
  signal, all session long, as often as it fires. Pip size is an input
  (defaults to 0.0001 for most FX pairs) since it varies by
  symbol/broker — set it to match whatever you apply this to. Meant for
  a low chart timeframe (1m–5m).

- `strategies/ema_rsi_strategy.pine` — a simpler EMA(9/21) crossover
  strategy, filtered by RSI so it skips longs when RSI is overbought
  and skips shorts when RSI is oversold, with ATR-based stop-loss and
  take-profit. Useful as a lighter-weight starting point or for a
  different symbol/timeframe. All numbers are exposed as inputs.

## Setup: get it running on Paper Trading

1. **Open TradingView** and go to the chart you want to trade — for
   the ICT/MGC strategy, use the MGC continuous contract, e.g.
   `COMEX:MGC1!` (any chart timeframe is fine, since the script pulls
   Daily/4H/15m/30m/1h data itself).
2. Open the **Pine Editor** tab at the bottom of the screen.
3. Click **Open** → **New blank script**, delete the placeholder code,
   and paste in the contents of whichever strategy you want to test
   (`ict_mtf_mgc_strategy.pine`, `scalping_5pip_strategy.pine`, or
   `ema_rsi_strategy.pine`).
4. Click **Add to Chart**. You should see the 4H FVG zone lines plot,
   a background highlight when price is inside a zone, and an info
   label (bottom-right of the latest bar) showing the current Daily
   Bias / FVG status / BOS counts.
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

All key parameters are `input.*` calls at the top of each script, grouped
in the settings dialog. Change them from the strategy's Settings dialog
(no code edits needed) and re-check the Strategy Tester results before
re-enabling auto-trading.

### `ict_mtf_mgc_strategy.pine`

- **Bias** — daily timeframe and EMA length used for the trend filter.
- **Fair Value Gap** — which timeframe FVGs are located on (default 4H).
- **Break of Structure** — the three confirmation timeframes (default
  15m/30m/1h), swing pivot lookback, and whether all three must agree.
- **Risk** — reward:risk multiple (default 2.0, i.e. 1:2), ATR stop
  buffer, max contracts, max dollar risk per trade, and the
  per-contract point value (set this to match whatever instrument you
  actually apply the script to).
- **Session** — toggle London and/or NY AM kill zones on/off, their
  windows (both expressed in the same timezone input), and whether to
  cap each at one trade.

## Backtesting 2019–2025

MGC (Micro Gold futures) started trading on COMEX in 2019, so a
2019–2025 range covers essentially the instrument's whole history —
worth knowing going in, since there's no earlier MGC data to test
against even if you wanted it.

I can't run this backtest myself from here — TradingView's Strategy
Tester executes inside your browser session against your account, and
I don't have a free source of 2019–2025 intraday (15m/30m/1h) MGC data
to reproduce it independently. Producing invented performance numbers
would be worse than no numbers, so here's how to get the real ones in
about a minute:

1. Apply `ict_mtf_mgc_strategy.pine` to `COMEX:MGC1!`.
2. Open **Strategy Tester** → the **Properties** tab (or the gear icon
   on the strategy) → **Backtesting range** → set it to
   `2019-01-01` through `2025-12-31` (or later, up to today).
3. Confirm the defaults match what you asked for: **Risk → Reward:Risk
   multiple = 2.0** (1:2), and under **Session** both "Trade London
   killzone" and "Trade NY AM killzone" checked with "Limit to 1 trade
   per killzone" on — that's London 02:00–05:00 and NY AM 08:30–11:30,
   both America/New_York, up to 2 trades/day total.
4. Read the **Performance Summary** and **List of Trades** tabs for
   the real win rate, total trades, max drawdown, and net P&L over
   that range. On a chart timeframe like 5m or 15m you'll get one
   backtest bar per underlying bar, which is what you want for
   accuracy here — a daily chart timeframe would understate how often
   the session/BOS logic actually fires intrabar.

### `scalping_5pip_strategy.pine`

- **Trend** — fast/slow EMA lengths (default 5/13).
- **Filter** — RSI length and the overbought/oversold thresholds.
- **Direction** — allow longs, shorts, or both (both on by default).
- **Risk** — pip size (match your symbol/broker), the fixed stop in pips
  (default 5, i.e. 1R = 5 pips), and the reward:risk multiple for the
  take-profit.

### `ema_rsi_strategy.pine`

- **Trend** — fast/slow EMA lengths (default 9/21).
- **Filter** — RSI length and the overbought/oversold thresholds used to
  skip weak-looking entries.
- **Direction** — allow longs, shorts, or both (shorts are off by
  default).
- **Risk** — ATR length and the stop-loss/take-profit multipliers.
- **Backtest window** — restrict the historical backtest to a date range
  without affecting live auto-trading.

## Next steps (optional, once you're happy with the logic)

- Add more filters (volume, higher-timeframe trend confirmation, session
  time filters) directly in the Pine script.
- Track performance over a few days/weeks of paper trading and compare
  it against the backtest to catch overfitting.
- If you want the bot to keep trading without a browser tab open, or to
  eventually go live on a real broker, migrate to a webhook-based setup:
  TradingView alert → your own small server → broker API (e.g. Alpaca
  paper/live). Ask and I'll scaffold that as a separate, opt-in upgrade.
