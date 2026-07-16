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
- **Sessions** — Highlights the Tokyo, London, and New York trading sessions as colored boxes, each showing its open/close lines, tick range, and average price. Each session runs on its own configured timezone (shown directly on its on-chart label, e.g. "Tokyo (Asia/Tokyo)", so it's never ambiguous which clock it's using). Off by default to avoid clutter. Adapted from TradingView's built-in Sessions indicator. Intraday timeframes only (it's automatically skipped on Daily/Weekly/Monthly charts).
- **Alerts** — Every major event (MA crossovers between consecutive pairs, Bollinger Band crosses, previous-day high/low breaks, inside/outside bars) has a matching alert condition you can wire up from TradingView's alert dialog.

## Session timezones, explained

Each session in the **Sessions** feature (Tokyo, London, New York) has its own "Session timezone" input, and it's worth understanding before you change it, since it's the setting most likely to confuse a new user.

You can enter it two ways:

- **An IANA time zone name**, like `America/New_York`, `Europe/London`, or `Asia/Tokyo` — **this is what TradeKit's defaults use, and what we recommend.**
- **A fixed GMT offset**, like `GMT-5`.

These look interchangeable but aren't, because of **daylight saving time (DST)** — the twice-a-year clock change that some countries observe. New York, for example, is UTC-5 (Eastern Standard Time) in winter but UTC-4 (Eastern Daylight Time) in summer; London is UTC+0 (GMT) in winter but UTC+1 (British Summer Time) in summer. The market's actual local opening bell (e.g. 9:30 AM in New York) doesn't move — only its distance from UTC does, twice a year. Tokyo, by contrast, doesn't observe DST at all and stays at UTC+9 year-round.

- Enter an **IANA name** and TradeKit automatically applies the correct offset for whatever date is on the chart, so the session box always lines up with the real market open/close, in every season, with nothing for you to maintain.
- Enter a **fixed GMT offset** instead, and it stays fixed all year — so for roughly half the year (whenever that market is observing DST), the session box will be drawn a full hour off from the real session.

**Bottom line:** leave the timezone fields on their defaults unless you have a specific reason to change them, and if you do repurpose a session for a different market, use its IANA name rather than a GMT offset. As a sanity check, each session's on-chart label always shows which timezone it's using (e.g. "Tokyo (Asia/Tokyo)"), so you can confirm at a glance that it's reading the clock you expect.

### Session hours at a glance

The exchange's local opening bell never moves — only its UTC/IST offset does, twice a year, for the two markets that observe DST. This is exactly what TradeKit computes for you automatically when you leave the timezone fields on their IANA defaults; the table below just shows what that resolves to, for reference.

| Session | Local hours | Standard time — UTC | Standard time — IST | DST — UTC | DST — IST |
|---|---|---|---|---|---|
| **Tokyo** (`Asia/Tokyo`) | 09:00–15:00 JST | 00:00–06:00 | 05:30–11:30 | *Japan doesn't observe DST — always the standard-time values* | *(same)* |
| **London** (`Europe/London`) | 08:30–16:30 local | 08:30–16:30 (~late Oct–late Mar) | 14:00–22:00 | 07:30–15:30 (~late Mar–late Oct) | 13:00–21:00 |
| **New York** (`America/New_York`) | 09:30–16:00 local | 14:30–21:00 (~early Nov–early Mar) | 20:00–02:30 next day | 13:30–20:00 (~early Mar–early Nov) | 19:00–01:30 next day |

Notes:
- IST (`UTC+5:30`) doesn't observe DST either, so its column shifts along with whichever UTC column applies.
- The New York row crosses midnight in IST — e.g. "20:00–02:30 next day" means the session starts at 8 PM IST and runs past midnight into 2:30 AM IST.
- DST date ranges above are approximate (they shift slightly year to year); TradeKit doesn't need you to track the exact date — the IANA timezone name handles the transition automatically.

## Getting started

1. Open any chart on [TradingView](https://www.tradingview.com/) and open the **Pine Editor** tab at the bottom of the screen.
2. Click **Open** → **New blank indicator** (or just clear the editor).
3. Copy the full contents of [`Tradekit.pine`](./Tradekit.pine) and paste it into the editor, replacing whatever is there.
4. Click **Add to Chart**.
5. Click the gear icon on the indicator's legend entry to open its settings, and toggle the features you want under the **Features** group.
6. (Optional) To set up alerts, right-click the chart → **Add Alert**, choose **TradeKit** as the condition, and pick one of the specific conditions (e.g. "Inside Bar", "BB Upper Cross", "MA 1 / MA 2 Cross") from the dropdown.

## License

TradeKit is released under the [Mozilla Public License 2.0](./LICENSE). MPL 2.0 was chosen in part because the Bollinger Bands and Sessions sections are adapted from TradingView's own built-in indicators of the same names, both published under the same license.

## Disclaimer

TradeKit is provided for educational and informational purposes only. It is not financial advice, and nothing in this script or repository constitutes a recommendation to buy, sell, or hold any security or instrument. Trading involves substantial risk of loss. Do your own research and consult a licensed financial advisor before making trading decisions.

## Suggested GitHub topics

`tradingview` `pinescript` `pine-script` `technical-analysis` `algorithmic-trading` `trading-indicator` `trading-tools` `vwap` `bollinger-bands` `moving-average`
