# Mempool Analyzer

Snapshot the current mempool: transaction count, total fees, fee rate distribution by bucket, congestion level, and a plain-English summary.

## What It Shows

- Total unconfirmed transactions and their combined size
- Fee rate distribution across buckets (1-2, 2-5, 5-10, 10-20, 20-50, 50-100, 100+ sat/vB)
- Minimum fee rate to enter the next block
- Congestion assessment (low/moderate/high/extreme)
- Estimated blocks needed to clear the mempool

## Usage

```bash
python mempool_analyzer.py
```

## How It Works

1. Calls `getrawmempool(true)` to get all unconfirmed transactions with metadata
2. Calls `getmempoolinfo` for aggregate stats
3. Buckets transactions by fee rate (fee / vsize)
4. Estimates next-block entry fee from the top 4M weight units of pending transactions
5. Generates a natural-language summary

## Protocol Context

The mempool is Bitcoin's fee market — a real-time auction for block space. Every full node maintains its own mempool (there is no single "the mempool"), though they converge because transactions propagate across the P2P network.

Fee rates are measured in **satoshis per virtual byte (sat/vB)**. Virtual bytes account for the SegWit weight discount: witness data costs 1 weight unit per byte vs 4 for non-witness data, and `vsize = weight / 4`. This is why SegWit transactions pay lower fees for the same economic activity — see [Chapter 3](../03-segwit.md).

The mempool is the core input to the **block template** that miners build. Rational miners select the highest-feerate transactions that fit within the 4M weight unit limit. The cluster mempool architecture (Bitcoin Core 28+, see [Chapter 12](../12-future-bitcoin.md)) improves this selection by grouping related transactions.
