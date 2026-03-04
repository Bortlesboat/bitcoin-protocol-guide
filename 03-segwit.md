# Chapter 3: Segregated Witness (SegWit)

## In Satoshi's Words

Satoshi never saw SegWit (he disappeared in 2011), but he anticipated the scaling challenges that drove its creation:

> "The current system where every user is a network node is not the intended configuration for large scale. That would be like every Usenet user runs their own NNTP server. The design supports letting users just be users."
>
> — Satoshi Nakamoto, [BitcoinTalk](https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/127/) (July 2010)

And he explicitly proposed raising the block size limit via a scheduled upgrade:

> "It can be phased in, like: `if (blocknumber > 115000) maxblocksize = largerlimit`. It can start being in versions way ahead, so by the time it reaches that block number and goes into effect, the older versions that don't have it are already obsolete."
>
> — Satoshi Nakamoto, [BitcoinTalk](https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/233/) (October 4, 2010)

SegWit took a different approach than Satoshi's simple size increase — but the problem it solved was the one he identified.

## The Upgrade That Made Everything Possible

SegWit (BIP 141, activated August 2017) is the most consequential upgrade in Bitcoin's history. It fixed transaction malleability, enabled the Lightning Network, introduced script versioning, and — critically — created the weight discount that makes Ordinals economically viable.

## The Problem: Transaction Malleability

Before SegWit, anyone could modify the `scriptSig` of a transaction *without invalidating it*. The signature itself was valid, but the encoding could be tweaked (e.g., adding extra padding). This changed the transaction's hash (txid) without changing what the transaction actually did.

Why this matters:
- **Lightning Network was impossible** — payment channels rely on pre-signed transactions referencing specific txids. If a txid changes in-flight, the pre-signed transaction becomes invalid
- **Exchange accounting broke** — exchanges tracked withdrawals by txid. Malleable transactions could show as "unconfirmed forever" even though the funds arrived (Mt. Gox cited this as a contributing factor)

## The Solution: Move Signatures Out

SegWit literally **segregates the witness** (signature data) from the transaction. The witness data is included in blocks but excluded from the txid calculation.

```
BEFORE SegWit:                    AFTER SegWit:
┌──────────────────┐              ┌──────────────────┐
│ version          │              │ version          │
│ inputs           │              │ marker + flag    │  ← new
│   txid + vout    │              │ inputs           │
│   scriptSig ◄────── signature  │   txid + vout    │
│   sequence       │  in here     │   scriptSig      │  ← empty
│ outputs          │              │   sequence       │
│   value          │              │ outputs          │
│   scriptPubKey   │              │   value          │
│ locktime         │              │   scriptPubKey   │
└──────────────────┘              │ witness ◄────────── signatures here
                                  │ locktime         │
                                  └──────────────────┘
```

**txid** = hash of everything except witness data
**wtxid** = hash of everything including witness data

Since the signature is no longer part of the txid, malleability is fixed.

## Weight Units: The New Size Metric

SegWit replaced the simple byte-count block size limit with a **weight** system:

```
weight = (non-witness bytes × 4) + (witness bytes × 1)
```

The block weight limit is **4,000,000 weight units** (4 MWU), replacing the old 1 MB block size limit.

### Why 4:1?

This is the key insight. Non-witness data (inputs, outputs, amounts) counts 4× as much as witness data (signatures). This means:

| Data type | Weight per byte | Why |
|-----------|----------------|-----|
| Non-witness | 4 WU/byte | This data goes into the UTXO set or affects transaction meaning |
| Witness | 1 WU/byte | Signatures are only needed for validation, then can be pruned |

**The practical effect:** Witness data is 75% cheaper to put on-chain than non-witness data.

### Block capacity comparison:

- Old limit: 1,000,000 bytes (~1 MB)
- New limit: 4,000,000 weight units
- Typical SegWit block: ~1.5-2 MB actual data (since witness bytes are discounted)
- Maximum theoretical: ~4 MB (if a block were 100% witness data)

