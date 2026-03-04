# First BRC-20 Token Deploy: `ordi`

**The transaction that launched Bitcoin's fungible token ecosystem.**

## The Story

On March 8, 2023, an anonymous developer known as "domo" deployed the first BRC-20 token: `ordi`. The deploy inscription set a max supply of 21,000,000 (mirroring Bitcoin's supply cap) with a mint limit of 1,000 per transaction. Within weeks, the entire supply was minted and the token reached a market cap exceeding $1 billion.

## The Transaction

**txid:** `b61b0172d95e266c18aea0c624db987e971a5d6d4ebc2aaed85da4642d635735`

[View on mempool.space](https://mempool.space/tx/b61b0172d95e266c18aea0c624db987e971a5d6d4ebc2aaed85da4642d635735)

### Summary

| Field | Value |
|-------|-------|
| Block height | 779,832 |
| Date | March 8, 2023 |
| Version | 1 |
| Size | 362 bytes |
| Weight | 644 WU |
| Inputs | 1 |
| Outputs | 1 |
| Fee | 4,830 sats |
| Script type | P2TR (Taproot) |

### The Inscription Content

Embedded in the witness via the ordinals envelope:

```json
{
  "p": "brc-20",
  "op": "deploy",
  "tick": "ordi",
  "max": "21000000",
  "lim": "1000"
}
```

Content-type: `text/plain;charset=utf-8`

### What's Interesting About This Transaction

**1. Taproot script-path spend**

Both the input and output are P2TR (Taproot) addresses:
- Input: `bc1pt87kqa72x0v2qq3xlxuw0muz94umgqmcmk3eqq06hr8tcasjgppqd5r04w`
- Output: `bc1pxaneaf3w4d27hl2y93fuft2xk6m4u3wc4rafevc6slgd7f5tq2dqyfgy06`

This is the reveal transaction of a commit-reveal pair. The witness contains the script-path data including the inscription envelope.

**2. The witness data breakdown**

The transaction has 3 witness elements:

```
Element 0: <signature>                    (64 bytes — Schnorr)
Element 1: <inscription script>           (contains the BRC-20 JSON)
Element 2: <control block>                (33 bytes — internal key + parity)
```

Element 1 decoded contains:
```
<internal_pubkey>
OP_CHECKSIG
OP_FALSE
OP_IF
  OP_PUSH3 "ord"
  OP_1
  OP_PUSH "text/plain;charset=utf-8"
  OP_0
  OP_PUSHDATA1 <BRC-20 JSON payload>
OP_ENDIF
```

**3. Tiny transaction, massive impact**

The entire transaction is only 362 bytes (644 WU). The inscription content is a 94-byte JSON string. This tiny payload created a token system with billions of dollars in trading volume.

Compare this to the [pizza transaction](pizza-tx.md) at 23,620 bytes — this is 65× smaller but arguably more consequential for Bitcoin's ecosystem.

**4. The fee was negligible**

4,830 sats (~$1.50 at the time). In the weeks following, as BRC-20 mania took hold, inscription fees regularly exceeded 100+ sat/vB as users competed for block space.

**5. How BRC-20 actually works (and doesn't)**

The JSON is just data — Bitcoin nodes don't parse or validate it. BRC-20 balances exist only in off-chain indexers that:
1. Scan every inscription for BRC-20 JSON
2. Validate operations (deploy → mint → transfer)
3. Track balances per address

If an indexer has a bug, BRC-20 balances can disagree between services. This is fundamentally different from Bitcoin's UTXO model where consensus is enforced by the protocol.

## The Witness Hex (Decoded)

```
Witness element 1 (hex):
209e2849b90a2353691fccedd467215c88eec89a5d0dcf468e6cf37abed344d746
ac                              → OP_CHECKSIG
00                              → OP_FALSE
63                              → OP_IF
036f7264                        → OP_PUSH3 "ord"
01                              → OP_1 (content type tag)
18                              → OP_PUSH24
746578742f706c61696e3b636861727365743d7574662d38
                                → "text/plain;charset=utf-8"
00                              → OP_0 (body tag)
4c5e                            → OP_PUSHDATA1 94 bytes
7b200a2020...                   → BRC-20 JSON payload
68                              → OP_ENDIF
```

## Verify It Yourself

```bash
# Decode the full transaction
bitcoin-cli getrawtransaction b61b0172d95e266c18aea0c624db987e971a5d6d4ebc2aaed85da4642d635735 2

# Extract the witness data
bitcoin-cli getrawtransaction b61b0172d95e266c18aea0c624db987e971a5d6d4ebc2aaed85da4642d635735 2 | jq '.vin[0].txinwitness'

# The second witness element contains the inscription script
# Decode the hex to see the BRC-20 JSON
echo "7b200a2020227022..." | xxd -r -p
```

## Protocol Concepts Illustrated

- [x] **Taproot script-path** — inscription data in the witness via MAST
- [x] **Schnorr signature** — 64-byte fixed-size sig (element 0)
- [x] **OP_FALSE OP_IF envelope** — data present but never executed
- [x] **Witness discount** — 644 WU for 362 bytes (data is cheap in witness)
- [x] **Commit-reveal pattern** — this is the reveal; commit TX came first
- [x] **Off-chain indexing** — Bitcoin doesn't know this is a "token"

---

**Back to:** [Examples Index](../README.md#bonus) | [Chapter 5: Ordinals](../05-ordinals-inscriptions.md)
