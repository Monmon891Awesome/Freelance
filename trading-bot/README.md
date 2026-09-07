# XAU/USD Automated Trading Bot (TradingView Pine Script)

A trend-following strategy for Gold (XAU/USD), written in Pine Script v6, meant
to run inside **TradingView's Strategy Tester** — which executes trades
automatically, bar by bar, against a simulated account. That auto-execution
*is* the automated paper trading: once the script is on a chart, you don't
click anything for it to trade.

## Strategy logic

- **Direction**: EMA(12) / EMA(26) crossover marks the trend flip.
- **Confirmation filter**: RSI(14) must be above 50 for longs / below 50 for
  shorts, so entries aren't taken directly into an exhausted move.
- **Risk management**: stop-loss and take-profit are set from ATR(14)
  (default 1.5x ATR stop, 3x ATR target — a 1:2 risk/reward), fixed at the
  bar the trade opens.
- **Position sizing**: quantity is computed so a stop-out loses a fixed
  `riskPct` of current equity (default 1%), not a hardcoded contract count.
- **Optional session filter**: restrict entries to a time window (e.g. the
  London/NY overlap, which is when gold is most liquid).

Everything above is exposed as an input so you can tune it without touching
code.

## Two different things TradingView calls "paper trading"

1. **Strategy Tester** (what this script targets) — attach the strategy to a
   chart, TradingView simulates every entry/exit against a virtual account
   automatically, and the Performance Summary / List of Trades tabs show the
   results. This works identically on historical bars (backtest) and on a
   live, updating chart (forward test) — no manual action needed either way.
2. **Trading Panel → Paper Trading broker** — a separate simulated brokerage
   account you trade *manually* by clicking Buy/Sell (or confirming an
   order). The `alertcondition()` calls in the script fire alerts you can use
   as a signal to place those trades yourself, but TradingView does not wire
   Pine strategy signals into that panel automatically. If you want fully
   hands-off execution against a real broker later, that's done via
   alert webhooks into a bridge service — out of scope for this script but
   the alerts are already there if you want to add it.

For "test an automated bot with paper trading," option 1 is what you want.

## Setup

1. Open TradingView → **Pine Editor** (bottom panel) → New blank script →
   paste in `xauusd_trend_strategy.pine` → **Add to Chart**.
2. Chart symbol: use a spot/CFD gold feed, e.g. `OANDA:XAUUSD`, `FX:XAUUSD`,
   or `TVC:GOLD`. (Futures traders can use `COMEX:GC1!` instead — note the
   contract's point value differs, so re-check position sizing.)
3. Open the **Strategy Tester** tab (bottom panel) to see backtested
   performance: net profit, win rate, max drawdown, list of trades, equity
   curve.
4. Suggested starting timeframe: **1h** (gold is noisy intraday; 15m works
   too but expect more whipsaws — tighten the RSI filter or widen the ATR
   stop if you go lower).
5. Leave the chart open on a live symbol and the strategy keeps
   forward-testing in real time, same simulated-fill logic as the backtest.

## Tuning notes

- Widen `Fast/Slow EMA Length` for a slower, less noisy trend signal.
- Raise `atrSLMult`/`atrTPMult` together to keep the risk/reward ratio but
  give trades more room (useful on higher timeframes).
- Turn off `useRsiFilter` to see how much the confirmation filter is
  actually helping vs. a plain EMA cross.
- Enable the session filter if you only want to trade the London/NY
  overlap, when gold's spread and liquidity are best.

## Disclaimer

This is a starting point for testing an automated strategy in a simulated
environment, not a proven profitable system. Past backtest performance does
not guarantee future results. Validate thoroughly in Strategy Tester across
multiple time ranges before considering any real capital, and never skip the
risk-management inputs.
