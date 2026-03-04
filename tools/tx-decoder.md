# Transaction Decoder

Decode any Bitcoin transaction into a human-readable analysis: inputs, outputs, script types, fee rate, SegWit savings, and inscription detection.

## Usage

```bash
python tx_decoder.py <txid>

# Example: decode the genesis coinbase
python tx_decoder.py 4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b
```

## What It Shows

- Transaction version, size, vsize, weight
- Each input: previous txid, vout, script type
- Each output: value (BTC + sats), script type, address
- Fee in satoshis and fee rate in sat/vB
- SegWit discount percentage (how much the sender saved)
- Whether the transaction contains an Ordinals inscription

## How It Works

1. Calls `getrawtransaction(txid, 2)` to get the fully decoded transaction
2. Parses each input's `scriptSig` and `witness` to identify the script type
3. Parses each output's `scriptPubKey` to identify the address type
4. Computes fee = sum(input values) - sum(output values)
5. Scans witness data for the Ordinals inscription marker (`ord`)

**Requires `txindex=1`** in your bitcoin.conf to decode arbitrary transactions (not just those in your wallet).

## Protocol Context

This tool demonstrates the core concepts from [Chapter 1 (UTXOs)](../01-utxo-model.md) and [Chapter 2 (Script)](../02-script-system.md) in action. Every transaction consumes UTXOs (inputs) and creates new ones (outputs). The script types visible in the output show the evolution of Bitcoin's address formats:

- `pubkeyhash` = P2PKH (1... addresses, 2009)
- `scripthash` = P2SH (3... addresses, 2012)
- `witness_v0_keyhash` = P2WPKH (bc1q..., 2017 SegWit)
- `witness_v1_taproot` = P2TR (bc1p..., 2021 Taproot)

The inscription detection shows how Ordinals embed data in the witness field using the `OP_FALSE OP_IF` envelope — see [Chapter 5](../05-ordinals-inscriptions.md).
