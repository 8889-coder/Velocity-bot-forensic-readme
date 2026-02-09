# velocity_bot: Polymarket Spread Strategy — Forensic Analysis

> **889 real-money trades. 1 catastrophic failure. 6 critical errors identified. 5,000 Monte Carlo simulations.**
>
> Built months before ClawdBot/OpenClaw went viral. This is what they're not telling you.

---

## What This Repo Contains

- **Forensic post-mortem** of a catastrophic strategy rewrite that destroyed a 500%+ returning bot
- **6 critical errors** that will kill most ClawdBot/OpenClaw Polymarket bots
- **Battle-tested parameters** extracted from 889 real-money trades
- **Monte Carlo methodology** (5,000 sims) for parameter calibration
- **The BOT RULE** — operational discipline framework for automated trading

This is not theory. This is scar tissue converted to documentation.

---

## Background

In mid-2025, I built `velocity_bot` to farm temporal arbitrage on Polymarket's 15-minute crypto prediction markets (BTC, ETH, SOL). The strategy exploits the ~30-second lag between Polymarket contract prices and Binance/exchange spot prices.

**Same strategy now generating $75K-$115K/week for ClawdBot users.** I was doing it with $3 starting capital before anyone had heard of ClawdBot.

### The Results

| Phase | Capital | Return | Trades | Status |
|-------|---------|--------|--------|--------|
| v1.0-v2.9 | $3 → $18 | +500% | ~200 | ✅ Proven |
| v3.0-v4.0 | $18 → ~$4 | -78% | ~400 | ❌ Catastrophic |
| v5.0.0 | $4 (restored) | Recovering | 289+ | ✅ Proven params restored |

### Why the Catastrophic Failure Matters More Than the Wins

If you only show green P&L, you're not educating anyone. You're marketing.

The v4.0 disaster happened because **a single unvalidated suggestion caused a full strategy rewrite**. Proven logic was replaced with untested theory. The bot went from printing money to bleeding it overnight.

**This is exactly what will happen to most ClawdBot users.** They'll have a good week, tweak something, and lose everything. The tool doesn't protect you from yourself.

---

## The 6 Critical Errors

These were identified through forensic analysis of the v3.0-v4.0 trade log. Every one of these is a live risk for anyone running a Polymarket bot right now.

### Error 1: Removed TRAIL Mechanism
**Impact:** Eliminated ~$20/day edge
**What happened:** The trailing stop (arm at +5¢, drop at -8¢) was the primary profit capture mechanism. It was removed in favor of "smarter" exit logic that had never been tested.
**Lesson:** If a mechanism is generating measurable edge, never remove it without A/B testing on 50+ trades minimum.

### Error 2: Removed STOP Loss
**Impact:** Enabled full position wipeout
**What happened:** Hard stop at -50% was removed because "the new logic handles risk differently." It doesn't. Markets don't care about your logic.
**Lesson:** STOP losses are non-negotiable. They're not part of the strategy. They're the floor under the strategy.

### Error 3: Forced 10% Kelly Floor
**Impact:** 16x overexposure on every trade
**What happened:** The Kelly criterion calculated optimal position size at 0.6% of bankroll. This was overridden to a 10% minimum "to make meaningful returns." That's not how Kelly works. Kelly is the MAXIMUM you should bet, not the minimum.
**Lesson:** If your calculated Kelly fraction is 0.6%, that IS your position size. If you want bigger positions, get a bigger edge, not a bigger override.

### Error 4: Validated on 4 Winning Trades
**Impact:** Survivorship bias masquerading as validation
**What happened:** New strategy was declared "working" after 4 consecutive wins. That's a coin-flip streak, not a strategy. You need 50+ trades minimum to distinguish signal from noise.
**Lesson:** The $100 → $347 overnight results everyone's screenshotting? That's 4 trades. Call me after 200.

### Error 5: Deployed 7 Versions Live with Zero Backtest
**Impact:** Every code change was tested with real money
**What happened:** In the rush to "improve," seven consecutive versions were pushed to production without backtesting against historical data. Each one compounded the previous errors.
**Lesson:** If you can't backtest it, you can't deploy it. Period. Your Polymarket bot is not a "move fast and break things" startup. It's a machine that holds your money.

### Error 6: Optimized on Single Example
**Impact:** Overfitting to noise
**What happened:** Parameters were tuned to perfectly fit one specific trade sequence. Performed beautifully on that sequence. Failed on everything else.
**Lesson:** Optimization requires out-of-sample validation. If your parameters were "discovered" by running your bot for one afternoon, you've discovered nothing.

---

## Battle-Tested Parameters (v5.0.0)

