# Trading Toolkit — Pine Script v6 Upgrade (prompt for Claude Code)

You're helping me evolve a TradingView Pine Script v6 indicator from a personal script into a polished, general-purpose open-source toolkit I'm publishing on GitHub. Work through the phases below **in order**, and commit to git after each phase with a clear, conventional-style commit message (`fix:`, `feat:`, `docs:`, etc.).

**Important constraint:** Pine Script only compiles and runs inside TradingView's own browser-based editor — there is no local compiler, linter, or test runner for it. Don't go looking for one. Instead, review every change carefully against Pine Script v6 syntax, and finish by giving me a manual QA checklist I can run through in the TradingView Pine Editor myself.

**After every phase**, read back through the whole file top to bottom before committing, since nothing will catch a mistake for you: check for variables or functions referenced before they're defined, duplicate names, unmatched parentheses, and inputs pointing at a group name that no longer exists after a rename.

**Also important:** the indicator must stay a single `.pine` file. Pine indicators can't `import` other local files the way normal projects do (that's a separate "library" publishing mechanism I don't want here) — everything below goes into one file that gets pasted whole into TradingView.

**Style:** match the existing code exactly — the `//====...====` banner comments between sections, the aligned `=` signs in related declarations, the current indentation. New sections should look like they were written by the same person as the rest of the file.

**Convention:** every feature has exactly one master on/off toggle, and it lives in the top `Features` group — that's the existing pattern for `enableEMA1`, `enableInsideBar`, `enableBB`, and `enableDHL`. A feature's own settings group (e.g. "VWAP", "Reference Levels") holds only its detail inputs (colors, lengths, multipliers) — never a second enable checkbox. Apply this to every new feature added in Phases 3–5.

**Working process:**
- Work straight through Phase 1 and Phase 2 without stopping — those are corrections and the core generalization, not open design questions.
- After Phase 2's commit, pause, give me a short summary, and wait for my go-ahead before starting Phase 3. Phases 3–5 add new features with real design judgment calls (alert scope, dashboard layout, etc.) I'd rather sanity-check before you build further on top of them.
- Once I approve, Phases 3–6 can run through to completion.
- If you hit a genuine judgment call with no clearly-better option, make the reasonable choice, note the assumption in your phase summary, and keep going — don't stop for those. Do stop and ask about anything where guessing wrong would be costly to undo, including the file-conflict case below.
- One commit per phase, straight on the current branch — no need for feature branches at this size.

## Starting point

Look for an existing `.pine` file at the repo root first (check a `scripts/` or `src/` folder too if the repo already has one, and tell me which you used). If you find one and it matches the baseline below, use it as-is. If you find one that differs meaningfully from the baseline — not just whitespace — stop and describe the differences before proceeding, rather than silently picking a version. If no `.pine` file exists anywhere, create `trading-toolkit.pine` at the repo root with this content as the baseline:

