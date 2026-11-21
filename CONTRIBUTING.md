# Contributing

Contributions are welcome only when they preserve the source-only and research-focused nature of the project.

## Before Opening A Contribution

Check that your change does not include:

- compiled `.ex5` files;
- real `.set` presets;
- raw MT5 reports;
- broker exports;
- account data;
- credentials;
- local paths;
- performance claims written as guarantees.

## Preferred Contributions

- documentation improvements;
- code readability improvements;
- safer defaults;
- test methodology notes;
- risk-disclosure improvements;
- fidelity fixes to the Pine Script port where it drifts from the MQL5 rules (the MQL5 file is the reference);
- bug fixes that do not expose private research data.

Changes to the trading rules must land in both `src/RB_XAUUSD_MT5.mq5` and `pine/RB_XAUUSD_MT5.pine`, or explain why the port cannot follow.

## Not Accepted

- signal-selling language;
- profit guarantees;
- raw backtest reports with sensitive metadata (MetaTrader or TradingView);
- broker/account-specific configuration;
- compiled binaries.
