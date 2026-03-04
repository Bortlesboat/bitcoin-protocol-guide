# Chapter 2: The Script System

## In Satoshi's Words

> "The nature of Bitcoin is such that once version 0.1 was released, the core design was set in stone for the rest of its lifetime. Because of that, I wanted to design it to support every possible transaction type I could think of. The problem was, each thing required special support code and data fields whether it was used or not, and only covered one special case at a time. It would have been an explosion of special cases. The solution was script, which generalizes the problem so transacting parties can describe their transaction as a predicate that the node network evaluates."
>
> — Satoshi Nakamoto, [BitcoinTalk](https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/69/) (June 17, 2010)

## Bitcoin's Hidden Programming Language

Every UTXO is locked by a small program called a **script**. To spend a UTXO, you must provide another script that makes the combined program evaluate to `true`. This is Bitcoin's programmability layer — simple, deliberate, and intentionally limited.

## How Script Execution Works

Bitcoin Script is **stack-based**, like Forth or a reverse Polish notation calculator. There are no loops (by design — scripts must terminate), no floating point, and no access to external state.

Spending a UTXO runs two scripts in sequence:

```
scriptSig (provided by spender) + scriptPubKey (set by previous creator)
```

As Satoshi put it:

> "The script is actually a predicate. It's just an equation that evaluates to true or false. Predicate is a long and unfamiliar word so I called it script."

The combined script executes on a stack. If the stack's top value is non-zero (truthy) when execution finishes, the spend is valid.

### Example: P2PKH (Pay-to-Public-Key-Hash)

This is the original Bitcoin address type (starts with `1`).

**scriptPubKey** (the lock):
```
OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
```

**scriptSig** (the key):
```
<sig> <pubKey>
```

**Execution trace:**

```
Stack              | Operation
-------------------|------------------------------------------
                   | Start with scriptSig
[sig]              | Push <sig>
[sig, pubKey]      | Push <pubKey>
                   | Now execute scriptPubKey
[sig, pubKey,      | OP_DUP — duplicate top item
 pubKey]           |
[sig, pubKey,      | OP_HASH160 — hash the top item
 hash(pubKey)]     |
[sig, pubKey,      | Push <pubKeyHash>
 hash(pubKey),     |
 pubKeyHash]       |
[sig, pubKey]      | OP_EQUALVERIFY — check equality, fail if not
[true]             | OP_CHECKSIG — verify signature against pubKey
```

The script checks: "Does the provided public key hash to the address, AND does the signature match that key?" Both must be true.

## Script Types: A History of Upgrades

Each new script type solved specific problems:

### P2PKH — Pay to Public Key Hash (2009)
- Address format: `1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa`
- The original. Lock to a hash of a public key
- Problem: No way to create complex spending conditions

Satoshi had already planned for these advanced use cases:

> "The design supports a tremendous variety of possible transaction types that I designed years ago. Escrow transactions, bonded contracts, third party arbitration, multi-party signature, etc. If Bitcoin catches on in a big way, these are things we'll want to explore in the future, but they all had to be designed at the beginning to make sure they would be possible later."
>
> — Satoshi Nakamoto, [BitcoinTalk](https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/69/) (June 17, 2010)

### P2SH — Pay to Script Hash (2012, BIP 16)
- Address format: `3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy`
- Lock to a *hash of a script*. The actual script is only revealed when spending
- Enables: multisig, timelocks, any complex condition
- The spender bears the cost of the complex script, not the sender

```
# P2SH scriptPubKey (simple — just a hash check)
OP_HASH160 <scriptHash> OP_EQUAL

# P2SH scriptSig (reveals the actual script)
<signatures...> <redeemScript>
```

**Example — 2-of-3 Multisig via P2SH:**
```
redeemScript = OP_2 <pubKey1> <pubKey2> <pubKey3> OP_3 OP_CHECKMULTISIG
```
The sender just sees a short hash. The complex multisig script is hidden until spending time.

