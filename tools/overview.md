# Bitcoin Protocol Tools

A collection of Python tools for querying and analyzing a local Bitcoin node. These tools complement the guide — every concept discussed in the chapters can be verified using these scripts against your own node.

## Requirements

- A fully synced Bitcoin Core or Bitcoin Knots node with `server=1` and `txindex=1`
- Python 3.10+
- `pip install requests`

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
# Clone the tools
git clone https://github.com/Bortlesboat/bitcoin-protocol-tools
cd bitcoin-protocol-tools

# Install dependencies
pip install requests

# Configure (edit config.py with your node's RPC settings)
cp config_example.py config.py

# Try it
python mempool_analyzer.py
python block_analyzer.py 939000
python tx_decoder.py <any-txid>
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

## Coming Soon

These tools are being packaged as:
- **bitcoinlib-rpc** — A pip-installable Python library with typed returns
- **bitcoin-mcp** — An MCP server so AI agents can query your node
- **Bitcoin Fee Observatory** — A Streamlit dashboard for fee market analytics
