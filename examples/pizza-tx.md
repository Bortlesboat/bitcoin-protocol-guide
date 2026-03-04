# The Pizza Transaction

**The most expensive pizza in history: 10,000 BTC for two large pizzas.**

## The Story

On May 22, 2010, Laszlo Hanyecz paid 10,000 BTC to Jeremy Sturdivant (jercos) for two Papa John's pizzas. This was the first known real-world transaction using Bitcoin as payment. May 22 is now celebrated as "Bitcoin Pizza Day."

At the time, 10,000 BTC was worth roughly $41. At $100,000/BTC, it would be worth $1 billion.

## The Transaction

**txid:** `a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d`

[View on mempool.space](https://mempool.space/tx/a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d)

### Summary

| Field | Value |
|-------|-------|
| Block height | 57,043 |
| Date | May 22, 2010 |
| Version | 1 |
| Size | 23,620 bytes |
| Weight | 94,480 WU |
| Inputs | 131 |
| Outputs | 1 |
| Total input | ~10,000.99 BTC |
| Total output | 10,000 BTC |
| Fee | ~0.99 BTC |

### What's Interesting About This Transaction

**1. 131 inputs, 1 output — UTXO consolidation**

Laszlo was an early miner. He had accumulated hundreds of small UTXOs from mining (many worth just 0.01 BTC — mining dust from early blocks). This transaction consolidates all 131 UTXOs into a single 10,000 BTC output.

This is a perfect illustration of the [UTXO model](../01-utxo-model.md): you can't partially spend a UTXO, so to make a 10,000 BTC payment you need to gather enough UTXOs to cover it.

**2. No change output**

The inputs totaled ~10,000.99 BTC and the single output was exactly 10,000 BTC. The ~0.99 BTC difference went entirely to the miner as a fee. This was before fee-awareness was built into wallets.

A modern wallet would create a change output to send the excess back to the sender. See [Chapter 1](../01-utxo-model.md) on change outputs.

**3. P2PKH script type**

This transaction predates SegWit and Taproot by many years. All scripts are P2PKH (Pay-to-Public-Key-Hash) — the original Bitcoin script type.

Output scriptPubKey:
```
OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
```

Recipient address: `17SkEw2md5avVNyYgj6RiXuQKNwkXaxFyQ`

**4. All inputs from one address**

All 131 inputs came from: `1XPTgDRhN8RFnzniWCddobD9iKZatrvH4`

This was Laszlo's mining address. Input values ranged from 0.01 BTC (mining dust) up to 3,753.88 BTC. The large inputs were likely from successful block mining (the block reward was 50 BTC in 2010).

**5. The fee was enormous (percentage-wise)**

~0.99 BTC as a fee — about 0.01% of the total transaction value. In 2010, fees were essentially irrelevant (blocks were nearly empty), so nobody optimized for them. The wallet simply didn't create a change output.

## Verify It Yourself

```bash
# Decode the full transaction
bitcoin-cli getrawtransaction a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d 2

# Check the output (single P2PKH output, 10,000 BTC)
bitcoin-cli getrawtransaction a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d 2 | jq '.vout'

# Count the inputs
bitcoin-cli getrawtransaction a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d 2 | jq '.vin | length'
# Returns: 131

# Check the block it was in
bitcoin-cli getrawtransaction a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d 2 | jq '.blockhash'
```

## Protocol Concepts Illustrated

- [x] **UTXO consolidation** — many inputs → single output
- [x] **No change output** — excess becomes miner fee
- [x] **P2PKH script** — the original lock/unlock mechanism
- [x] **Implicit fees** — no fee field, just inputs minus outputs
- [x] **Early mining** — many small UTXOs from solo mining

---

**Back to:** [Examples Index](../README.md#bonus) | [Chapter 1: UTXO Model](../01-utxo-model.md)