### P2WPKH — Pay to Witness Public Key Hash (2017, SegWit)
- Address format: `bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t4` (bech32)
- Same logic as P2PKH but signature data moves to the *witness* field
- Result: smaller transaction weight, lower fees
- Detailed in [Chapter 3](03-segwit.md)

### P2WSH — Pay to Witness Script Hash (2017, SegWit)
- Like P2SH but with witness data
- Enables complex scripts with the SegWit weight discount

### P2TR — Pay to Taproot (2021, BIP 340/341/342)
- Address format: `bc1p...` (bech32m)
- Two spending paths: key-path (just a signature) or script-path (reveal a script)
- Uses Schnorr signatures instead of ECDSA
- Detailed in [Chapter 4](04-taproot.md)

## Notable Opcodes

| Opcode | What it does |
|--------|-------------|
| `OP_DUP` | Duplicate top stack item |
| `OP_HASH160` | SHA-256 then RIPEMD-160 |
| `OP_EQUAL` | Check if top two items are equal |
| `OP_EQUALVERIFY` | `OP_EQUAL` + fail immediately if false |
| `OP_CHECKSIG` | Verify a signature against a public key |
| `OP_CHECKMULTISIG` | Verify M-of-N signatures |
| `OP_RETURN` | Mark output as provably unspendable (used for data embedding) |
| `OP_CHECKLOCKTIMEVERIFY` | Require a minimum block height or timestamp |
| `OP_CHECKSEQUENCEVERIFY` | Require a minimum relative timelock |
| `OP_IF` / `OP_ELSE` / `OP_ENDIF` | Conditional execution |

### Disabled opcodes

Satoshi disabled several opcodes early on due to security concerns:
- `OP_CAT` (concatenate) — now proposed for re-enabling (potential covenants)
- `OP_MUL`, `OP_DIV` — arithmetic that could cause issues
- `OP_SUBSTR`, `OP_LEFT`, `OP_RIGHT` — string manipulation

These are **consensus rules** — any transaction using them is invalid, period.

## OP_RETURN: Data Embedding

`OP_RETURN` creates a provably unspendable output. Nodes can safely prune these from the UTXO set since they can never be spent.

```
OP_RETURN <up to 80 bytes of arbitrary data>
```

Before OP_RETURN, people embedded data by creating fake addresses (which bloated the UTXO set permanently). OP_RETURN was a compromise: "If you must put data on-chain, at least don't pollute the UTXO set."

This is relevant to the [Ordinals debate](06-bip110-debate.md) — OP_RETURN has a size limit, but witness data does not (post-SegWit).

## Sighash Types

When signing a transaction, you choose *what parts* of the transaction the signature covers:

| Sighash | Signs | Use case |
|---------|-------|----------|
| `SIGHASH_ALL` | All inputs + all outputs | Normal spending (default) |
| `SIGHASH_NONE` | All inputs, no outputs | "I'm paying, send it wherever" |
| `SIGHASH_SINGLE` | All inputs + matching output | Partial signing for swaps |
| `SIGHASH_ANYONECANPAY` | Only this input | Combinable with above — crowdfunding |

`ANYONECANPAY | ALL` means: "I sign my input and all outputs, but anyone can add more inputs." This enables on-chain crowdfunding — multiple people contribute inputs to fund a shared set of outputs.

## Verify It Yourself

```bash
# Decode a transaction to see scriptSig and scriptPubKey
bitcoin-cli getrawtransaction <txid> 2

# Decode a raw script (hex)
bitcoin-cli decodescript <hex>

# See the fields:
# "asm" — human-readable opcodes
# "type" — script type (pubkeyhash, scripthash, witness_v0_keyhash, witness_v1_taproot)
```

## Key Takeaways

1. Every UTXO is locked by a script (scriptPubKey)
2. Spending requires a matching script (scriptSig/witness) that makes the combined program return true
3. Script is stack-based with no loops — intentionally limited
4. Script types evolved: P2PKH → P2SH → P2WPKH/P2WSH → P2TR
5. Each upgrade enabled new capabilities while maintaining backward compatibility (soft forks)

---

**Next:** [Chapter 3 — SegWit](03-segwit.md) — How moving signatures out of the transaction changed everything.
