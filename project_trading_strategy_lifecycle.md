---
name: Trading Strategy Lifecycle
description: Mitch's philosophy for the day-trading engine — paper mode is crash-and-burn, no restrictions; strategies graduate from backtest → paper → cash
type: project
originSessionId: cb96d6a7-e0ee-480a-a748-e0207e02ed39
---
Paper mode = crash and burn sandbox. No restrictions on what to try.

**Strategy lifecycle: Backtest → Paper → Cash. Drop losers ruthlessly.**

1. Backtest on 1yr historical 15m data — must show profit factor > 1.0 and Sharpe > 0
2. Run on paper for 2–4 weeks minimum — real market conditions, real fills, real slippage
3. Only strategies winning in paper graduate to live cash
4. No instrument or strategy restrictions in paper mode — it costs nothing to test

**Why:** Mitch explicitly said "paper account, crash and burn, nothing to lose." Applying SPY-only restrictions or removing strategies from paper mode is wrong — let everything run and let the data decide.

**How to apply:** When Mitch asks whether to restrict a strategy to certain symbols or instruments in paper mode, say no — run it on everything, watch the results, restrict only in live cash mode based on paper performance.

Current active strategies (all paper as of 2026-05-12):
- candle_patterns (volume gate 1.2x, session filter, EMA pullback)
- vwap_breakout (all instruments, no symbol restriction in paper)
- rsi_mean_reversion (proven winner — +$51.73 over 1yr backtest)
- gpirl_microstructure (new, OHLC proxy backtest shows SPY edge, live L2 pending evaluation)
