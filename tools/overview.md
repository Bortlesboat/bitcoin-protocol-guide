# Bitcoin Protocol Tools

A collection of Python tools for querying and analyzing a local Bitcoin node. These tools complement the guide — every concept discussed in the chapters can be verified using these scripts against your own node.

## Install

```bash
pip install bitcoinlib-rpc
```

**Requirements:** A fully synced Bitcoin Core or Bitcoin Knots node with `server=1` and `txindex=1`, Python 3.10+.

**Source:** [github.com/Bortlesboat/bitcoinlib-rpc](https://github.com/Bortlesboat/bitcoinlib-rpc)

## The Toolkit

| Tool | What It Does | Related Chapter |
|------|-------------|-----------------|
| [Mempool Analyzer](mempool.md) | Real-time mempool snapshot with fee buckets and congestion analysis | [Ch 10: Mining](../10-mining-hardware.md) |
| [Transaction Decoder](tx-decoder.md) | Decode any transaction: script types, fee rate, inscription detection | [Ch 1: UTXOs](../01-utxo-model.md), [Ch 2: Script](../02-script-system.md) |
| [Block Analyzer](block-analyzer.md) | Analyze any block: pool ID, SegWit/Taproot adoption, fee distribution | [Ch 3: SegWit](../03-segwit.md), [Ch 10: Mining](../10-mining-hardware.md) |
| [Fee Tracker](fee-tracker.md) | Fee estimation for multiple confirmation targets, CSV logging | [Ch 10: Mining](../10-mining-hardware.md) |
| [Next Block Predictor](next-block.md) | What a miner would mine right now: revenue, fee composition | [Ch 10: Mining](../10-mining-hardware.md) |

## Quick Start

```bash
pip install bitcoinlib-rpc

# CLI tools (auto-detect node via cookie authentication)
bitcoin-status
bitcoin-mempool
bitcoin-block 939290
bitcoin-tx a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d
bitcoin-fees --once
bitcoin-nextblock
```

### Python API

```python
from bitcoinlib_rpc import BitcoinRPC
from bitcoinlib_rpc.mempool import analyze_mempool
from bitcoinlib_rpc.blocks import analyze_block

rpc = BitcoinRPC()  # auto-detects cookie auth

mempool = analyze_mempool(rpc)
print(f"Congestion: {mempool.congestion}")
print(f"Next block min fee: {mempool.next_block_min_fee:.1f} sat/vB")

block = analyze_block(rpc, 939290)
print(f"Mined by: {block.pool_name}")
print(f"Taproot adoption: {block.taproot_pct:.1f}%")
```

## Architecture

All tools share a common RPC client (`rpc.py`) that handles authentication via Bitcoin's cookie file. The tools are read-only — they query the node but never send transactions or modify state.

```
┌─────────────────────┐
│   Your Bitcoin Node  │
│  (Core or Knots)     │
│  server=1, txindex=1 │
└──────────┬──────────┘
           │ JSON-RPC (port 8332)
           │ Cookie authentication
           ▼
┌─────────────────────┐
│      rpc.py          │ ← Shared RPC client
└──────────┬──────────┘
     ┌─────┼─────┬──────────┬──────────┐
     ▼     ▼     ▼          ▼          ▼
  mempool  tx   block    fee_tracker  next_block
```

## Ecosystem

These tools are part of a larger Bitcoin product suite:

- **[bitcoinlib-rpc](https://github.com/Bortlesboat/bitcoinlib-rpc)** — The pip-installable Python library powering these tools (available now)
- **bitcoin-mcp** — An MCP server so AI agents can query your node (coming soon)
- **Bitcoin Fee Observatory** — A Streamlit dashboard for fee market analytics (coming soon)
