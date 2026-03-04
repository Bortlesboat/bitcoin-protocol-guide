# Large Image Inscription

**When the witness discount meets digital art: a 395 KB image that nearly fills a block.**

## What This Demonstrates

This example illustrates the extreme case of witness data usage. A single inscription can occupy nearly an entire block's worth of witness space — the scenario that drives the [BIP-110 debate](../06-bip110-debate.md).

## The Mechanics

### How large inscriptions fit in blocks

The block weight limit is 4,000,000 WU. For a large image inscription:

```
Block weight budget:        4,000,000 WU
─────────────────────────────────────────
Typical block overhead:        ~10,000 WU  (coinbase tx, headers)
Image in witness:             ~395,000 WU  (395 KB × 1 WU/byte)
Transaction non-witness:         ~500 WU  (inputs, outputs × 4)
                              ─────────
Total for inscription:        ~405,500 WU
Remaining for other txs:    ~3,594,500 WU
```

A 395 KB image consumes roughly **10%** of a block's weight capacity. At the witness discount rate, this costs the same as ~100 KB of non-witness data.

### The inscription structure

For a large image, the data is split across multiple pushes (max 520 bytes each):

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_1
  OP_PUSH "image/png"
  OP_0
  OP_PUSH <bytes 0-519>       // chunk 1
  OP_PUSH <bytes 520-1039>    // chunk 2
  OP_PUSH <bytes 1040-1559>   // chunk 3
  ... (760+ pushes for a 395 KB image)
OP_ENDIF
<pubkey>
OP_CHECKSIG
```

### Cost calculation

At 20 sat/vB (moderate fee environment):

```
Inscription weight:    ~395,500 WU
Virtual size:          ~395,500 / 4 = ~98,875 vB
Fee:                   98,875 × 20 = 1,977,500 sats
                       ≈ 0.0198 BTC
                       ≈ $1,980 at $100K/BTC
```

Without the SegWit discount, the same image would cost **4× more** (~$7,920) because it would count as 395,500 × 4 = 1,582,000 WU.

## Why This Is Controversial

### The "abuse" argument

Critics call large inscriptions "witness abuse" because:

1. **The witness discount was meant for signatures** — A typical Schnorr signature is 64 bytes. A 395 KB image is ~6,172 signatures worth of data
2. **It displaces financial transactions** — Block space used for images can't be used for payments
3. **The data persists forever** — Every full node must store this data permanently (unless pruning)

### The "legitimate use" counter

Defenders argue:

1. **The fee was paid at market rate** — No discount beyond what the protocol provides to all witness data
2. **Witness data CAN be pruned** — Unlike UTXO data, old witness data is only needed for initial validation
3. **Block fullness varies** — Most blocks aren't consistently full; inscriptions fill otherwise-empty space
4. **The protocol allows it** — If the rules permit it, restricting it requires a consensus change (soft fork)

## The Numbers in Practice

During peak inscription activity (2023-2024):

| Metric | Before Ordinals | Peak Ordinals | Normal 2025 |
|--------|----------------|---------------|-------------|
| Avg block size | ~1.3 MB | ~2.5 MB | ~1.8 MB |
| Avg block weight | ~3.0M WU | ~3.9M WU | ~3.5M WU |
| Median fee rate | 5-15 sat/vB | 50-200+ sat/vB | 10-30 sat/vB |
| Daily inscriptions | 0 | 200,000+ | ~20,000 |

## Verify It Yourself

```bash
# Find a block with a large inscription (look for blocks near 4M weight)
bitcoin-cli getblock <blockhash> 1 | jq '{weight, size, nTx}'

# Decode a large transaction and check its witness size
bitcoin-cli getrawtransaction <large-inscription-txid> 2 | jq '{size, vsize, weight}'

# Compare: weight should be much less than size * 4
# (because witness data dominates and gets the discount)
```

You can find large inscriptions by browsing [ordinals.com](https://ordinals.com) or checking block explorers for blocks with unusually high weight-to-transaction-count ratios.

## Protocol Concepts Illustrated

- [x] **Witness discount economics** — 1 WU/byte vs 4 WU/byte makes a 4× cost difference
- [x] **Block weight limits** — 4M WU constrains total block content
- [x] **Data chunking** — 520-byte push limit requires splitting large content
- [x] **The BIP-110 trigger** — large inscriptions are the primary target of the proposal
- [x] **Fee market dynamics** — inscriptions competing with payments for block space

---

**Back to:** [Examples Index](../README.md#bonus) | [Chapter 6: BIP-110 Debate](../06-bip110-debate.md)
