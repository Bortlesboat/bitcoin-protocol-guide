# Chapter 5: Ordinals & Inscriptions

## What Satoshi Said About Data on the Blockchain

> "It would be unwise to have permanently recorded plaintext messages for everyone to see. It would be an accident waiting to happen. If there's going to be a message system, it should be a separate system parallel to the bitcoin network. Messages should not be recorded in the block chain."
>
> — Satoshi Nakamoto, [BitcoinTalk](https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/239/) (October 23, 2010)

This quote is the primary ammunition in the "spam debate" — Satoshi explicitly opposed non-financial data on-chain. Ordinals launched 13 years later, using mechanisms (SegWit + Taproot) that didn't exist when he wrote this.

## NFTs and Tokens on Bitcoin — Using Only Existing Rules

Ordinals didn't require any protocol change. Casey Rodarmor built the entire system using Bitcoin's existing consensus rules — SegWit's witness discount, Taproot's script-path spending, and the UTXO model itself. That's what makes the debate so interesting.

## Ordinal Theory: Numbering Satoshis

Every satoshi that has ever existed or will exist gets a unique **ordinal number** based on the order it was mined:

- The first satoshi ever mined (block 0, first output) = ordinal 0
- The 50 billionth satoshi (end of block 0) = ordinal 4,999,999,999
- Satoshis from block 1 start at ordinal 5,000,000,000

```
Block 0:  sats 0 — 4,999,999,999
Block 1:  sats 5,000,000,000 — 9,999,999,999
Block 2:  ...
(after 1st halving, blocks produce 25 BTC = 2.5B sats each)
```

### Tracking transfers

When a transaction spends UTXOs, ordinals transfer using **first-in-first-out (FIFO)** ordering:

```
Input 0: sats [0, 1, 2, 3, 4]      Output 0: sats [0, 1, 2]  (value: 3 sats)
Input 1: sats [5, 6, 7]            Output 1: sats [3, 4, 5]  (value: 3 sats)
                                    Output 2: sats [6, 7]      (value: 2 sats)
```

Sats flow from inputs to outputs in order. This is a convention, not a protocol rule — Bitcoin nodes don't track ordinals. Only ordinal-aware software (like `ord`) interprets transactions this way.

### Rarity

Because ordinals follow mining events, some are "rarer" than others:

| Rarity | Event | Frequency |
|--------|-------|-----------|
| Common | Any non-special sat | ~2.1 quadrillion |
| Uncommon | First sat of a block | ~6.9 million |
| Rare | First sat after difficulty adjustment | ~3,437 |
| Epic | First sat after halving | 32 (ever) |
| Legendary | First sat of a cycle (halving × difficulty) | 5 (ever) |
| Mythic | First sat of block 0 | 1 |

This is entirely a social convention — the protocol doesn't care.

## Inscriptions: Data on Bitcoin

An **inscription** attaches content to a specific satoshi. The content is embedded in a Taproot script-path witness using the envelope format from [Chapter 4](04-taproot.md).

### The Commit-Reveal Pattern

Inscriptions use a two-transaction pattern:

**Step 1 — Commit Transaction:**
Create a Taproot output whose script tree contains the inscription data. The output itself looks like any other P2TR output — you can't tell it contains an inscription by looking at it.

**Step 2 — Reveal Transaction:**
Spend the commit output using the script-path, which publishes the inscription data on-chain in the witness field.

```
Commit TX:                           Reveal TX:
┌──────────┐                        ┌──────────────────────────┐
│ Input:   │                        │ Input: commit output     │
│ funding  │──►┌──────────┐         │ Witness:                 │
│ UTXO     │   │ P2TR     │──────►  │   signature              │
└──────────┘   │ output   │         │   inscription script:    │
               │ (looks   │         │     OP_FALSE OP_IF       │
               │ normal)  │         │       "ord"              │
               └──────────┘         │       content-type       │
                                    │       <image data>       │
                                    │     OP_ENDIF             │
                                    │     <pubkey> OP_CHECKSIG │
                                    │   control block          │
                                    └──────────────────────────┘
```

### Why two transactions?

The commit transaction hides the inscription data until reveal. This prevents front-running — nobody can see what you're inscribing and race to inscribe it first on a different sat.

### Inscription Envelope Detail

