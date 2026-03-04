# Block Analyzer

Analyze any block by height or hash: mining pool identification, SegWit/Taproot adoption percentages, fee distribution, weight utilization, and subsidy vs fee breakdown.

## Usage

```bash
# By height
python block_analyzer.py 939000

# By hash
python block_analyzer.py 0000000000000000000123abc...
```

## What It Shows

- Block metadata: height, hash, timestamp, size, weight, transaction count
- Mining pool identification (parsed from coinbase text)
- Weight utilization (% of 4M WU limit)
- Subsidy vs fees breakdown (the "security budget" split)
- Transaction type distribution: % SegWit, % Taproot
- Fee rate distribution across transactions in the block
- Top 5 highest-fee transactions

## How It Works

1. Resolves height to hash via `getblockhash` if needed
2. Calls `getblock(hash, 2)` for the full block with decoded transactions
3. Parses the coinbase transaction's scriptSig for pool identification strings (matches against known pools: Foundry, AntPool, ViaBTC, F2Pool, etc.)
4. Iterates all transactions, classifying inputs/outputs by script type
5. Computes fee rate for each transaction and builds the distribution

## Protocol Context

This tool brings [Chapter 10 (Mining)](../10-mining-hardware.md) to life. The weight utilization shows how close miners get to the 4M weight unit limit from [Chapter 3 (SegWit)](../03-segwit.md). The subsidy-vs-fees ratio is the "security budget" question discussed in [Chapter 12](../12-future-bitcoin.md) — as halvings continue, transaction fees must increasingly fund network security.

Pool identification works because miners include their name in the coinbase transaction's `scriptSig` field. This is a convention, not a protocol requirement — miners can put anything in the coinbase (Satoshi famously put a newspaper headline in the genesis block, see [Chapter 8](../08-satoshi-timeline.md)).
