# Validation Limits

Validation is a process, not a single backtest.

## Known Limits

- Historical tests may overstate live performance.
- MT5 Strategy Tester depends on local data quality.
- Broker feeds can differ.
- Spreads and slippage can be worse in live conditions.
- Gold/XAUUSD can gap or reprice abruptly.
- Parameter choices can be overfit.
- Low trade count can make results statistically fragile.
- Thresholds expressed in points (`MinATR_points`, `MinSL_points`, `SpreadMax_points`) are absolute dollar distances and do not scale with gold's price level, so a parameter set validated at one price level behaves differently at another. EA 1.02 offers `threshold_mode = 1` as a volatility-relative alternative.
- A stack of many gates can make a strategy trade so rarely that no statistic about it is meaningful. Find which gate rejects most candidate bars before tuning any single one.
- Results that depend on the evaluation model (every tick versus bar close) are evidence of execution sensitivity, not of an edge.

## Recommended Additional Tests

- Out-of-sample periods.
- Walk-forward validation.
- Monte Carlo reshuffling and bootstrapping.
- Spread and slippage stress.
- Different brokers or data feeds.
- A second backtest engine (the Pine Script port on TradingView), comparing trade distributions rather than profit.
- Demo forward testing.
- Failure-mode review for losing years and drawdown clusters.

## Publication Limit

This public source-only export is not a production certification. It is a research artifact for review and further testing.
