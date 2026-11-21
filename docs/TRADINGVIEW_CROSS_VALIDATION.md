# TradingView Cross-Validation

This document describes the Pine Script v6 port of the Expert Advisor and how to use it as an **independent second backtest engine** for the MQL5 source. It explains what was ported, how faithfully, where the two platforms necessarily diverge, and how to compare results without fooling yourself.

It does not publish performance tables, presets or raw reports. The public research boundary described in [Backtest Research Summary](BACKTEST_RESEARCH_SUMMARY.md) applies to TradingView results exactly as it applies to MetaTrader 5 results.

## Why A Second Engine

A single backtest engine on a single data feed cannot tell you whether an edge is real or an artifact of that engine and feed. The MetaTrader 5 Strategy Tester depends on the broker's history, its tick-modeling mode and broker-specific symbol settings. TradingView's strategy tester runs on a different feed (for example `OANDA:XAUUSD`), a different fill model and a different indicator library.

Running the same rules on both and comparing the **shape** of the results (trade count, direction mix, holding time, where trades cluster in time) is a cheap robustness check. Agreement does not prove an edge. Strong disagreement is a warning that the strategy depends on execution details.

## What Is Included

```text
pine/
`-- RB_XAUUSD_MT5.pine   Pine Script v6 strategy, GPL-3.0-only
```

The Pine strategy keeps the input names of the MQL5 source so a parameter set can be read side by side. It adds one group of **platform emulation** inputs that have no equivalent in the EA because MetaTrader supplies them from the symbol:

| Pine input | MT5 origin | Default |
|---|---|---|
| `Server timezone` | broker server time | `Europe/Athens` (EET/EEST, the most common MT5 server clock) |
| `point_size` | `SYMBOL_POINT` | `0.01` (2-digit XAUUSD; use `0.001` for 3-digit brokers) |
| `units_per_lot` | contract size | `100` (1.00 lot = 100 oz) |
| `lot_min`, `lot_step`, `lot_max` | `SYMBOL_VOLUME_*` | `0.01`, `0.01`, `100` |
| `assumed_spread_points` | live bid/ask | `25` (used for SL/TP geometry only, see below) |

## How To Run It

1. Open TradingView, create a new Pine Script and paste `pine/RB_XAUUSD_MT5.pine`.
2. Attach it to a **1-minute** XAUUSD chart (`OANDA:XAUUSD`, `FOREXCOM:XAUUSD`, `FX:XAUUSD` or your broker's symbol if TradingView lists it). The script refuses other timeframes because the EA's signal logic runs on M1.
3. In the strategy *Properties* tab keep `Process orders on close` enabled (the script sets it). Set `Slippage` in ticks to roughly half your broker's typical spread to charge the spread on fills, and set a commission if your broker charges one.
4. Use **Deep Backtesting** to cover more than the visible chart history. On a Premium plan, 1-minute history for gold reaches back several years.
5. Export the trade list and compare it with the Strategy Tester report as described below. Keep both exports outside the public repository.

The same strategy can be driven programmatically through TradingView's MCP tools (`validate_pine_script`, `run_pine_backtest`, `get_ohlcv`), which is how the port was compiled and sanity-checked. Those tools return the same report fields as the on-chart tester.

## Mapping From MQL5 To Pine

| EA component | MQL5 implementation | Pine port | Fidelity |
|---|---|---|---|
| ATR (all timeframes) | `iATR` = **simple** moving average of true range | `ta.sma(ta.tr(true), len)` | Exact formula. `ta.atr` (Wilder RMA) is deliberately **not** used. |
| ADX (M1 and M15) | `iADX` (ADX.mq5): per-bar +DI/-DI ratios smoothed with EMA, DX smoothed with EMA | Re-implemented step by step in `f_mt5_adx` | Exact formula. `ta.dmi` (Wilder) is deliberately **not** used. |
| Bollinger Bands | SMA with population standard deviation | `ta.sma` + `ta.stdev` (biased) | Exact formula. |
| EMA bias H1, MTF confirm EMAs, macro EMAs | `CopyBuffer(..., 0, 0, 1)` reads the **forming** HTF bar | Last completed HTF EMA via `request.security(... [1], lookahead_on)` then one EMA step with the current close | Exact reconstruction of the forming value. |
| M15 structure levels | `iHighest/iLowest(..., lookback, shift 1)` | `request.security("15", ta.highest(high, n)[1], lookahead_on)` | Exact. |
| M5 ATR regime, M5 squeeze, M15 ADX | Forming HTF bar values | **Last completed** HTF bar values | Approximation, see divergences. |
| Session filter, hour/weekday masks, risk multipliers | Broker server time | Same logic on the configured timezone; weekday renumbered to MT5 convention (0 = Sunday) | Exact given the right timezone. |
| Daily reset at `session_reset_hour` | `GetSessionDayKey` | Same key computed in the configured timezone | Exact. |
| Risk sizing and lot normalization | `CalcLots` / `NormalizeLots` | Same math; `qty = lots * units_per_lot` | Exact, including the "below minimum lot = no trade" rule. |
| Stop-loss cooldown and daily R limit | `OnTradeTransaction` on `DEAL_REASON_SL` | Closed trades with exit comment `SL` and negative profit | Equivalent. |
| Squeeze release / cycle memory state machine | Advances only when earlier gates pass | Same ordering preserved | Exact quirk reproduction. |
| Retest entries, extension guard | `TryRetestEntry`, `ArmRetest` | Ported with the same expiry and invalidation rules | Equivalent. |
| Break-even and ATR trailing | `PositionModify` each tick | `strategy.exit` re-issued when the stop moves | Equivalent at bar resolution. |
| Equity risk guard and hard stop | `ACCOUNT_EQUITY` | `strategy.equity` | Equivalent. |
| Spread check | `SpreadOK()` on live bid/ask | Not available: TradingView has one price series | Removed, see divergences. |
| Order-failure cooldown | Broker rejections | Not applicable | Removed. |
| `SYMBOL_TRADE_STOPS_LEVEL` check | Broker minimum stop distance | Not applicable | Removed. |
| Symbol guard `InpAllowedSymbol` | `_Symbol` | User attaches to the right chart | Removed. |

## Known Divergences

These are structural and cannot be removed by changing inputs. Read them before interpreting any difference between the two engines.

### 1. Tick evaluation versus bar-close evaluation

The EA evaluates every tick and enters at the **first** tick where all conditions hold, using the forming M1 bar's high, low and close at that instant. Pine evaluates once per historical bar using the bar's final OHLC and fills at the bar close (`process_orders_on_close = true`).

Consequences:

- The EA can enter earlier inside a bar, at a different price, or enter on a bar that no longer qualifies by its close. The port only trades bars that qualify at the close.
- The impulse test (`range >= impulse_mult * ATR`) and `close_pos` are stricter at bar close than at an intrabar moment.
- Expect the EA to produce **more** trades than the port on identical data.

TradingView's `Calculate on every history tick` property approximates intrabar evaluation. In the sanity runs it changed the trade count materially, which is itself useful information: the strategy is sensitive to the evaluation model.

### 2. Forming versus completed higher-timeframe bars

EMAs are rebuilt exactly. ATR, ADX and Bollinger/Keltner values on M5 and M15 use the last **completed** HTF bar, while the EA reads the forming bar. Early in an HTF bar the forming ATR is biased low (small true range so far), so the EA's volatility regime and squeeze tests are slightly noisier than the port's.

### 3. Spread

MetaTrader fills buys at ask and triggers long stops on bid. TradingView has one series. The port uses `assumed_spread_points` only to reproduce the EA's SL/TP distances (which are measured from ask for longs). To also pay the spread on fills, set the strategy `Slippage` property to about half the spread in ticks: entries and stop exits are then both slipped adversely, which approximates one spread per round trip.

The EA's `SpreadMax_points` filter cannot be reproduced. On a spread-widening event the EA stays flat while the port trades.

### 4. Data feed and time

- `OANDA:XAUUSD` is not your broker's feed. Highs, lows and especially gaps around the daily rollover differ.
- TradingView forex symbols have a different session boundary than most MT5 brokers; the daily reset hour and session windows are interpreted in the configured timezone, so match it to your broker's server time (EET/EEST for most, UTC for some).
- Deep Backtesting starts indicators cold at `date_from`. Allow a warm-up (the slow EMA on H1 and the 200-bar ATR average on M5 need days, not minutes) before trusting trades.

### 5. Indicator initialization

Pine's `ta.ema` and MT5's `iMA` seed their EMAs differently on the first bars. After a few hundred bars the values converge. This only matters at the start of a Deep Backtesting range.

## Recommended Comparison Protocol

Compare distributions, not profit figures.

1. Run both engines on the same calendar range with the same parameter set and the same timezone assumption.
2. Compare trade count per month, long/short mix, average holding time and the share of exits by stop versus target. These should be in the same neighborhood. If the EA shows several times more trades than the port, divergence 1 dominates and the strategy depends on intrabar timing.
3. Re-run the port with `Calculate on every history tick` enabled and see whether it moves toward the EA.
4. Stress both with wider spread/slippage and with commission. Breakout strategies on M1 are cost-sensitive.
5. Run the port on at least two TradingView feeds (for example OANDA and FOREXCOM). Disagreement between feeds on the same engine bounds how much feed noise to expect between TradingView and your broker.
6. Treat any single-feed, single-engine improvement as unproven until the other engine agrees in direction.

## Observations From Porting

These are qualitative notes from compiling and sanity-running the port on OANDA 1-minute gold history. They are not performance claims and no figures are published.

- **Point-based thresholds do not scale with price.** `MinATR_points`, `MinSL_points`, `SpreadMax_points` and `break_even_lock_points` are absolute dollar distances. Gold's price level changed materially between 2024 and 2026, so a 1-minute ATR floor that is permissive at one price level is restrictive at another. In the sanity runs the port produced no trades for months at the lower price level and started trading as price and volatility rose; lowering the ATR floor for that period made trades appear. Anyone re-validating the EA across years should consider expressing these floors as a fraction of price or of a longer ATR.
- **The strategy is highly selective.** With the public defaults it trades far less often than `max_trades_day` suggests. Statistics from a few dozen trades are fragile; see [Validation Limits](VALIDATION_LIMITS.md).
- **The evaluation model matters.** Enabling intrabar simulation changed the number of trades noticeably. Results that only hold under one evaluation model should not be trusted.
- **Lot normalization bites small accounts.** With structural stops of tens of dollars and a fixed-fraction risk below 1 %, the computed size on a 10 000 unit account rounds to the minimum lot or to zero. The port reproduces this; account size is part of the strategy's behavior, not just a scaling factor.

## What This Port Is Not

- Not a trading product, signal or recommendation.
- Not a replacement for the MQL5 source. The EA remains the reference implementation.
- Not evidence of profitability on either platform.
- Not a published preset. The Pine defaults are the research defaults of the public source, not a tuned configuration.

## Publication Boundary

TradingView strategy reports, screenshots of the Strategy Tester, exported trade lists and saved-script identifiers are private research artifacts. They stay outside the repository, exactly like MetaTrader reports. See [Publication Checklist](PUBLICATION_CHECKLIST.md).
