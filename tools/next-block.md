# Next Block Predictor

See exactly what a miner would mine right now: the block template with revenue breakdown, weight utilization, fee rate distribution, and the highest-fee transactions.

## Usage

```bash
python next_block.py
```

## What It Shows

- Next block height
- Total weight utilization (% of 4M WU limit)
- Miner revenue: subsidy (3.125 BTC) + total fees
- Transaction count in the template
- Fee rate distribution (min, median, max, percentiles)
- Top 5 highest-fee transactions with their txids and fee rates

## How It Works

1. Calls `getblocktemplate({"rules": ["segwit"]})` to get the current candidate block
2. Parses all transactions in the template
3. Computes weight utilization, fee totals, and rate distribution
4. Sorts by fee rate to identify the highest-paying transactions

## Protocol Context

The block template is what mining pools distribute to their hashers via the Stratum protocol (see [Chapter 10](../10-mining-hardware.md)). In the default configuration, the pool's node constructs the template — this is the **block template centralization problem**.

Stratum V2's Job Declaration protocol allows individual miners to construct their own templates, partially decentralizing transaction selection. This tool shows you what YOUR node would include, which may differ from what a pool's node selects.

The block template is also where the fee market meets reality. The `getblocktemplate` RPC returns transactions sorted by effective feerate (accounting for parent-child relationships via ancestor feerate in pre-cluster-mempool versions, and chunk feerate in cluster mempool). This is the optimal ordering for miner revenue maximization — see [Chapter 12](../12-future-bitcoin.md) for how cluster mempool improves this.
