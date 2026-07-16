# TradeKit

TradeKit is an all-in-one Pine Script v6 indicator for TradingView that bundles the handful of overlays most discretionary traders reach for — moving averages, Bollinger Bands, session/reference levels, VWAP, and candle-pattern detection — into a single script with a consistent settings layout, so you don't have to stack five separate indicators (and five separate legends) on your chart.

<!-- screenshot/GIF goes here -->

## Why these features

Every feature below has one master on/off toggle in the **Features** group at the top of the settings, and its own settings group for the details (colors, lengths, styles). Nothing runs unless you turn it on.

- **MA 1–4** — Four independently configurable moving averages (length, type, and color each), for tracking trend across multiple timeframes at once — e.g. a fast/slow pair for entries plus a 50/200 pair for the broader trend. Each can be switched between SMA, EMA, SMMA (RMA), WMA, and VWMA without touching the code.
- **Inside Bar** — Flags bars that print entirely inside the prior bar's range, a common pre-breakout consolidation pattern.
- **Outside Bar** — Flags bars that fully engulf the prior bar's range, often marking a volatility expansion or reversal.
- **Bollinger Bands** — Standard-deviation bands around a moving average, for gauging volatility and mean-reversion extremes. Adapted from TradingView's built-in Bollinger Bands indicator.
- **Daily High/Low** — Draws the current session's high and low live as they form, so you always have today's range on screen.
- **Previous Day/Week/Month High/Low** — Classic higher-timeframe reference levels; prior-period extremes frequently act as support/resistance on the current chart. Previous Day is on by default, Week and Month are off by default to avoid cluttering the chart with levels most traders don't check every session.
- **VWAP** — The volume-weighted average price, a fair-value benchmark widely used by intraday and institutional traders, with optional standard-deviation bands.
- **Dashboard** — A compact table showing price relative to each enabled MA, your Bollinger Band position, and distance from the previous day's high/low, so you don't have to eyeball the chart to get the current read.
- **Alerts** — Every major event (MA crossovers between consecutive pairs, Bollinger Band crosses, previous-day high/low breaks, inside/outside bars) has a matching alert condition you can wire up from TradingView's alert dialog.

## Getting started

1. Open any chart on [TradingView](https://www.tradingview.com/) and open the **Pine Editor** tab at the bottom of the screen.
2. Click **Open** → **New blank indicator** (or just clear the editor).
3. Copy the full contents of [`Tradekit.pine`](./Tradekit.pine) and paste it into the editor, replacing whatever is there.
4. Click **Add to Chart**.
5. Click the gear icon on the indicator's legend entry to open its settings, and toggle the features you want under the **Features** group.
6. (Optional) To set up alerts, right-click the chart → **Add Alert**, choose **TradeKit** as the condition, and pick one of the specific conditions (e.g. "Inside Bar", "BB Upper Cross", "MA 1 / MA 2 Cross") from the dropdown.

## License

TradeKit is released under the [Mozilla Public License 2.0](./LICENSE). MPL 2.0 was chosen in part because the Bollinger Bands section is adapted from TradingView's own built-in Bollinger Bands indicator, which is published under the same license.

## Disclaimer

TradeKit is provided for educational and informational purposes only. It is not financial advice, and nothing in this script or repository constitutes a recommendation to buy, sell, or hold any security or instrument. Trading involves substantial risk of loss. Do your own research and consult a licensed financial advisor before making trading decisions.

## Suggested GitHub topics

`tradingview` `pine-script` `pine-script-v6` `technical-analysis` `trading-indicator` `trading-tools` `algorithmic-trading`