```
witness:
  <signature>
  <script>:
    OP_FALSE           // pushes 0 to stack
    OP_IF              // 0 is falsy, so this block is SKIPPED
      OP_PUSH3 "ord"   // ordinals protocol marker
      OP_1             // tag: content type
      OP_PUSH "image/png"
      OP_0             // tag: body
      OP_PUSH <chunk1> // data (max 520 bytes per push)
      OP_PUSH <chunk2> // continued...
      OP_PUSH <chunk3>
    OP_ENDIF
    <pubkey>
    OP_CHECKSIG        // actual spending condition
  <control block>      // Taproot internal key + merkle proof
```

The `OP_FALSE OP_IF` trick means the inscription data is **present in the witness** (recorded on-chain) but **never executed** by the script interpreter. The only thing that actually runs is `<pubkey> OP_CHECKSIG`.

### Size limits

- Each `OP_PUSH` can push up to **520 bytes**
- Multiple pushes can be chained for larger content
- Taproot removed the old 10,000-byte script size limit
- Only constrained by the **4M weight unit block limit**
- Maximum single inscription: ~400 KB (nearly filling an entire block's witness)

## BRC-20 Tokens

BRC-20 is a token standard built *on top of* inscriptions. It embeds JSON in a text inscription:

### Deploy a new token:
```json
{
  "p": "brc-20",
  "op": "deploy",
  "tick": "ordi",
  "max": "21000000",
  "lim": "1000"
}
```

### Mint tokens:
```json
{
  "p": "brc-20",
  "op": "mint",
  "tick": "ordi",
  "amt": "1000"
}
```

### Transfer tokens:
```json
{
  "p": "brc-20",
  "op": "transfer",
  "tick": "ordi",
  "amt": "500"
}
```

**Important:** Bitcoin nodes don't understand BRC-20. The token balances are tracked by off-chain indexers that read inscription data and maintain a state. If two indexers disagree on the rules, they'll show different balances. This is fundamentally different from Ethereum's ERC-20 where the smart contract IS the ledger.

## The Economics: Why Inscriptions Use Witness Data

Thanks to SegWit's witness discount ([Chapter 3](03-segwit.md)):

| Where data goes | Cost per byte | For 100 KB |
|-----------------|---------------|------------|
| Non-witness (scriptPubKey) | 4 WU | 400,000 WU |
| Witness (inscription) | 1 WU | 100,000 WU |

At 10 sat/vB, inscribing 100 KB in witness costs roughly:
- 100,000 WU / 4 = 25,000 vB × 10 sat/vB = 250,000 sats (~$250 at $100K/BTC)

The same data in non-witness would cost 4× more. This is why inscriptions exclusively use the witness field.

## Runes: A Better Token Standard

**Runes** (launched April 2024 at the halving) was Casey Rodarmor's answer to BRC-20's inefficiency. Key differences:

| | BRC-20 | Runes |
|---|---|---|
| Data location | Inscription (witness) | OP_RETURN (80 bytes) |
| UTXO impact | Creates junk UTXOs | UTXO-aware (clean) |
| Indexer dependency | Heavy | Lighter |
| Creator | @domodata | @rodarmor |

Runes use the UTXO model natively — token balances are attached to UTXOs and transfer following standard Bitcoin transaction rules. This is more Bitcoin-native than BRC-20's off-chain indexer approach.

## Verify It Yourself

```bash
# Decode an inscription reveal transaction
bitcoin-cli getrawtransaction <reveal-txid> 2

# Look at the witness field — you'll see the OP_FALSE OP_IF envelope
# The hex between OP_IF and OP_ENDIF is the inscription data

# Check the ord indexer for inscription details
# (requires running the ord binary)
ord index
ord inscription <inscription-id>
```

You can also use [mempool.space](https://mempool.space) or [ordinals.com](https://ordinals.com) to view inscriptions without running your own indexer.

## Key Takeaways

1. Ordinal theory assigns unique numbers to every satoshi — a social convention, not a protocol rule
2. Inscriptions embed data in Taproot witness using `OP_FALSE OP_IF` — data is recorded but never executed
3. Commit-reveal prevents front-running of inscriptions
4. SegWit's witness discount makes inscriptions 4× cheaper than other on-chain data
5. BRC-20 tokens are JSON inscriptions tracked by off-chain indexers — not enforceable by consensus
6. Runes is a UTXO-native alternative that doesn't bloat the UTXO set
7. Inscription fees are now a significant component of miner revenue (see [Chapter 10](10-mining-hardware.md) for why this matters post-halving)
8. RGB protocol offers an alternative: client-side validated tokens that don't use block space at all (see [Chapter 12](12-future-bitcoin.md))

---

**Next:** [Chapter 6 — The BIP-110 Debate](06-bip110-debate.md) — Should Bitcoin restrict this? Both sides of the "spam war."