## Why This Matters for Ordinals

The 4:1 witness discount was designed to make signatures cheap. But the witness field can hold *arbitrary data* — not just signatures. This is how Ordinals/Inscriptions work:

1. Inscription data goes in the witness (1 WU per byte)
2. Without the discount, the same data would cost 4× more in fees
3. A 400 KB image in witness data only "costs" 400,000 WU — leaving room for actual transactions in the same block

This was an unintended consequence. SegWit's designers wanted cheaper signatures, not cheaper JPEGs. This tension drives the [BIP-110 debate](06-bip110-debate.md).

## Virtual Bytes (vBytes)

For fee calculation purposes, Bitcoin uses **virtual bytes**:

```
vsize = weight / 4  (rounded up)
```

Fee rates are expressed in **sat/vB** (satoshis per virtual byte). This normalizes fee calculations across legacy and SegWit transactions.

Example:
- Legacy transaction: 250 bytes = 1000 WU = 250 vB
- SegWit transaction: 150 non-witness + 100 witness = (150×4) + (100×1) = 700 WU = 175 vB
- Same fee rate → SegWit pays less total fee

## Script Versioning

SegWit introduced **witness versions** (0-16), enabling future upgrades without hard forks:

- **Witness v0**: P2WPKH, P2WSH (the original SegWit scripts)
- **Witness v1**: Taproot (P2TR) — activated November 2021
- **Witness v2-16**: Reserved for future soft forks

This is how Bitcoin can keep upgrading without breaking backward compatibility. Old nodes see SegWit outputs as "anyone-can-spend" (from their perspective), but they trust that miners enforce the new rules.

## Address Formats

| Type | Example prefix | Encoding | SegWit? |
|------|---------------|----------|---------|
| P2PKH | `1...` | Base58Check | No (legacy) |
| P2SH | `3...` | Base58Check | Wrapped SegWit possible |
| P2WPKH | `bc1q...` | Bech32 | Native SegWit v0 |
| P2WSH | `bc1q...` (longer) | Bech32 | Native SegWit v0 |
| P2TR | `bc1p...` | Bech32m | Native SegWit v1 |

Bech32 addresses are case-insensitive, have better error detection, and are more efficient in QR codes.

## How SegWit Activated: The UASF Drama

SegWit's activation was politically contentious:

1. **BIP 9** signaling: Miners signal readiness by setting bits in block headers
2. Miners stalled at ~30% signaling for months (some wanted bigger blocks instead)
3. **BIP 148 UASF** (User Activated Soft Fork): Node operators threatened to reject non-SegWit blocks after August 1, 2017
4. **BIP 91**: Miners capitulated and locked in SegWit days before the UASF deadline
5. SegWit activated at block 481,824 (August 24, 2017)

This established that users (node operators) ultimately control Bitcoin's consensus rules, not miners.

## Verify It Yourself

```bash
# Compare a legacy vs SegWit transaction
bitcoin-cli getrawtransaction <legacy-txid> 2
bitcoin-cli getrawtransaction <segwit-txid> 2

# Check the "weight" and "vsize" fields
# SegWit tx will have weight < size * 4

# See block weight stats
bitcoin-cli getblock <blockhash> 2 | grep -E "weight|size"
```

## Key Takeaways

1. SegWit moves signatures to a separate witness field, fixing malleability
2. Witness data gets a 75% discount (1 WU/byte vs 4 WU/byte)
3. Block limit is now 4M weight units (~1.5-2 MB typical, up to ~4 MB theoretical)
4. The witness discount unintentionally made on-chain data embedding cheap — enabling Ordinals
5. Script versioning enabled Taproot and future upgrades without hard forks

---

**Next:** [Chapter 4 — Taproot](04-taproot.md) — Schnorr signatures, MAST, and the upgrade that made inscriptions technically possible.
