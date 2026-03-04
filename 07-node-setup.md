# Chapter 7: Node Setup

## Running Bitcoin Knots on Windows

Running your own node is how you go from "I read about it" to "I verified it myself." This guide covers setting up Bitcoin Knots with an existing blockchain on an external drive.

## Prerequisites

- **Bitcoin Knots** installed (includes `bitcoin-qt.exe` GUI and `bitcoin-cli.exe`)
- **Blockchain data** on an external/secondary drive (or willingness to download ~700+ GB)
- **~300 GB free space** on the data drive (for catch-up sync + transaction index)
- Windows 10/11

## Step 1: Create bitcoin.conf

Bitcoin looks for its config file at:
```
%APPDATA%\Bitcoin\bitcoin.conf
```

Create this file with your settings:

```ini
# Point to your blockchain data directory
datadir=E:/

# Enable RPC server (needed for bitcoin-cli)
server=1

# Build transaction index (enables getrawtransaction lookups)
txindex=1
```

### What each setting does:

| Setting | Purpose |
|---------|---------|
| `datadir=E:/` | Store blockchain on external drive instead of C: |
| `server=1` | Enable the JSON-RPC server so bitcoin-cli works |
| `txindex=1` | Index ALL transactions (not just wallet-related ones). Required for `getrawtransaction <txid> 2` on arbitrary transactions |

### Optional settings:

```ini
# Limit memory usage (default is 450 MB for dbcache)
dbcache=1024

# Limit upload bandwidth (default unlimited)
maxuploadtarget=5000

# Prune old blocks (incompatible with txindex=1)
# prune=550000

# Listen for incoming connections
listen=1

# Set RPC credentials (auto-generated if not set)
# rpcuser=yourusername
# rpcpassword=yourpassword
```

**Note:** `txindex=1` and `prune` are mutually exclusive. We want txindex for exploring transactions, so no pruning.

## Step 2: Start Bitcoin Knots

Launch `bitcoin-qt.exe` (the GUI). It will:

1. Read `bitcoin.conf` from `%APPDATA%\Bitcoin\`
2. Find existing blockchain data on `E:/`
3. Start syncing from where the data left off

If your blockchain data is from September 2025, it needs to catch up ~6 months of blocks. This takes hours to days depending on your hardware and internet speed.

### First-time with txindex

If you're enabling `txindex=1` for the first time on existing data, Bitcoin Knots will need to **build the transaction index from scratch**. This scans the entire blockchain and can take several hours. You'll see progress in the GUI's debug console or logs.

## Step 3: Verify with bitcoin-cli

While Bitcoin Knots is running, open a terminal:

```bash
# Check node status
"C:/Program Files/Bitcoin/daemon/bitcoin-cli.exe" -getinfo

# Expected output (example):
# Chain: main
# Blocks: 890123
# Headers: 890456
# Verification progress: 0.9999
# Difficulty: 123456789.12
```

### Common commands for exploration:

```bash
CLI="C:/Program Files/Bitcoin/daemon/bitcoin-cli.exe"

# Get blockchain info
"$CLI" getblockchaininfo

# Get a specific block by height
"$CLI" getblockhash 0
"$CLI" getblock <hash> 2

# Decode a transaction (requires txindex=1)
"$CLI" getrawtransaction <txid> 2

# Get network info
"$CLI" getnetworkinfo

# Get mempool stats
"$CLI" getmempoolinfo

# Get peer info
"$CLI" getpeerinfo
```

## Step 4: Create a CLI Shortcut

Typing the full path every time is tedious. Create a helper:

### Option A: Shell alias (Git Bash / WSL)

Add to your `~/.bashrc` or `~/.bash_profile`:
```bash
alias btc='"C:/Program Files/Bitcoin/daemon/bitcoin-cli.exe"'
```

Then: `btc -getinfo`

### Option B: Batch file

Create `btc.bat` somewhere in your PATH:
```batch
@echo off
"C:\Program Files\Bitcoin\daemon\bitcoin-cli.exe" %*
```

Then: `btc -getinfo`

## Troubleshooting

### "Cannot connect to Bitcoin server"
- Make sure `bitcoin-qt.exe` (or `bitcoind.exe`) is running
- Check that `server=1` is in bitcoin.conf
- Check the debug log: `E:/debug.log`

### "Transaction not found" for getrawtransaction
- `txindex=1` must be set AND the index must be fully built
- Check progress: `bitcoin-cli getindexinfo`
- If you added txindex after initial sync, you need to wait for reindexing

### Slow initial sync
- Increase `dbcache` (e.g., `dbcache=4096` if you have RAM to spare)
- Make sure your data drive isn't USB 2.0 (USB 3.0+ or internal SSD recommended)
- Close other disk-heavy applications during sync

### Data directory structure

After syncing, your data directory (`E:/` in our case) contains:

```
E:/
├── blocks/           # Raw block data (~650+ GB)
│   ├── blk00000.dat
│   ├── blk00001.dat
│   └── ...
├── chainstate/       # Current UTXO set (LevelDB, ~12 GB)
├── indexes/
│   └── txindex/      # Transaction index (if txindex=1, ~35 GB)
├── peers.dat         # Known peer addresses
├── banlist.json      # Banned peers
├── debug.log         # Log file
└── wallet.dat        # Wallet file (if created)
```

## Bitcoin Knots vs. Bitcoin Core

Bitcoin Knots is a fork of Bitcoin Core maintained by Luke Dashjr. Key differences:

| Feature | Bitcoin Core | Bitcoin Knots |
|---------|-------------|---------------|
| Maintainer | Bitcoin Core team | Luke Dashjr |
| Policy | Permissive relay | Stricter relay (filters inscriptions) |
| BIP-110 | Not implemented | Enforced |
| OP_RETURN | Unlimited (v30+) | 83-byte limit |
| Codebase | Reference | Core + patches |

Both follow the same consensus rules — they agree on which blocks are valid. The difference is in **relay policy**: what transactions they'll propagate to other nodes before they're mined.

## Verify Everything From This Guide

Now that you have a node running with `txindex=1`, you can verify every claim in this guide:

```bash
# Chapter 1: UTXO model — check UTXO set stats
btc gettxoutsetinfo

# Chapter 2: Script — decode any transaction's scripts
btc getrawtransaction <txid> 2 | jq '.vin[0].scriptSig, .vout[0].scriptPubKey'

# Chapter 3: SegWit — compare weight vs size
btc getrawtransaction <segwit-txid> 2 | jq '{size, weight, vsize}'

# Chapter 4: Taproot — look at witness data
btc getrawtransaction <taproot-txid> 2 | jq '.vin[0].txinwitness'

# Chapter 5: Ordinals — decode an inscription transaction
btc getrawtransaction <inscription-txid> 2 | jq '.vin[0].txinwitness'
# Look for the OP_FALSE OP_IF envelope in the witness hex
```

## Key Takeaways

1. `bitcoin.conf` needs `server=1` for CLI access and `txindex=1` for arbitrary transaction lookups
2. Point `datadir` to your existing blockchain data to avoid re-downloading
3. Building txindex from scratch takes hours but enables full exploration
4. Create a shell alias or batch file for convenient CLI access
5. Bitcoin Knots and Core share consensus rules but differ in relay policy

---

**Back to:** [Table of Contents](README.md)