Extracted from 889 trades. Validated across 5,000 Monte Carlo simulations. Survived catastrophic failure and recovery.

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| BOUNCE | 4¢ minimum | Price must reverse 4¢+ before entry signal fires. Filters noise from signal. |
| BUY_ZONE | 15¢ - 32¢ | Entry only in this range. Below 15¢ = no momentum. Above 32¢ = overpaying for upside. |
| TRAIL_ARM | +5¢ from entry | Trailing stop activates after 5¢ profit. Below this, position managed by STOP only. |
| TRAIL_DROP | -8¢ from peak | Exit when price drops 8¢ from highest point since entry. Captures trends, exits reversals. |
| STOP | -50% | Hard floor. Non-negotiable. If position is down 50%, exit immediately. No exceptions. |
| CEILING | 88¢ | Never enter above 88¢. Contracts approaching $1.00 have asymmetric downside risk. |
| RUNWAY | 240 seconds | Minimum time between trades. Prevents overtrading during volatility spikes. |
| KELLY | Calculated per trade | Position size = Kelly criterion output. Never override. Never floor. |

### Why These Specific Values

Each parameter was calibrated through the Monte Carlo process:

1. **Set initial range** based on market microstructure analysis
2. **Run 5,000 simulations** across historical 15-minute market data
3. **Measure**: Sharpe ratio, max drawdown, win rate, profit factor
4. **Select**: Parameter set that maximizes Sharpe while keeping max drawdown < 30%
5. **Validate**: Out-of-sample test on withheld data
6. **Confirm**: Live deployment for 50+ trades before declaring validated

Parameters that survive this process are not guesses. They're evidence.

---

## The BOT RULE

Written after the catastrophic failure. Tattooed on the codebase.

```
1. No strategy changes without data
2. Collect 50+ trades before tuning ANY parameter
3. Incremental changes only (one parameter at a time)
4. Never deploy untested logic with real money
5. If the bot is profitable, the DEFAULT is to change nothing
6. Every change must have a hypothesis, a measurement plan, and a rollback trigger
```

ClawdBot/OpenClaw won't teach you this. These rules don't exist in the docs, the GitHub skills, or the viral tweets. They exist in the trade log of someone who learned by losing.

---

## Monte Carlo Methodology

### Setup
- **Simulations**: 5,000 per parameter sweep
- **Data**: Historical Polymarket 15-minute market prices (BTC, ETH, SOL)
- **Method**: Bootstrap resampling of actual trade sequences
- **Objective**: Maximize risk-adjusted return (Sharpe) subject to max drawdown constraint

### Process
```
For each parameter combination:
    For each of 5,000 simulations:
        1. Sample trade sequence (with replacement)
        2. Apply strategy with current parameters
        3. Record: final PnL, max drawdown, win rate, Sharpe
    Calculate: mean and 5th percentile of all metrics
    Reject if: 5th percentile max drawdown > 30%
    Rank remaining by: mean Sharpe ratio
```

### Key Findings
- **TRAIL mechanism accounts for ~60% of total edge**. Removing it (Error 1) was catastrophic.
- **Kelly overrides destroy performance non-linearly**. A 2x override (1.2% vs 0.6%) doesn't double risk — it quadruples ruin probability.
- **BUY_ZONE upper bound (32¢) is the most sensitive parameter**. Moving it to 40¢ drops Sharpe by 35%.
- **RUNWAY (240s) prevents ~40% of losing trades**. Most impulse entries within 240s of last trade are noise.

---

## What This Means for ClawdBot/OpenClaw Users

You don't need:
- ❌ A $20,000 Mac Mini setup
- ❌ 145,000 GitHub stars of wrapper code
- ❌ Another "33 automations" Medium article

You need:
- ✅ Calibrated parameters for YOUR capital level
- ✅ A Monte Carlo framework (not vibes)
- ✅ Hard risk limits that you don't override
- ✅ The discipline to not touch a working bot

### Get an Audit

| Tier | Price | What You Get |
|------|-------|-------------|
| Parameter Set | $250 USDC | Proven parameters + catastrophic failure briefing (PDF) |
| Full Audit | $750 USDC | Your config reviewed + Monte Carlo calibration for your capital + 30 days Telegram support |
| Done-for-you | $2,000 USDC | Deployment on your infrastructure + ongoing optimization + revenue share |

**Contact:** [@PJV_Blacktrace on Telegram] or (https://t.me/eicisbot) if you like bots.

---

## About

**Blacktrace** — Systematic trading research since 2004. 20 years quantitative fund management. Melbourne, Australia.

Built velocity_bot months before ClawdBot went viral. Same strategy. Different discipline.

The edge isn't the code. It's the post-mortem.

---

*This analysis is for educational purposes. Not financial advice. Trading prediction markets involves substantial risk of loss. Past performance does not guarantee future results.*
