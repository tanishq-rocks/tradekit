# TradeKit

TradeKit is an all-in-one Pine Script v6 indicator for TradingView that bundles the handful of overlays most discretionary traders reach for — moving averages, Bollinger Bands, session/reference levels, VWAP, and candle-pattern detection — into a single script with a consistent settings layout, so you don't have to stack five separate indicators (and five separate legends) on your chart. The repo also includes a small family of standalone multi-symbol [screeners](#screeners) for scanning an entire index for the same patterns instead of checking one chart at a time.

<!-- screenshot/GIF goes here -->

## Table of Contents

- [Why these features](#why-these-features)
- [Session timezones, explained](#session-timezones-explained)
  - [Session hours at a glance](#session-hours-at-a-glance)
  - [Session notes](#session-notes)
- [Getting started](#getting-started)
- [Screeners](#screeners)
  - [Symbol lists (`Symbols/`)](#symbol-lists-symbols)
  - [A note on attribution](#a-note-on-attribution)
- [License](#license)
- [Disclaimer](#disclaimer)
- [Suggested GitHub topics](#suggested-github-topics)

## Why these features

Every feature below has one master on/off toggle in the **Features** group at the top of the settings, and its own settings group for the details (colors, lengths, styles). Nothing runs unless you turn it on.

- **MA 1–4** — Four independently configurable moving averages (length, type, and color each), for tracking trend across multiple timeframes at once — e.g. a fast/slow pair for entries plus a 50/200 pair for the broader trend. Each can be switched between SMA, EMA, SMMA (RMA), WMA, and VWMA without touching the code.
- **Inside Bar** — Flags bars that print entirely inside the prior bar's range, a common pre-breakout consolidation pattern.
- **Outside Bar** — Flags bars that fully engulf the prior bar's range, often marking a volatility expansion or reversal.
- **C2C Bar** — A more specific inside bar: it also requires the close to continue in the direction of the prior candle (prior candle green and this close above the prior close, or prior candle red and this close below the prior close), which a plain inside bar doesn't check. Highlights the candle blue after a green prior candle, black after a red one. Off by default; since it's a stricter version of Inside Bar, its color takes priority over Inside Bar's on any candle where both would fire.
- **Bollinger Bands** — Standard-deviation bands around a moving average, for gauging volatility and mean-reversion extremes. Adapted from TradingView's built-in Bollinger Bands indicator.
- **Daily High/Low** — Draws the current session's high and low live as they form, so you always have today's range on screen.
- **Previous Day/Week/Month High/Low** — Classic higher-timeframe reference levels; prior-period extremes frequently act as support/resistance on the current chart. Previous Day is on by default, Week and Month are off by default to avoid cluttering the chart with levels most traders don't check every session.
- **VWAP** — The volume-weighted average price, a fair-value benchmark widely used by intraday and institutional traders, with optional standard-deviation bands.
- **Dashboard** — A compact table showing price relative to each enabled MA, your Bollinger Band position, and distance from the previous day's high/low, so you don't have to eyeball the chart to get the current read.
- **Sessions** — Highlights the Tokyo, London, and New York trading sessions as colored boxes, each showing its open/close lines, tick range, and average price. Each session runs on its own configured timezone (shown directly on its on-chart label, e.g. "Tokyo (Asia/Tokyo)", so it's never ambiguous which clock it's using), and has a free-text "Notes" field (e.g. which symbols you're trading that session) that shows up right on the label when filled in. Off by default to avoid clutter. Adapted from TradingView's built-in Sessions indicator. Intraday timeframes only (it's automatically skipped on Daily/Weekly/Monthly charts).
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

### Session notes

Each of the three session settings groups (Tokyo Session, London Session, New York Session) ends with a **Notes** field — a multi-line text area (press Enter/Return inside it to start a new line, same as any normal text box), not a single-line box. Whatever you type shows up as extra line(s) directly on that session's on-chart label, right after its name and timezone, only while the session is active. Clear it and nothing extra is added — it costs you nothing unless you use it.

Each session's Notes field comes pre-filled with two reference lines to start you off:

1. **Majors** — the currencies most actively traded during that session, since liquidity for a currency tends to peak while its home market is open:
   - Tokyo: JPY, AUD, NZD
   - London: EUR, GBP, CHF
   - New York: USD, CAD
2. **Killzone** — a "killzone" is trading jargon (popularized by the ICT/"Smart Money Concepts" school of technical analysis) for a narrower, couple-hour window inside a session when institutional order flow is typically heaviest and moves are more likely to originate — narrower than the full exchange session TradeKit actually draws the box for. Killzones are conventionally quoted in New York time, but since that has the exact same DST headache [described above](#session-timezones-explained) — New York is UTC-5 in winter and UTC-4 in summer — we converted them to IST for you instead:
   - Tokyo (Asian Killzone): **06:30–10:30 IST** standard, **05:30–09:30 IST** during US DST (roughly March–November)
   - London (London Killzone): **12:30–15:30 IST** standard, **11:30–14:30 IST** during US DST
   - New York (New York Killzone): **17:30–20:30 IST** standard, **16:30–19:30 IST** during US DST

Notice the whole window shifts **one hour earlier in IST** while the US observes DST — that's the US clock moving forward, not IST moving (IST never observes DST). Both values are pre-filled directly in the note text so you always have the right one in front of you without needing to remember which half of the year it is.

These defaults are just a starting point in an editable text field — edit, replace, or clear them freely. TradeKit doesn't use this text for any calculation or drawing; it's purely a visual reference for you, and treating the killzone hours above as gospel isn't the point here — cross-check them against whatever source you actually trade off of if precision matters to your strategy.

For example, the default Tokyo session label reads:

```
Range: 42.5
Avg: 151.23
Tokyo (Asia/Tokyo)
Majors: JPY, AUD, NZD
Killzone (Asian): 06:30-10:30 IST (05:30-09:30 IST during US DST, ~Mar-Nov)
```

Notes are entirely local to your copy of the script (part of the indicator's saved settings on your chart), not published or shared anywhere.

## Getting started

1. Open any chart on [TradingView](https://www.tradingview.com/) and open the **Pine Editor** tab at the bottom of the screen.
2. Click **Open** → **New blank indicator** (or just clear the editor).
3. Copy the full contents of [`Tradekit.pine`](./Tradekit.pine) and paste it into the editor, replacing whatever is there.
4. Click **Add to Chart**.
5. Click the gear icon on the indicator's legend entry to open its settings, and toggle the features you want under the **Features** group.
6. (Optional) To set up alerts, right-click the chart → **Add Alert**, choose **TradeKit** as the condition, and pick one of the specific conditions (e.g. "Inside Bar", "BB Upper Cross", "MA 1 / MA 2 Cross") from the dropdown.

## Screeners

Unlike `Tradekit.pine`, these don't overlay a chart — each renders a results table listing every symbol (from a whole index, if you like) currently matching its pattern, so you can scan a market in one shot instead of flipping through charts one at a time. Each lives in its own file under [`Screeners/`](./Screeners/) and is pasted into its own separate Pine Editor tab.

- **[`C2C_Screener.pine`](./Screeners/C2C_Screener.pine)** — "Close-to-Close (C2C) Inside Bar Screener". Scans for the same pattern as TradeKit's **C2C Bar** feature above (an inside bar whose close continues in the direction of the prior candle) across a whole symbol list at once. Settings include how many candles back to look, a direction filter (Both/Bullish/Bearish), and the symbol list (see below).
- **[`MSS_Screener.pine`](./Screeners/MSS_Screener.pine)** — "ICT Market Structure Shift (MSS) Screener". Scans for ICT (Inner Circle Trader) Market Structure Shift setups, or a retracement into the Optimal Trade Entry (OTE) zone / an order block following one — your choice via the Screening Method input — built on a ZigZag swing-point engine with configurable length, source, and displacement filter.
- **[`Pre_2_Swing_Sweep.pine`](./Screeners/Pre_2_Swing_Sweep.pine)** — "ICT Liquidity Sweep Pattern Scanner". Scans for liquidity sweeps: price pushing through a prior ZigZag swing high/low and reversing back past it. The Screening Method input lets you check the 1st, 2nd (this file's default — hence the name), or 3rd previous swing point.

All three share the same symbol-list mechanism: a **Preset Index List** dropdown with common Nifty indices (50/100/200/500, Bank, Auto, Pharma, etc.) already hardcoded in, paginated in 40-symbol pages since that's `request.security()`'s per-script limit, or a **Paste Symbols** text box if you'd rather scan a custom list (comma-separated, `EXCHANGE:SYMBOL` format, e.g. `NSE:RELIANCE, NSE:TCS`).

**Using a screener:** open the Pine Editor, paste one of the three files above into a *new* tab (don't overwrite `Tradekit.pine`), click **Add to Chart** — since it renders a table rather than an overlay, it doesn't matter which symbol/chart you add it to. Then pick a Preset Index List (and page) or paste your own symbol list in its settings.

### Symbol lists (`Symbols/`)

[`Symbols/`](./Symbols/) holds plain-text, comma-delimited ticker lists for Nifty 50/100/200/500 and various sector indices, in the same `EXCHANGE:SYMBOL` format the screeners' Paste Symbols box expects. **There's no automatic link between these files and the screeners** — Pine Script runs on TradingView's servers and can't read files out of this repo, so nothing here happens automatically. They're a convenience reference: open the one you want, copy its contents, and paste it into a screener's symbol input (or into a TradingView watchlist). Since each screener already has its own hardcoded Preset Index List covering the same indices, you'll mostly only reach for these when you want a custom subset the presets don't cover.

### A note on attribution

`C2C_Screener.pine` is original work. `MSS_Screener.pine` and `Pre_2_Swing_Sweep.pine` are adapted from public scripts by TradingView user **Arun_K_Bhaskar**, published under MPL 2.0 — each file's header (`// © Arun_K_Bhaskar`) preserves that attribution, as MPL 2.0 requires.

## License

TradeKit is released under the [Mozilla Public License 2.0](./LICENSE). MPL 2.0 was chosen in part because the Bollinger Bands and Sessions sections are adapted from TradingView's own built-in indicators of the same names, both published under the same license. The same license covers the [screeners](#screeners): see [the attribution note above](#a-note-on-attribution) for which are original and which are adapted from another author's published script.

## Disclaimer

TradeKit is provided for educational and informational purposes only. It is not financial advice, and nothing in this script or repository constitutes a recommendation to buy, sell, or hold any security or instrument. Trading involves substantial risk of loss. Do your own research and consult a licensed financial advisor before making trading decisions.

## Suggested GitHub topics

`tradingview` `pinescript` `pine-script` `technical-analysis` `algorithmic-trading` `trading-indicator` `trading-tools` `vwap` `bollinger-bands` `moving-average` `stock-screener` `nifty`
