# 03 — Trading Strategy

This document explains **how Jerrrix Poly-Kalshi Arbitrage Bot** finds and executes arbitrage between Kalshi and Polymarket, and how risk is managed.

---

## What Is Cross-Exchange Arbitrage?

In prediction markets, each contract pays **$1** if the outcome wins and **$0** if it loses. So for a binary event:

- **YES** + **NO** = **$1.00** (one of them always wins.)

**Arbitrage** appears when the same event is traded on two platforms and the total cost to buy one YES and one NO (across the two) is **less than $1.00**. The bot buys YES on one venue and NO on the other so that, no matter the outcome, it receives $1.00 per pair while paying less than $1.00.

---

## When Does the Bot Trade?

The bot considers an arbitrage opportunity **valid** when:

```text
Cost of YES (platform A) + Cost of NO (platform B) + Fees < $1.00
```

- It uses the **best ask** (lowest sell price) on each side.
- It includes **Kalshi fees** in the cost (Polymarket has no trading fees).
- It only trades when the total cost is below a **threshold** (e.g. 99.5¢), so that after fees there is still a small profit.

So the strategy is: **buy YES on one exchange and NO on the other** when the combined cost is below the threshold.

---

## Four Types of Arbitrage

The bot supports four patterns:

| Type | Buy | Sell | Description |
|------|-----|------|-------------|
| **Poly YES, Kalshi NO** | Polymarket YES | Kalshi NO | Buy YES on Polymarket, NO on Kalshi. |
| **Kalshi YES, Poly NO** | Kalshi YES | Polymarket NO | Buy YES on Kalshi, NO on Polymarket. |
| **Poly same market** | Polymarket YES + NO | — | Both legs on Polymarket (rare). |
| **Kalshi same market** | Kalshi YES + NO | — | Both legs on Kalshi (rare). |

In practice, most opportunities are **cross-platform**: Poly YES + Kalshi NO or Kalshi YES + Poly NO.

---

## Example (Numbers)

- **Polymarket YES** best ask: **40¢**
- **Kalshi NO** best ask: **50¢**
- **Kalshi fee** (on NO): **~2¢**

Total cost: 40¢ + 50¢ + 2¢ = **92¢**.  
Payout: **$1.00**.  
**Profit: 8¢ per contract** (before any slippage).

The bot only executes when the **effective** total cost (including fees) is below the configured threshold (e.g. 99.5¢), so that profit is positive.

---

## Fee Handling

- **Kalshi:** Fees are included in the bot’s cost calculation (so a 40¢ + 50¢ opportunity is evaluated as 40¢ + 50¢ + Kalshi fee). Formula used: `ceil(0.07 × contracts × price × (1 - price))` in cents.
- **Polymarket:** No trading fees; only Kalshi fees are added to the cost.

This way the bot does not treat a “barely profitable” situation as an arb when fees would wipe out the edge.

---

## Execution Flow

1. **Discovery**  
   The bot loads matched markets between Kalshi and Polymarket (same event, same type: moneyline, spread, total, etc.) and caches them.

2. **Orderbook stream**  
   It subscribes to Kalshi and Polymarket WebSockets and keeps a live view of best bid/ask and size for each market.

3. **Arb detection**  
   For each market pair it checks whether:
   - Poly YES + Kalshi NO (plus fees) < threshold, or  
   - Kalshi YES + Poly NO (plus fees) < threshold.

4. **Circuit breaker**  
   Before sending orders, it checks:
   - Per-market position limit  
   - Total position limit  
   - Daily loss limit  
   - Consecutive error count  

   If any limit is hit, the bot **stops placing new trades** until the circuit breaker allows again (e.g. after cooldown or reset).

5. **Concurrent legs**  
   When an arb is valid and allowed by the circuit breaker, the bot sends **both legs at once** (e.g. buy Poly YES and buy Kalshi NO in parallel) to reduce the chance that one side fills and the other doesn’t.

6. **Position and P&amp;L**  
   Fills are recorded; the bot tracks matched contracts, unmatched exposure (e.g. one leg filled, the other not), and profit/loss.

---

## Risk and Best Practices

- **Dry run first:** Use `DRY_RUN=1` until you are comfortable with logs and behavior. No real orders are sent in dry run.
- **Circuit breaker:** Keep it enabled (`CB_ENABLED=true`) and set `CB_MAX_POSITION_PER_MARKET`, `CB_MAX_TOTAL_POSITION`, and `CB_MAX_DAILY_LOSS` to values you are willing to lose.
- **Slippage:** Real fills can be worse than the quoted best ask; the bot uses the best available price at detection time, so actual profit can be lower or even negative if the book moves.
- **Partial fills:** If one leg fills and the other doesn’t (or fills less), you have **unhedged exposure** (directional risk). The bot tracks this; you may need to close the excess manually or with other tools.
- **API/connectivity:** Consecutive errors (e.g. API failures) trip the circuit breaker and halt trading until the cooldown/reset. Check logs and API status if the bot stops.

---

## Summary

| Concept | Meaning |
|--------|--------|
| **Arb condition** | YES cost (A) + NO cost (B) + fees < $1.00 |
| **Strategy** | Buy YES on one platform, NO on the other when the combined cost is below threshold. |
| **Fees** | Kalshi fees are included in cost; Polymarket has no trading fees. |
| **Safety** | Circuit breaker limits position size and daily loss; dry run avoids real orders. |

For configuration of thresholds and limits, see **[02 - Configuration](02-configuration.md)**. For setup and first run, see **[01 - Getting Started](01-getting-started.md)**.
