# Changelog

## 2.0.0

Upgrade from a personal script to a general-purpose, publishable toolkit.

**Added**
- Selectable moving average type (SMA, EMA, SMMA (RMA), WMA, VWMA) for each of the 4 MAs, previously hardcoded to EMA.
- Previous Day / Week / Month High-Low reference levels, computed non-repainting via `request.security`.
- VWAP with optional standard-deviation bands.
- Outside Bar detection, mirroring the existing Inside Bar feature.
- Alert conditions for MA crossovers (consecutive pairs), Bollinger Band crosses, and previous-day high/low breaks.
- A compact on-chart dashboard summarizing price vs. active indicators.

**Fixed**
- Removed a stray duplicate `//@version=6` and commented-out `indicator()` call left over from merging two scripts into one.
- Removed a dead line-cleanup block that could never execute.
- Relabeled the shared EMA source input and the Daily High/Low toggle so their names match what they actually do.

**Changed**
- Renamed EMA 1–4 to MA 1–4 throughout, since each is now a configurable MA type rather than a fixed EMA.
