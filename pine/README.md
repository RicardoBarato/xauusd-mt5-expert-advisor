# Pine Script Port

`RB_XAUUSD_MT5.pine` is a Pine Script v6 strategy that reproduces the rules of `src/RB_XAUUSD_MT5.mq5` for TradingView, so the Expert Advisor can be cross-validated on a second backtest engine and a second data feed.

- Attach it to a **1-minute** XAUUSD chart.
- Input names match the MQL5 source. The *Platform emulation* group supplies what MetaTrader normally reads from the symbol (timezone, point size, contract size, lot limits, assumed spread).
- Read [docs/TRADINGVIEW_CROSS_VALIDATION.md](../docs/TRADINGVIEW_CROSS_VALIDATION.md) for the mapping, the known divergences and the comparison protocol before interpreting any result.

The MQL5 source remains the reference implementation. This port is research tooling, not a trading product, and no performance figures are published for it.

License: GPL-3.0-only, same as the rest of the repository.
