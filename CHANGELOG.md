# Changelog

## 1.01: volatility gates read the current ATR

- Fixed `GetVolRatio` and `ATRAccelerationOK`: the 200-value ATR window was copied into a non-series array, so `buf[0]` was the **oldest** value of the window instead of the current one. The M5 volatility regime and the M1 ATR-acceleration trigger were therefore comparing ATR from about 200 bars ago with the window average. Both arrays are now set as series.
- This changes which trades the EA takes. Any backtest or preset produced with 1.00 is not comparable with 1.01 and must be re-run.
- The Pine Script port already used the current ATR and now matches the EA on this point; a comment marks the alignment.

## TradingView cross-validation port

- Added `pine/RB_XAUUSD_MT5.pine`, a Pine Script v6 strategy that reproduces the Expert Advisor rules for TradingView so the strategy can be backtested on a second engine and data feed.
- Re-implemented MetaTrader indicator formulas in Pine (SMA-based ATR, EMA-smoothed ADX) and rebuilt higher-timeframe EMAs as forming-bar values to match `CopyBuffer` semantics.
- Added platform-emulation inputs (server timezone, point size, contract size, lot limits, assumed spread) for what MetaTrader reads from the symbol.
- Added `docs/TRADINGVIEW_CROSS_VALIDATION.md` with the MQL5-to-Pine mapping, the structural divergences between the engines and a comparison protocol based on trade distributions rather than profit.
- Documented in `VALIDATION_LIMITS.md` that point-based thresholds do not scale with gold's price level and that evaluation-model sensitivity is a warning sign.
- Extended the publication checklist and contributing guide to cover TradingView artifacts and to require the Pine port to follow rule changes in the MQL5 source.

## Public documentation boundary update

- Clarified that the repository is a public source-only research release.
- Added project-status language explaining that the original private research cycle behind this export is closed for public purposes.
- Clarified that the project remains available for study, review, forks and independent evolution.
- Reinforced that public documentation does not publish performance metrics, real presets, raw reports, broker exports, account data or production parameters.

## Public performance metrics cleanup

- Removed public numeric backtest performance metrics from the current documentation surface.
- Reframed public backtest documentation around source-code review, methodology, validation limits and publication boundaries.
- Added publication-checklist items blocking public performance tables and comparison tables.

## 1.0.0-public-draft

- Created source-only public export for RB XAU/USD MT5.
- Added GPL-3.0-only license.
- Added sanitized MQL5 source file.
- Added public documentation for risk, methodology, architecture and validation limits.
- Added example inputs using fictitious values only.
- Excluded binaries, real presets, raw reports, broker exports and private research artifacts.