```pinescript
//@version=6
indicator("Trading Toolkit", shorttitle="Trading Toolkit", overlay=true)

//==================================================
// FEATURES
//==================================================
groupFeatures = "Features"

enableEMA1      = input.bool(true,  "Enable EMA 1", group=groupFeatures)
enableEMA2      = input.bool(true,  "Enable EMA 2", group=groupFeatures)
enableEMA3      = input.bool(true,  "Enable EMA 3", group=groupFeatures)
enableEMA4      = input.bool(true,  "Enable EMA 4", group=groupFeatures)
enableInsideBar = input.bool(true,  "Enable Inside Bar", group=groupFeatures)
enableBB = input.bool(true, "Enable Bollinger Bands", group=groupFeatures)
enableDHL = input.bool(true, "Enable Previous Day High/Low", group=groupFeatures)

//==================================================
// SOURCE
//==================================================
src = input.source(close, "Source")

//==================================================
// EMA 1
//==================================================
groupEMA1 = "EMA 1"

len1   = input.int(9, "Length", minval=1, group=groupEMA1)
color1 = input.color(color.blue, "Color", group=groupEMA1)

//==================================================
// EMA 2
//==================================================
groupEMA2 = "EMA 2"

len2   = input.int(13, "Length", minval=1, group=groupEMA2)
color2 = input.color(color.aqua, "Color", group=groupEMA2)

//==================================================
// EMA 3
//==================================================
groupEMA3 = "EMA 3"

len3   = input.int(50, "Length", minval=1, group=groupEMA3)
color3 = input.color(color.green, "Color", group=groupEMA3)

//==================================================
// EMA 4
//==================================================
groupEMA4 = "EMA 4"

len4   = input.int(200, "Length", minval=1, group=groupEMA4)
color4 = input.color(color.red, "Color", group=groupEMA4)

//==================================================
// EMA CALCULATIONS
//==================================================
ema1 = ta.ema(src, len1)
ema2 = ta.ema(src, len2)
ema3 = ta.ema(src, len3)
ema4 = ta.ema(src, len4)

//==================================================
// EMA PLOTS
//==================================================
plot(enableEMA1 ? ema1 : na, title="EMA 1", color=color1, linewidth=1)
plot(enableEMA2 ? ema2 : na, title="EMA 2", color=color2, linewidth=1)
plot(enableEMA3 ? ema3 : na, title="EMA 3", color=color3, linewidth=1)
plot(enableEMA4 ? ema4 : na, title="EMA 4", color=color4, linewidth=1)

//==================================================
// INSIDE BAR SETTINGS
//==================================================
groupIB = "Inside Bar"

insideColor = input.color(color.white, "Inside Bar Color", group=groupIB)
showIBLabel = input.bool(false, "Show IB Label", group=groupIB)

//==================================================
// INSIDE BAR LOGIC
//==================================================
insideBar = enableInsideBar and barstate.isconfirmed and high < high[1] and low > low[1]

// Color candle
barcolor(insideBar ? insideColor : na)

// Label
plotshape(
     showIBLabel and insideBar,
     title="Inside Bar",
     style=shape.triangleup,
     location=location.belowbar,
     color=insideColor,
     size=size.tiny,
     text="IB"
)

//==================================================
// ALERT
//==================================================
alertcondition(enableInsideBar and insideBar,
     title="Inside Bar",
     message="Inside Bar detected")

//==================================================
// BOLLINGER BANDS
//==================================================
groupBB = "Bollinger Bands"

TT_LENGTH  = "The time period to be used in calculating the MA which creates the base for the Upper and Lower Bands."
TT_MA_TYPE = "Determines the type of Moving Average that is applied to the basis plot line."
TT_SOURCE  = "Determines what data from each bar will be used in calculations."
TT_MULT    = "The number of Standard Deviations away from the MA that the Upper and Lower Bands should be."

bbLength = input.int(20, "Length", minval=1, tooltip=TT_LENGTH, group=groupBB)
bbMAType = input.string("EMA", "Basis MA Type",
     options=["SMA", "EMA", "SMMA (RMA)", "WMA", "VWMA"],
     tooltip=TT_MA_TYPE,
     group=groupBB)

bbSrc = input.source(close, "Source", tooltip=TT_SOURCE, group=groupBB)

bbMult = input.float(2.0, "StdDev",
     minval=0.001,
     maxval=50,
     tooltip=TT_MULT,
     group=groupBB)

bbOffset = input.int(0, "Offset",
     minval=-500,
     maxval=500,
     display=display.none,
     group=groupBB)

// MA Function
ma(source, length, _type) =>
    switch _type
        "SMA"         => ta.sma(source, length)
        "EMA"         => ta.ema(source, length)
        "SMMA (RMA)"  => ta.rma(source, length)
        "WMA"         => ta.wma(source, length)
        "VWMA"        => ta.vwma(source, length)

// BB Calculation
basis = ma(bbSrc, bbLength, bbMAType)
dev = bbMult * ta.stdev(bbSrc, bbLength)
upper = basis + dev
lower = basis - dev

// BB Plots
pBasis = plot(enableBB ? basis : na, "BB Basis", color=#2962FF, offset=bbOffset)
pUpper = plot(enableBB ? upper : na, "BB Upper", color=#F23645, offset=bbOffset)
pLower = plot(enableBB ? lower : na, "BB Lower", color=#089981, offset=bbOffset)

fill(pUpper, pLower,
     title="BB Fill",
     color=enableBB ? color.rgb(33,150,243,95) : na)


//@version=6
// indicator("Daily High / Low", overlay = true)

//==================================================
// SETTINGS
//==================================================
groupDHL = "Daily High / Low"

highColor = input.color(color.lime, "High Color", group=groupDHL)
lowColor  = input.color(color.red, "Low Color", group=groupDHL)

lineWidth = input.int(2, "Line Width", minval=1, maxval=5, group=groupDHL)

lineStyleInput = input.string(
     "Dotted",
     "Line Style",
     options = ["Solid", "Dashed", "Dotted"],
     group = groupDHL)

//==================================================
// LINE STYLE
//==================================================
lineStyle =
     lineStyleInput == "Solid"  ? line.style_solid :
     lineStyleInput == "Dashed" ? line.style_dashed :
                                  line.style_dotted

//==================================================
// VARIABLES
//==================================================
newDay = timeframe.change("D")

var float dayHigh = na
var float dayLow = na
var int dayStart = na

var line highLine = na
var line lowLine = na

var line[] highLines = array.new_line()
var line[] lowLines = array.new_line()

//==================================================
// DELETE ALL LINES WHEN DISABLED
//==================================================
if not enableDHL
    if array.size(highLines) > 0
        for i = 0 to array.size(highLines) - 1
            line.delete(array.get(highLines, i))

    if array.size(lowLines) > 0
        for i = 0 to array.size(lowLines) - 1
            line.delete(array.get(lowLines, i))

    array.clear(highLines)
    array.clear(lowLines)

    highLine := na
    lowLine := na

//==================================================
// DRAW DAILY HIGH / LOW
//==================================================
if enableDHL

    if newDay

        // Finish previous day's lines
        if not na(highLine)
            line.set_xy2(highLine, bar_index - 1, dayHigh)

        if not na(lowLine)
            line.set_xy2(lowLine, bar_index - 1, dayLow)

        // Start new day
        dayHigh := high
        dayLow := low
        dayStart := bar_index

        highLine := line.new(
             dayStart,
             dayHigh,
             dayStart,
             dayHigh,
             color = highColor,
             width = lineWidth,
             style = lineStyle)

        lowLine := line.new(
             dayStart,
             dayLow,
             dayStart,
             dayLow,
             color = lowColor,
             width = lineWidth,
             style = lineStyle)

        array.push(highLines, highLine)
        array.push(lowLines, lowLine)

    else

        dayHigh := math.max(dayHigh, high)
        dayLow := math.min(dayLow, low)

        line.set_xy1(highLine, dayStart, dayHigh)
        line.set_xy2(highLine, bar_index, dayHigh)

        line.set_xy1(lowLine, dayStart, dayLow)
        line.set_xy2(lowLine, bar_index, dayLow)

        // Update style dynamically
        line.set_color(highLine, highColor)
        line.set_color(lowLine, lowColor)

        line.set_width(highLine, lineWidth)
        line.set_width(lowLine, lineWidth)

        line.set_style(highLine, lineStyle)
        line.set_style(lowLine, lineStyle)
```

