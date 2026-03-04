# Bitcoin CLI Command Reference

Quick reference for `bitcoin-cli` commands useful for exploring the protocol. All commands assume a running node with `server=1` and `txindex=1`.

## Setup

```bash
# Set up an alias (add to ~/.bashrc)
alias btc='"C:/Program Files/Bitcoin/daemon/bitcoin-cli.exe"'
```

## Node Status

```bash
# Quick status overview
btc -getinfo

# Detailed blockchain info
btc getblockchaininfo

# Network connections and version
btc getnetworkinfo

# Connected peers
btc getpeerinfo
```

## Blocks

```bash
# Get block hash by height
btc getblockhash 0                    # Genesis block
btc getblockhash 481824               # First SegWit block
btc getblockhash 709632               # First Taproot block

# Get block details (verbosity: 0=hex, 1=JSON, 2=JSON with tx details)
btc getblock <blockhash> 1            # Block header + txid list
btc getblock <blockhash> 2            # Full block with decoded transactions

# Get current block count
btc getblockcount

# Get block header only
btc getblockheader <blockhash>
```

## Transactions

```bash
# Decode a transaction (requires txindex=1 for non-wallet txs)
btc getrawtransaction <txid> 2

# Get raw hex (for manual decoding)
btc getrawtransaction <txid> 0

# Decode raw hex without looking up the blockchain
btc decoderawtransaction <hex>

# Decode a script
btc decodescript <hex>
```

## UTXO Set

```bash
# UTXO set statistics (takes a moment)
btc gettxoutsetinfo

# Check if a specific output is unspent
btc gettxout <txid> <vout>
# Returns null if spent, JSON if unspent
```

## Mempool

```bash
# Mempool overview
btc getmempoolinfo

# List all transactions in mempool
btc getrawmempool

# Detailed mempool entries (with fees, size, time)
btc getrawmempool true

# Get specific mempool entry
btc getmempoolentry <txid>

# Fee estimation (target: blocks until confirmation)
btc estimatesmartfee 1     # Next block
btc estimatesmartfee 6     # Within ~1 hour
btc estimatesmartfee 144   # Within ~1 day
```

## Mining Info

```bash
# Current mining/difficulty info
btc getmininginfo

# Current network difficulty
btc getdifficulty

# Get block template (what a miner would build)
btc getblocktemplate '{"rules": ["segwit"]}'
```

## Network

```bash
# Peer info
btc getpeerinfo

# Add a peer manually
btc addnode <ip:port> add

# Get network totals (bytes sent/received)
btc getnettotals

# Ban a peer
btc setban <ip> add
```

## Index Status

```bash
# Check indexing progress (txindex, blockfilterindex, etc.)
btc getindexinfo
```

## Historic Transactions to Explore

```bash
# Genesis block coinbase (the first ever Bitcoin transaction)
btc getblockhash 0
# Then: btc getblock <hash> 2 | look at tx[0]

# The Pizza Transaction (10,000 BTC for two pizzas)
btc getrawtransaction a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d 2

# First SegWit transaction (block 481824)
btc getblockhash 481824
# Then explore transactions in that block

# First Taproot spend (block 709632)
btc getblockhash 709632

# Hal Finney's transaction (first non-Satoshi transaction, block 170)
btc getblockhash 170
```

## Useful Patterns

```bash
# Get the coinbase text from a block (miners often embed messages)
btc getblockhash <height> | xargs btc getblock - 2 | jq -r '.tx[0].vin[0].coinbase' | xxd -r -p

# Count transactions in a block
btc getblock <hash> 1 | jq '.nTx'

# Get all output values from a transaction
btc getrawtransaction <txid> 2 | jq '[.vout[].value] | add'

# Check weight vs size of a transaction
btc getrawtransaction <txid> 2 | jq '{size, vsize, weight}'

# See script types in a transaction's outputs
btc getrawtransaction <txid> 2 | jq '.vout[].scriptPubKey.type'
```

## Tips

- `jq` is your best friend for parsing JSON output
- Pipe to `| less` for long outputs
- The `2` in `getrawtransaction <txid> 2` means "verbose JSON" (0 = raw hex, 1 = JSON without prevout details)
- `getblock <hash> 2` with verbosity 2 includes full transaction details (large output for big blocks)
- Use mempool.space as a visual complement to CLI exploration
