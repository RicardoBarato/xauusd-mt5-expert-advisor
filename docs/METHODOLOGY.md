# Methodology

RB XAU/USD MT5 is designed around systematic research for gold/XAUUSD in MetaTrader 5.

## Research Focus

- XAUUSD/gold market behavior.
- Objective rule-based trading logic.
- Historical backtesting.
- Risk-controlled execution.
- Market-regime filtering.
- Repeatable validation workflow.

## Validation Expectations

Any serious evaluation should include:

- multiple historical periods;
- out-of-sample testing;
- walk-forward testing;
- Monte Carlo analysis;
- spread and slippage stress;
- commission and swap assumptions;
- broker-feed comparison;
- cross-engine comparison (MetaTrader 5 Strategy Tester against the TradingView port, see `TRADINGVIEW_CROSS_VALIDATION.md`);
- demo forward testing.

## What This Does Not Prove

No single backtest proves that a trading system will work in the future. A strategy can fail because of regime change, execution cost, parameter instability, overfitting, broker differences or poor risk control.