## Phase 1 — Cleanup (small commit, do this first)

- [ ] Delete the stray `//@version=6` and the commented-out `indicator("Daily High / Low", overlay = true)` line in the middle of the file — leftovers from merging two scripts together.
- [ ] Rename the title of the top-level `src = input.source(close, "Source")` input to `"EMA Source"` (keep the variable name `src`). It only feeds the moving averages, not Bollinger Bands, which has its own `bbSrc` — the shared name is misleading.
- [ ] Rename the input label `"Enable Previous Day High/Low"` to `"Enable Daily High/Low"`. The `enableDHL` logic tracks the *current* day live and freezes it at day's end — it never shows yesterday as a reference on today's bars, so the label should match what it does. Don't touch the logic itself here; genuine previous-period levels are added fresh in Phase 3.
- [ ] Delete the `if not enableDHL` block that loops over `highLines`/`lowLines` to delete lines. It can never have anything to delete: Pine recalculates the whole script from bar 0 whenever an input changes, so when `enableDHL` is false, those arrays were never populated in that run to begin with.
- [ ] Add a one-line comment directly above the Bollinger Bands section noting it's adapted from TradingView's built-in Bollinger Bands indicator (open-source, Mozilla Public License 2.0).

## Phase 2 — Generalize the moving averages (the main "make it generic" change)

