# Architecture

This document describes the `XAUUSD MT5 Expert Advisor` source release at a high level without publishing private presets, production parameters or non-public research artifacts.

## Main Components

### Signal Module

Evaluates price action, trend and breakout/retest conditions using market data available inside MetaTrader 5.

### Regime And Filter Module

Applies volatility, trend-quality and session filters to avoid some unfavorable conditions.

### Risk Module

Calculates risk-based position sizing, drawdown guards and operational limits.

### Execution Module

Uses the MT5 trading API to submit orders only when the configured safety flags allow trading.

### Operational Protection Module

Includes symbol checks, spread checks, session checks, cooldowns and limits on trading frequency.

### Diagnostics

The source includes runtime checks and control flow suitable for Strategy Tester review. This public export does not include raw logs or private diagnostic reports.

## Source Files

```text
src/RB_XAUUSD_MT5.mq5      reference implementation (MQL5)
pine/RB_XAUUSD_MT5.pine    Pine Script v6 port for TradingView cross-validation
```

### TradingView Port

The Pine strategy mirrors the module structure above with the same input names and the same gate order inside the tick pipeline, so the two sources can be read side by side. MetaTrader indicator formulas are re-implemented where Pine's built-ins differ (ATR as a simple average of true range, ADX with exponential smoothing). Higher-timeframe EMAs are rebuilt as forming-bar values to match `CopyBuffer(..., 0, 0, 1)`; other higher-timeframe indicators use the last completed bar.

The port is a validation tool, not a second product. The MQL5 file stays the reference and the Pine file follows it. Divergences that cannot be removed (tick versus bar-close evaluation, missing spread series, broker-specific symbol settings) are listed in `TRADINGVIEW_CROSS_VALIDATION.md`.

## Notes

The public repository intentionally excludes production presets and private research artifacts. Users should treat the included defaults as research defaults, not as a recommended live configuration.
