# Reversion Trader

**A  delta-neutral pairs trading bot for perpetual Crytocurrency contracts — with a live dashboard, multi-layer risk management, and zero local persistence.**

---
### SEE IT HERE 

[https://reversion-afab32929037.herokuapp.com/dashboard/](https://reversion-afab32929037.herokuapp.com/dashboard/) 

*Password for public viewing = public* 

Note: the application sleeps between runs to save compute budget so it takes 2-3 seconds to load up outside these runtimes.

## What Is This?

- The script monitors crypto pairs that have had historic correlation (mathematically co-integration).
- The algorithm detects statistical variations from these correlation norms, and measures this change with a rolling z-score. 
- Once the Z-score metric exceeds a thresold the algorithm automatically executes mean-reversion trades on both legs simultaneously. It goes long one asset and short another when the spread deviates from its historical norm — then closes when the spread reverts.
- There are layers of risk management to try mitigate occasions when the pair of crytocurrencies does not revert in a timly manner. 

---

```

## Risk Management

Risk is managed at three levels:

### 1. Position-Level NET PnL Stops
### 2. Cooldown System
### 3. Account-Level Circuit Breaker
### 4. Pyramiding Entries a time cooldown between adds

```
---

## Lessons Learned

Building and running this system live taught us more than any backtest ever could. Here's what we'd tell ourselves six months ago:

### 1. Cointegration Breaks — and It Breaks Fast

A pair that passes every statistical test on historical data can decorrelate overnight. SUI/LINK looked bulletproof in backtesting. Then a single protocol upgrade on one chain sent the spread into a one-way drift that never reverted. The 180-day cooldown freeze exists because we learned that "maybe it'll come back" is the most expensive sentence in pairs trading. If a pair stops you out, respect it and walk away.

### 2. Statelessness Isn't a Nice-to-Have — It's Survival

The original version persisted state to a JSON file on disk. On the third Heroku dyno restart in one week, we found the bot had opened duplicate positions because the state file was stale. Rebuilding everything from the exchange on every cycle added ~2 seconds of latency per cycle and eliminated an entire class of bugs. The exchange is the only source of truth that survives restarts, deploys, and your own mistakes.

### 3. Exchange-Level Stops Are Non-Negotiable

Your bot can crash. Your server can go down. Your internet can drop. If your only stop-loss logic lives in your Python process, you have no stop-loss. We place trigger-stop orders directly on Hyperliquid for every open position and refresh them every cycle. The in-process position stop is a first line of defense; the exchange-level stop is the parachute.

### 4. Sizing to Equity Is a Trap

Early versions sized each leg as a percentage of *current* equity. This meant that after a drawdown, position sizes shrank, and the bot needed a larger percentage move to recover — the classic "volatility drag" death spiral. Worse, after a winning streak, sizes ballooned right when mean-reversion probability was weakest. Fixed-fraction sizing based on starting equity per unit, gated by a hard margin check, was far more stable.

### 5. Don't Pyramid Into Chaos

Pyramiding (adding units as the z-score extends further) is theoretically optimal — you're adding at better prices. In practice, it's a leverage bomb. Without a z-score cap (we use 2.6) and a time cooldown between adds, the bot would happily stack 7 units into a spread that was diverging because of a structural break, not a temporary dislocation. The pyramid guards saved us from ourselves more than once.

### 6. Dont skip the dashboard

We almost skipped the dashboard. "We'll just check the logs." Within a week, the dashboard became the primary way we understood what the bot was doing. Seeing z-scores charted in real time, with entry thresholds overlaid, gave us intuition that log files never could. The 30 seconds it takes to glance at the dashboard has prevented more bad interventions than any single risk rule.

### 7. Alert Fatigue Is Real — Be Selective

The first version sent a phone notification on every entry, every exit and every cycle error. Within three days we were ignoring all of them. Now we only alert on: emergency closes, position stops, execution failures, and job-level crashes. If a notification doesn't require you to *do something*, it shouldn't interrupt your day.


---

## Disclaimer

This is live trading software. It is provided as-is with no warranty. 
This is no way a recommendation for trading or purchasing any of the ticker / assets / derivatives displayed or mentioned.
Past performance of statistical arbitrage strategies does not guarantee future results. 
The historical performance of this strategy is not strong, giving it more value as a public project. It is not wise to replicate this strategy.  