- [ ] Move the existing `ma(source, length, _type) =>` switch function from the Bollinger Bands section to a shared spot near the top of the file, above the moving-average section. Pine requires a function to be defined before its first call, and both sections need it now.
- [ ] Rename `groupEMA1`..`groupEMA4` (and the `"EMA 1"`.."EMA 4"` strings, and `enableEMA1..4`) to `groupMA1`..`groupMA4` / `"MA 1"`.."MA 4"` / `enableMA1..4`. Search the *whole* file for each name, not just its declaration — the `enableEMA1`-style names are also used later in the four `plot()` calls, and a partial rename won't compile.
- [ ] For each of the 4 MA groups, add a `"Type"` input using `input.string` with `options=["SMA", "EMA", "SMMA (RMA)", "WMA", "VWMA"]`, styled exactly like the existing Bollinger Bands "Basis MA Type" input. Default all four to `"EMA"` so behavior is unchanged unless someone touches the new dropdown.
- [ ] Replace the four `ta.ema(src, lenN)` calls with `ma(src, lenN, typeN)` calls using the shared function.
- [ ] Keep default lengths (9/13/50/200) and colors (blue/aqua/green/red) exactly as they are.

## Phase 3 — Real reference levels + VWAP

- [ ] Add `enablePDHL`, `enablePWHL`, `enablePMHL` (Previous Day / Week / Month High-Low) to the `Features` group. Default `enablePDHL` to `true`, the other two to `false` — three extra levels on by default is more clutter than most people want.
- [ ] Give each its own settings group ("Previous Day High/Low", etc.) with color, width, and style inputs matching the existing Daily High/Low group's pattern — no enable toggle inside these groups (see Convention above).
- [ ] Implement each with the non-repainting pattern, using names that won't collide with the existing `dayHigh`/`dayLow` variables — `[pdHigh, pdLow] = request.security(syminfo.tickerid, "D", [high[1], low[1]], lookahead=barmerge.lookahead_on)` (swap `"D"` for `"W"` / `"M"`, rename to `pwHigh`/`pwLow` and `pmHigh`/`pmLow`), plotted as levels spanning the current period.
- [ ] Guard against `na`: on symbols/timeframes without a full prior period of history yet, `request.security` can return `na` for these — don't draw or extend a line when the value is `na`.
- [ ] Add `enableVWAP` to the `Features` group and a `"VWAP"` settings group for its detail inputs (an optional bands toggle plus a StdDev multiplier, mirroring the Bollinger Bands StdDev input). Use the built-in `ta.vwap()` for the calculation rather than hand-rolling cumulative price/volume sums — it already resets on session boundaries correctly, which is the easiest part of a VWAP to get subtly wrong by hand.

## Phase 4 — Alerts + dashboard

- [ ] Add alert conditions using `alertcondition()` only, matching the existing Inside Bar pattern — don't mix in the newer `alert()` call — for: price crossing the Bollinger Bands, and price breaking the previous day's high or low.
- [ ] Add MA crossover alerts, but scope them to consecutive pairs only — MA1/MA2, MA2/MA3, MA3/MA4 — rather than all six possible pairings, so the alert list stays manageable.
- [ ] Add `enableDashboard` to the `Features` group ("Show Dashboard"). Its own settings group can hold a `tablePosition` input (`input.string` with the four corner options) and basic text/background color inputs, for consistency with how configurable the rest of the script is.
- [ ] Build the `table.new()` dashboard itself: price vs. each enabled MA, BB position, and distance from the previous day's high/low. Only show rows for features that are currently enabled elsewhere in the script — don't show a stale or zeroed-out row for a disabled feature. Keep it compact.

## Phase 5 — Outside Bar (nice to have, skip if short on time)

- [ ] Add an Outside Bar toggle mirroring the existing Inside Bar implementation's style exactly (same `barstate.isconfirmed` guard, same barcolor/plotshape/alertcondition pattern), using `high > high[1] and low < low[1]`.

## Phase 6 — Repo packaging

- [ ] Write `README.md`: short project description, a feature list that explains *why* each major toggle exists (not just a settings dump), a "getting started" section (how to add the script to a TradingView chart), a brief "not financial advice — for educational/informational purposes" disclaimer, and an HTML comment placeholder `<!-- screenshot/GIF goes here -->` where I'll drop in a real screenshot later.
- [ ] Write a `LICENSE` file using Mozilla Public License 2.0 (matching the license of the reused Bollinger Bands code), and mention this choice in the README.
- [ ] Write a minimal `.gitignore` (editor/OS cruft — there's no build output for Pine).
- [ ] Write a `CHANGELOG.md` with one entry summarizing this upgrade (list the features added/fixed), and add a version comment (e.g. `// Version 2.0.0`) directly under the `indicator()` line in the script.
- [ ] In the README, suggest 5–8 GitHub topics (e.g. `tradingview`, `pine-script`, `indicator`, `technical-analysis`, `trading-tools`) for me to add manually in the repo settings.

## When you're done

- [ ] Confirm `//@version=6` is still the literal first line of the file with nothing before it — not even a blank line — since that alone will stop it from compiling.
- [ ] Confirm every existing toggle still defaults to its current on/off state, and check TradingView's drawing-object budget has headroom (`max_lines_count` defaults to ~50; bump it in the `indicator()` declaration if the Daily + Previous Day/Week/Month lines running together ever get close).
- [ ] Give me a short manual QA checklist (5–10 items) to run through in the TradingView Pine Editor before publishing — e.g. "switch each MA's Type dropdown and confirm the line updates," "confirm an alert fires on a known historical inside bar."
- [ ] Give me a short summary of everything added or changed — good as a starting point for the CHANGELOG entry and for my own review — along with any assumptions you made along the way.
- [ ] Stop after committing locally — don't create or push to a remote GitHub repo. I'll review everything and do that step myself.
