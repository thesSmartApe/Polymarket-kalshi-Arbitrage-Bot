# 01 — Getting Started

This guide walks you through installing and running **Jerrrix Poly-Kalshi Arbitrage Bot** from scratch. We recommend running in **dry run** (paper trading) first.

---

## What You Need

- **Rust 1.75+** (we’ll install it below)
- **Kalshi account** and API key + private key (PEM)
- **Polymarket wallet** (Ethereum/Polygon) with USDC for live trading
- **dotenvx** (optional but recommended for loading `.env`)

---

## Step 1: Install Rust

On macOS/Linux:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

On Windows, use the installer from [rustup.rs](https://rustup.rs). After installation, open a new terminal and check:

```bash
rustc --version
cargo --version
```

You should see version 1.75 or newer.

---

## Step 2: Clone the Repository

```bash
git clone https://github.com/thesSmartApe/Polymarket-kalshi-Arbitrage-Bot.git
cd Polymarket-kalshi-Arbitrage-Bot
```

---

## Step 3: Build the Bot

```bash
cargo build --release
```

The first build may take a few minutes. When it finishes, the binary is at `target/release/jerrrix-arb-bot` (or `jerrrix-arb-bot.exe` on Windows).

---

## Step 4: Set Up Credentials (`.env`)

Create a file named `.env` in the project root (same folder as `Cargo.toml`).

### Minimum for dry run (paper trading)

The bot still needs valid Kalshi and Polymarket credentials to connect; it just won’t place real orders when `DRY_RUN=1`.

```bash
# Kalshi (required for discovery and orderbook)
KALSHI_API_KEY_ID=your_kalshi_api_key_id
KALSHI_PRIVATE_KEY_PATH=/path/to/your/kalshi_private_key.pem

# Polymarket (required for discovery and orderbook)
POLY_PRIVATE_KEY=0xYourEthereumPrivateKeyHex
POLY_FUNDER=0xYourWalletAddress

# Run in paper trading mode (no real orders)
DRY_RUN=1
RUST_LOG=info
```

- Replace `your_kalshi_api_key_id` with your Kalshi API Key ID.
- Replace `/path/to/your/kalshi_private_key.pem` with the real path to your Kalshi PEM file.
- Replace `0xYourEthereumPrivateKeyHex` and `0xYourWalletAddress` with your Polymarket wallet private key and address (keep the `0x` prefix).

See **[02 - Configuration](02-configuration.md)** for how to get these and for all other options.

---

## Step 5: Run the Bot

### Option A: With dotenvx (recommended)

If you have [dotenvx](https://github.com/dotenvx/dotenvx) installed:

```bash
dotenvx run -- cargo run --release
```

This loads variables from `.env` automatically.

### Option B: Without dotenvx

On Linux/macOS you can source `.env` manually (never commit `.env`):

```bash
export $(grep -v '^#' .env | xargs)
cargo run --release
```

On Windows (PowerShell), set variables from `.env` manually or use a tool that loads `.env` before running `cargo run --release`.

---

## What You Should See

When the bot starts successfully you should see logs similar to:

- `Jerrrix Arb Bot v2.0`
- `[KALSHI] API key loaded`
- `[POLYMARKET] Client ready for 0x...`
- `Discovery complete: Market pairs found: N`
- `Heartbeat | Markets: ...`

As long as `DRY_RUN=1`, **no real orders are sent**. The bot will discover markets, stream orderbooks, and log when it *would* trade, so you can safely get used to the behavior.

---

## Next Steps

- **[02 - Configuration](02-configuration.md)** — Full list of environment variables and what they do.
- **[03 - Trading Strategy](03-trading-strategy.md)** — How the bot finds and executes arbitrage and how risk is controlled.

Only switch to live trading (`DRY_RUN=0`) after you understand the config and strategy and have tested in dry run.
