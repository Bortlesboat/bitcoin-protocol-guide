# Chapter 4: Taproot

## The Upgrade Ordinals Couldn't Exist Without

Taproot (BIPs 340/341/342, activated November 2021) is the most elegant upgrade in Bitcoin's history. It replaced ECDSA with Schnorr signatures, introduced Merkelized Abstract Syntax Trees (MAST), and made complex scripts indistinguishable from simple payments on-chain.

It also, accidentally, created the exact mechanism Ordinals use to embed data.

## What Taproot Gives You

A P2TR (Pay-to-Taproot) output can be spent two ways:

```
                    P2TR Output
                    ┌─────────┐
                    │ tweaked  │
                    │ public   │
                    │ key      │
                    └────┬────┘
                   ┌─────┴─────┐
              Key Path     Script Path
              (fast)       (flexible)
                │               │
           Single Schnorr   Reveal a script
           signature        from the MAST
```

### Key-Path Spend
Just provide a Schnorr signature. On-chain, this looks identical to any other single-sig spend. Nobody can tell if there were hidden script conditions you *could* have used.

### Script-Path Spend
Reveal one specific script from a Merkle tree of possible scripts. Only the executed branch is published on-chain — unused branches stay hidden.

## Schnorr Signatures (BIP 340)

Taproot replaces ECDSA with Schnorr signatures. Why?

| Property | ECDSA | Schnorr |
|----------|-------|---------|
| Signature size | 71-72 bytes (DER encoded) | 64 bytes (fixed) |
| Linearity | No | Yes |
| Batch validation | No | Yes |
| Multi-signature | Requires CHECKMULTISIG | Key aggregation (MuSig2) |

### Linearity is the killer feature

Schnorr signatures are **linear**: you can add public keys together and add signatures together. This means a 3-of-3 multisig can produce a single public key and single signature that's indistinguishable from a solo spend.

```
# ECDSA multisig (old way, on-chain footprint):
OP_2 <pubkey1> <pubkey2> <pubkey3> OP_3 OP_CHECKMULTISIG
→ Reveals all 3 pubkeys + 2 signatures on-chain

# Schnorr + MuSig2 (new way):
<aggregated_pubkey>  →  <aggregated_sig>
→ Looks like a single-key spend. Nobody knows it was multisig.
```

### Batch Validation

Nodes can validate multiple Schnorr signatures simultaneously, ~2.5x faster than verifying each independently. This makes initial block download faster.

## MAST: Merkelized Abstract Syntax Trees

MAST lets you commit to a *tree* of possible scripts but only reveal the one you actually use.

```
            Root Hash (committed in the output)
               ┌──────┴──────┐
            Hash AB         Hash CD
          ┌───┴───┐       ┌───┴───┐
      Script A  Script B  Script C  Script D

To spend via Script C:
  Reveal: Script C + Hash D + Hash AB
  Verifier: hash(C) + hash(D) = Hash CD
            Hash CD + Hash AB = Root Hash ✓
```

### Why this matters:
- **Privacy**: Unused branches are never revealed
- **Efficiency**: Only the executed script goes on-chain
- **Composability**: You can have thousands of possible spending conditions in a single output

### Real example — Escrow with timeout:
```
Script A: 2-of-3 multisig (buyer, seller, arbitrator)
Script B: After 30 days, seller can claim unilaterally
Script C: After 90 days, buyer gets refund
```
If buyer and seller agree, they use the key-path (just a signature, no scripts revealed). The escrow conditions existed but were never published. Maximum privacy.

## Key Tweaking

Here's where it gets clever. A Taproot output commits to a single public key, but that key is *tweaked* to commit to the script tree:

```
tweaked_key = internal_key + hash(internal_key || merkle_root) × G
```

- **internal_key**: Your actual public key
- **merkle_root**: Root of your MAST tree (or nothing, for key-path only)
- **G**: The generator point (secp256k1 constant)

To spend via key-path: sign with the tweaked private key. Nobody can tell if a script tree exists.

To spend via script-path: reveal the internal key + merkle proof + the specific script.

## How Ordinals Use Taproot

Ordinals exploit the script-path mechanism:

1. **Commit transaction**: Create an output whose Taproot script tree contains the inscription data
2. **Reveal transaction**: Spend that output via the script-path, publishing the data on-chain

The inscription envelope is a script that looks like:

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1           // content type tag
  OP_PUSH "image/png" // MIME type
  OP_PUSH 0           // content tag
  OP_PUSH <data>      // actual file data (can be split across multiple pushes)
OP_ENDIF
<pubkey>
OP_CHECKSIG
```

The `OP_FALSE OP_IF ... OP_ENDIF` block **never executes** — the `OP_FALSE` ensures the `OP_IF` branch is skipped. The actual spending happens via `OP_CHECKSIG`. But the data is still published on-chain in the witness, permanently recorded in the blockchain.

This is why the data goes in the witness (cheap thanks to SegWit discount) and uses Taproot's script-path (to embed arbitrary data in the script tree). Without both SegWit and Taproot, this mechanism wouldn't exist.

More details in [Chapter 5](05-ordinals-inscriptions.md).

## Taproot Activation: Speedy Trial

Unlike SegWit's contentious activation, Taproot used "Speedy Trial" (BIP 8 variant):

- 90% miner signaling threshold
- Locked in June 12, 2021 (within the first signaling period)
- Activated at block 709,632 (November 14, 2021)
- Near-universal support — no controversy at the time

The irony: Taproot activated smoothly because everyone thought it was a privacy/efficiency upgrade. Nobody anticipated Ordinals (which launched in January 2023).

## Verify It Yourself

```bash
# Decode a Taproot transaction
bitcoin-cli getrawtransaction <p2tr-txid> 2

# Look for:
# - "type": "witness_v1_taproot" in scriptPubKey
# - witness field containing script-path data (if not key-path)
# - "witness": ["<sig>"] for key-path (just one 64-byte signature)

# Get output info for a Taproot UTXO
bitcoin-cli gettxout <txid> <vout>
```

## Key Takeaways

1. Taproot outputs have two spending paths: key-path (fast, private) and script-path (flexible)
2. Schnorr signatures are smaller, enable key aggregation, and support batch validation
3. MAST hides unused script branches — only the executed path is revealed
4. Key tweaking commits to a script tree without revealing it
5. Ordinals use the script-path to embed data in the witness via an `OP_FALSE OP_IF` envelope
6. Without both SegWit (cheap witness) and Taproot (script-path data), inscriptions wouldn't work

---

**Next:** [Chapter 5 — Ordinals & Inscriptions](05-ordinals-inscriptions.md) — How the theory becomes NFTs and tokens.
