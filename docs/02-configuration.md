# 02 — Configuration

This document describes all **environment variables** and configuration options for **Jerrrix Poly-Kalshi Arbitrage Bot**. Set these in your `.env` file or export them before running the bot.

---

## Required Variables

These must be set for the bot to start.

| Variable | Description | Example |
|----------|-------------|--------|
| `KALSHI_API_KEY_ID` | Your Kalshi API key ID (from Kalshi Settings → API Keys). | `a1b2c3d4-...` |
| `KALSHI_PRIVATE_KEY_PATH` | Full path to the RSA private key PEM file for Kalshi API signing. | `/home/user/kalshi_private_key.pem` or `C:\keys\kalshi.pem` |
| `POLY_PRIVATE_KEY` | Ethereum private key for your Polymarket wallet, with `0x` prefix. | `0xabc123...` |
| `POLY_FUNDER` | Your Polymarket wallet address (same as the one derived from the private key), with `0x`. | `0x1234...abcd` |

---

## Bot Behavior

| Variable | Default | Description |
|----------|--------|-------------|
| `DRY_RUN` | `1` | **`1`** = paper trading (no real orders). **`0`** = live execution. Always start with `1` and only set `0` when you are ready to trade with real money. |
| `RUST_LOG` | `info` | Log level: `error`, `warn`, `info`, `debug`, `trace`. Use `debug` or `trace` for more detail. |
| `FORCE_DISCOVERY` | `0` | **`1`** = ignore cache and re-fetch all Kalshi↔Polymarket market mappings. Use when markets or listings have changed. |
| `PRICE_LOGGING` | `0` | **`1`** = enable verbose price-update logging. Can be noisy. |

---

## Test Mode (Synthetic Arb)

Use these only for testing the execution path; they inject a fake arb so the bot tries to execute.

| Variable | Default | Description |
|----------|--------|-------------|
| `TEST_ARB` | `0` | **`1`** = inject a synthetic arb opportunity after a short delay (for testing). |
| `TEST_ARB_TYPE` | `poly_yes_kalshi_no` | Type of synthetic arb. Options: `poly_yes_kalshi_no`, `kalshi_yes_poly_no`, `poly_same_market`, `kalshi_same_market`. |

Example (test execution path in dry run):

```bash
TEST_ARB=1 DRY_RUN=1 dotenvx run -- cargo run --release
```

---

## Circuit Breaker (Risk Limits)

The circuit breaker stops the bot from trading when limits are hit (e.g. too many contracts or too much loss).

| Variable | Default | Description |
|----------|--------|-------------|
| `CB_ENABLED` | `true` | **`true`** = circuit breaker on. **`false`** = disable (not recommended for live trading). |
| `CB_MAX_POSITION_PER_MARKET` | `100` | Maximum contracts per single market. |
| `CB_MAX_TOTAL_POSITION` | `500` | Maximum total contracts across all markets. |
| `CB_MAX_DAILY_LOSS` | `5000` | Max daily loss in **cents** before the bot halts (e.g. `5000` = $50). |
| `CB_MAX_CONSECUTIVE_ERRORS` | `5` | Number of consecutive errors (e.g. API failures) before the bot halts. |
| `CB_COOLDOWN_SECS` | `60` | Seconds to wait after the circuit breaker trips before allowing trading again. |

Tuning suggestions:

- **Conservative:** Lower `CB_MAX_POSITION_PER_MARKET` and `CB_MAX_TOTAL_POSITION`, lower `CB_MAX_DAILY_LOSS`.
- **More aggressive:** Raise these limits only if you understand the risk and have tested in dry run.

---

## Obtaining Credentials

### Kalshi

1. Log in at [Kalshi](https://kalshi.com).
2. Go to **Settings → API Keys**.
3. Create a new API key with **trading** permissions.
4. Download the private key (PEM file) and store it somewhere safe.
5. Copy the **API Key ID** into `KALSHI_API_KEY_ID`.
6. Set `KALSHI_PRIVATE_KEY_PATH` to the full path of that PEM file.

### Polymarket

1. Use an Ethereum wallet (e.g. MetaMask) that you will use for Polymarket on Polygon.
2. Export the **private key** (with `0x` prefix) → `POLY_PRIVATE_KEY`.
3. Use the **wallet address** (with `0x`) → `POLY_FUNDER`.
4. Fund the wallet with **USDC on Polygon** for live trading.

---

## Example `.env` Files

### Minimal (dry run)

```bash
KALSHI_API_KEY_ID=your_key_id
KALSHI_PRIVATE_KEY_PATH=/path/to/kalshi.pem
POLY_PRIVATE_KEY=0xYourPrivateKey
POLY_FUNDER=0xYourAddress
DRY_RUN=1
RUST_LOG=info
```

### Live trading with stricter limits

```bash
KALSHI_API_KEY_ID=your_key_id
KALSHI_PRIVATE_KEY_PATH=/path/to/kalshi.pem
POLY_PRIVATE_KEY=0xYourPrivateKey
POLY_FUNDER=0xYourAddress
DRY_RUN=0
RUST_LOG=info
CB_MAX_POSITION_PER_MARKET=50
CB_MAX_TOTAL_POSITION=200
CB_MAX_DAILY_LOSS=3000
```

### Debugging

```bash
# ... required vars ...
DRY_RUN=1
RUST_LOG=debug
PRICE_LOGGING=1
```

---

## Summary

- **Required:** Kalshi API key ID + PEM path, Polymarket private key + funder address.
- **Safety:** Keep `DRY_RUN=1` until you are ready for live trading; use the circuit breaker to cap size and loss.
- **Discovery:** Use `FORCE_DISCOVERY=1` when you want to refresh market mappings.

For how the bot uses these settings when trading, see **[03 - Trading Strategy](03-trading-strategy.md)**.
