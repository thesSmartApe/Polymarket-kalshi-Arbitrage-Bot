# Polymarket-Kalshi Arbitrage Bot

A arbitrage system for cross-platform prediction market trading between Kalshi and Polymarket.

## Quick Start

### 1. Install Dependencies

```bash
# Rust 1.75+
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Build
cd e_poly_kalshi_arb
cargo build --release
```

### 2. Set Up Credentials

Create a `.env` file:

```bash
# === KALSHI CREDENTIALS ===
KALSHI_API_KEY_ID=your_kalshi_api_key_id
KALSHI_PRIVATE_KEY_PATH=/path/to/kalshi_private_key.pem

# === POLYMARKET CREDENTIALS ===
POLY_PRIVATE_KEY=0xYOUR_WALLET_PRIVATE_KEY
POLY_FUNDER=0xYOUR_WALLET_ADDRESS

# === BOT CONFIGURATION ===
DRY_RUN=1
RUST_LOG=info
```

### 3. Run

```bash
# Dry run (paper trading)
dotenvx run -- cargo run --release

# Live execution
DRY_RUN=0 dotenvx run -- cargo run --release
```

---

## Environment Variables

### Required

| Variable                  | Description                                                 |
| ------------------------- | ----------------------------------------------------------- |
| `KALSHI_API_KEY_ID`       | Your Kalshi API key ID                                      |
| `KALSHI_PRIVATE_KEY_PATH` | Path to RSA private key (PEM format) for Kalshi API signing |
| `POLY_PRIVATE_KEY`        | Ethereum private key (with 0x prefix) for Polymarket wallet |
| `POLY_FUNDER`             | Your Polymarket wallet address (with 0x prefix)             |

### Bot Configuration

| Variable          | Default | Description                                           |
| ----------------- | ------- | ----------------------------------------------------- |
| `DRY_RUN`         | `1`     | `1` = paper trading (no orders), `0` = live execution |
| `RUST_LOG`        | `info`  | Log level: `error`, `warn`, `info`, `debug`, `trace`  |
| `FORCE_DISCOVERY` | `0`     | `1` = re-fetch market mappings (ignore cache)         |
| `PRICE_LOGGING`   | `0`     | `1` = verbose price update logging                    |

### Test Mode

| Variable        | Default              | Description                                                                                    |
| --------------- | -------------------- | ---------------------------------------------------------------------------------------------- |
| `TEST_ARB`      | `0`                  | `1` = inject synthetic arb opportunity for testing                                             |
| `TEST_ARB_TYPE` | `poly_yes_kalshi_no` | Arb type: `poly_yes_kalshi_no`, `kalshi_yes_poly_no`, `poly_same_market`, `kalshi_same_market` |

### Circuit Breaker

| Variable                     | Default | Description                                 |
| ---------------------------- | ------- | ------------------------------------------- |
| `CB_ENABLED`                 | `true`  | Enable/disable circuit breaker              |
| `CB_MAX_POSITION_PER_MARKET` | `100`   | Max contracts per market                    |
| `CB_MAX_TOTAL_POSITION`      | `500`   | Max total contracts across all markets      |
| `CB_MAX_DAILY_LOSS`          | `5000`  | Max daily loss in cents before halt         |
| `CB_MAX_CONSECUTIVE_ERRORS`  | `5`     | Consecutive errors before halt              |
| `CB_COOLDOWN_SECS`           | `60`    | Cooldown period after circuit breaker trips |

---

## Obtaining Credentials

### Kalshi

1. Log in to [Kalshi](https://kalshi.com)
2. Go to **Settings → API Keys**
3. Create a new API key with trading permissions
4. Download the private key (PEM file)
5. Note the API Key ID

### Polymarket

1. Create or import an Ethereum wallet (MetaMask, etc.)
2. Export the private key (include `0x` prefix)
3. Fund your wallet on Polygon network with USDC
4. The wallet address is your `POLY_FUNDER`

---

## Usage Examples

### Paper Trading (Development)

```bash
# Full logging, dry run
RUST_LOG=debug DRY_RUN=1 dotenvx run -- cargo run --release
```

### Test Arbitrage Execution

```bash
# Inject synthetic arb to test execution path
TEST_ARB=1 DRY_RUN=0 dotenvx run -- cargo run --release
```

### Production

```bash
# Live trading with circuit breaker
DRY_RUN=0 CB_MAX_DAILY_LOSS=10000 dotenvx run -- cargo run --release
```

### Force Market Re-Discovery

```bash
# Clear cache and re-fetch all market mappings
FORCE_DISCOVERY=1 dotenvx run -- cargo run --release
```

---

## How It Works

### Arbitrage Mechanics

In prediction markets, YES + NO = $1.00 guaranteed.

**Arbitrage exists when:**

```
Best YES ask (platform A) + Best NO ask (platform B) < $1.00
```

**Example:**

```
Kalshi YES ask:  42¢
Poly NO ask:     56¢
Total cost:      98¢
Guaranteed:     100¢
Profit:           2¢ per contract
```

### Four Arbitrage Types

| Type                 | Buy                 | Sell          |
| -------------------- | ------------------- | ------------- |
| `poly_yes_kalshi_no` | Polymarket YES      | Kalshi NO     |
| `kalshi_yes_poly_no` | Kalshi YES          | Polymarket NO |
| `poly_same_market`   | Polymarket YES + NO | (rare)        |
| `kalshi_same_market` | Kalshi YES + NO     | (rare)        |

### Fee Handling

- **Kalshi**: `ceil(0.07 × contracts × price × (1-price))` - factored into arb detection
- **Polymarket**: Zero trading fees

---

## Architecture

```
src/
├── main.rs              # Entry point, WebSocket orchestration
├── types.rs             # MarketArbState
├── execution.rs         # Concurrent leg execution, in-flight deduplication
├── position_tracker.rs  # Channel-based fill recording, P&L tracking
├── circuit_breaker.rs   # Risk limits, error tracking, auto-halt
├── discovery.rs         # Kalshi↔Polymarket market matching
├── cache.rs             # Team code mappings (EPL, NBA, etc.)
├── kalshi.rs            # Kalshi REST/WS client
├── polymarket.rs        # Polymarket WS client
├── polymarket_clob.rs   # Polymarket CLOB order execution
└── config.rs            # League configs, thresholds
```

---

## Development

### Run Tests

```bash
cargo test
```

### Enable Profiling

```bash
cargo build --release --features profiling
```

### Benchmarks

```bash
cargo bench
```

---

## Project Status

- [x] Kalshi REST/WebSocket client
- [x] Polymarket REST/WebSocket client
- [x] Lock-free orderbook cache
- [x] SIMD arb detection
- [x] Concurrent order execution
- [x] Position & P&L tracking
- [x] Circuit breaker
- [x] Market discovery & caching
- [ ] Risk limit configuration UI
- [ ] Multi-account support

# poly-kalshi-arb
