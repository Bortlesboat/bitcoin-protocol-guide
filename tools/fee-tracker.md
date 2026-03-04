# Fee Tracker

Monitor Bitcoin fee estimates across multiple confirmation targets. Can run as a one-shot snapshot or a continuous polling loop that logs to CSV.

## Usage

```bash
# One-shot: current fee estimates
python fee_tracker.py --once

# Continuous: poll every 60 seconds, log to CSV
python fee_tracker.py
```

## What It Shows

| Confirmation Target | Meaning |
|-------------------|---------|
| 1 block (~10 min) | Next-block priority |
| 3 blocks (~30 min) | High priority |
| 6 blocks (~1 hour) | Medium priority |
| 25 blocks (~4 hours) | Low priority |
| 144 blocks (~1 day) | Economy / no rush |

For each target, displays the estimated fee rate in sat/vB with a plain-English urgency label.

## How It Works

1. Calls `estimatesmartfee(target, "ECONOMICAL")` for each confirmation target
2. Converts the result from BTC/kvB to sat/vB (multiply by 100,000)
3. Calls `getmempoolinfo` for current mempool size and minimum relay fee
4. In continuous mode, appends a timestamped row to `fee_history.csv`

## Protocol Context

Bitcoin's fee estimation is one of the hardest problems in the protocol. `estimatesmartfee` uses historical data about how long transactions at various fee rates took to confirm. It does NOT look at the current mempool — it's a backward-looking statistical model.

This is why mempool-based fee estimation (looking at what's currently pending) can sometimes be more accurate for near-term predictions. The [Mempool Analyzer](mempool.md) provides that complementary view.

The fee market is the long-term security model for Bitcoin. As the block subsidy halves every ~4 years (see [Chapter 10](../10-mining-hardware.md)), transaction fees must increasingly compensate miners for securing the network. Understanding fee dynamics is understanding Bitcoin's economic sustainability.
