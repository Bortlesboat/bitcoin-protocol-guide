# Chapter 9: Wallets & Self-Custody

## In Satoshi's Words

> "The root problem with conventional currency is all the trust that's required to make it work. The central bank must be trusted not to debase the currency, but the history of fiat currencies is full of breaches of that trust. Banks must be trusted to hold our money and transfer it electronically, but they lend it out in waves of credit bubbles with barely a fraction in reserve."
>
> — Satoshi Nakamoto, [P2P Foundation](http://p2pfoundation.ning.com/forum/topics/bitcoin-open-source), February 11, 2009

That quote isn't about wallets. It's about why wallets exist. Satoshi's entire design assumed that individuals would hold their own keys. There is no "Bitcoin bank" in the whitepaper. There is no custodian. There is no account recovery department. The system is built — from the cryptographic primitives up — on the premise that the person who controls the private key controls the money.

This chapter is about how to actually do that without losing everything.

---

## Part I: What Is a "Wallet"?

A Bitcoin wallet is a misleading term. It doesn't hold Bitcoin. Nothing "holds" Bitcoin — UTXOs exist on a distributed ledger maintained by thousands of nodes. What a wallet actually holds is **private keys** (or the seed material to derive them). The wallet software's job is to:

1. Generate and store private keys
2. Derive addresses from those keys
3. Monitor the blockchain for UTXOs spendable by those keys
4. Construct and sign transactions when you want to spend

If you lose the keys, the Bitcoin still exists on the blockchain. It's just permanently unspendable. Chainalysis estimates that 3.7 million BTC — roughly 17.6% of the total supply — is lost forever, most of it from the early years when people didn't understand what they were holding.

### Wallet vs. Account

In traditional finance, you have an "account" at an institution. The institution holds the money. You have a credential (password, card number) to instruct the institution. If you lose the credential, the institution issues a new one. The money never moves.

In Bitcoin, there is no institution. The key IS the money. If you lose the key, there is no recovery. If someone copies the key, they can spend everything. There is no customer service number. There is no chargeback.

This is the fundamental tradeoff of self-custody: you remove the counterparty risk of trusting an institution, but you assume the operational risk of securing your own keys. Both risks are real. The question is which one you're better equipped to manage.

---

## Part II: Wallet Types

### Hot Wallets

A **hot wallet** is any wallet where the private keys exist on an internet-connected device. This includes:

- **Mobile wallets** — BlueWallet, Muun, Phoenix, Green (Blockstream), Nunchuk
- **Desktop wallets** — Sparrow, Electrum, Bitcoin Core (with wallet enabled), Wasabi
- **Browser extensions** — Alby (Lightning-focused), Xverse (Ordinals-focused)

Hot wallets are convenient for daily spending. They are NOT appropriate for storing significant amounts of Bitcoin. A hot wallet on your phone is the equivalent of the cash in your physical wallet — useful for buying coffee, not for holding your savings.

**The threat model is simple:** if malware can access the device's memory, it can access the keys. Android and iOS sandboxing provide some protection, but sophisticated attacks (zero-day exploits, compromised apps, clipboard hijacking) can breach it.

### Cold Wallets (Hardware Wallets)

A **cold wallet** keeps private keys on a device that never touches the internet. The keys are generated on the device, stored on the device, and used for signing on the device. The signed transaction is then transferred to an online device for broadcasting.

Hardware wallets are the standard recommendation for any amount of Bitcoin you'd be upset to lose. We'll cover specific hardware wallets in detail in Part V.

### Air-Gapped Wallets

An **air-gapped wallet** takes cold storage further: the signing device has no USB, Bluetooth, WiFi, or NFC connection to any other device. Communication happens exclusively through:

- **QR codes** — The signing device displays a QR code; the online device scans it (and vice versa)
- **MicroSD cards** — Transaction data is transferred via removable storage (ColdCard's approach)

Air-gapping eliminates an entire class of firmware-level attacks. Even if the hardware wallet's firmware were malicious, it would need a communication channel to exfiltrate keys — and an air-gapped device has none.

ColdCard, SeedSigner, and Jade (in QR mode) support full air-gapped operation.

### Paper Wallets

A **paper wallet** is a printed private key (typically as a QR code and/or WIF-encoded string). They were popular in 2013–2016 and are now considered obsolete for several reasons:

1. **No partial spending** — To spend any amount, you must import the entire private key into a hot wallet, exposing it. If you don't sweep the entire balance immediately, the change goes to a new address that your paper wallet doesn't control.
2. **Generation risk** — Most paper wallet generators ran in web browsers. If the page was compromised, every printed wallet was compromised.
3. **Physical fragility** — Paper degrades. Ink fades. Water destroys it. Fire destroys it.
4. **Single address** — Paper wallets are a single key/address pair. No HD derivation, no privacy rotation.

If you find an old paper wallet, sweep the entire balance to a modern HD wallet immediately. Don't try to spend "part" of it.

### Brain Wallets (And Why They're Terrible)

A **brain wallet** is a private key derived from a passphrase you memorize. The passphrase is hashed (typically SHA-256) to produce a 256-bit private key.

Brain wallets are catastrophically insecure. Here's why:

**Humans are bad at entropy.** A 256-bit private key requires 256 bits of entropy to be secure. The average English sentence has approximately 1–2 bits of entropy per character. A "strong" 20-word passphrase that you can actually remember has maybe 40–60 bits of entropy. That's 2^200 times weaker than a properly random key.

**Automated brainwallet crackers exist.** Services and scripts continuously hash dictionaries, common phrases, song lyrics, Bible verses, literary quotes, and password lists, then check the resulting addresses for balances. If your brain wallet passphrase appears in any corpus of human language ever recorded, it will be found — usually within hours of receiving funds.

Real examples of brain wallets that were drained:
- `"correct horse battery staple"` (the XKCD example) — drained instantly
- `"how much wood could a woodchuck chuck if a woodchuck could chuck wood"` — drained
- `"to be or not to be that is the question"` — drained
- Random-seeming phrases that people believed were unique — also drained

**Do not use brain wallets.** Use BIP 39 mnemonic seeds generated by a hardware wallet's true random number generator.

---

## Part III: Key Generation — From Entropy to Addresses

### The BIP 39 Standard: Mnemonic Seeds

BIP 39 (Mnemonic code for generating deterministic keys) defines the standard that most Bitcoin wallets use today. The process has four steps:

```
Entropy (128 or 256 bits of randomness)
    │
    ▼
Mnemonic words (12 or 24 words from a fixed 2048-word list)
    │
    ▼
Seed (512-bit value derived via PBKDF2)
    │
    ▼
Master key pair (root of the HD derivation tree)
```

**Step 1: Generate entropy.** The wallet's random number generator produces 128 bits (for 12 words) or 256 bits (for 24 words) of entropy. This MUST come from a cryptographically secure random number generator (CSPRNG). Hardware wallets use dedicated hardware RNGs; software wallets use the OS-provided CSPRNG.

**Step 2: Compute the checksum.** The entropy is SHA-256 hashed. The first `ENT/32` bits of the hash are appended to the entropy. For 128-bit entropy: 4 checksum bits → 132 bits total. For 256-bit entropy: 8 checksum bits → 264 bits total.

**Step 3: Map to words.** The resulting bit string is split into 11-bit groups. Each 11-bit value (0–2047) indexes into the BIP 39 word list. 132 bits / 11 = 12 words. 264 bits / 11 = 24 words.

| Entropy (bits) | Checksum (bits) | Total (bits) | Words |
|---------------|----------------|-------------|-------|
| 128 | 4 | 132 | 12 |
| 160 | 5 | 165 | 15 |
| 192 | 6 | 198 | 18 |
| 224 | 7 | 231 | 21 |
| 256 | 8 | 264 | 24 |

**Step 4: Derive the seed.** The mnemonic words (as a UTF-8 string) are fed through PBKDF2-HMAC-SHA512 with 2048 iterations. The salt is `"mnemonic" + passphrase` (if a passphrase is used; otherwise just `"mnemonic"`). This produces a 512-bit seed.

**Why 24 words instead of 12?** Twelve words provide 128 bits of entropy — sufficient security against brute force for the foreseeable future. Twenty-four words provide 256 bits, which adds margin against hypothetical future attacks (including speculative quantum computing scenarios). Most hardware wallets default to 24 words. Both are considered safe today.

The BIP 39 word list is carefully designed:
- No word is a prefix of another (e.g., "build" and "building" cannot both appear)
- The first four letters uniquely identify each word — so you only need to stamp four letters on steel
- Words were chosen for unambiguous spelling across English speakers

### BIP 32: Hierarchical Deterministic Derivation

Once you have a 512-bit seed, **BIP 32** (Hierarchical Deterministic Wallets, authored by Pieter Wuille) defines how to derive an entire tree of key pairs from it.

The seed is HMAC-SHA512 hashed with the key `"Bitcoin seed"` to produce:
- Left 256 bits → master private key
- Right 256 bits → master chain code

From this master key pair, child keys are derived using a tree structure:

```
Master (m)
├── m/0     (first child)
│   ├── m/0/0     (first grandchild)
│   └── m/0/1
├── m/1
│   ├── m/1/0
│   └── m/1/1
└── m/2
    └── ...
```

Each derivation level uses an **index** (32-bit integer). Indices below 2^31 are **normal** derivation; indices at or above 2^31 are **hardened** derivation (written with `'` or `h`). Hardened derivation prevents a leaked child public key + chain code from compromising parent keys.

**Why this matters:** a single 12- or 24-word seed phrase backs up an unlimited number of addresses. Before BIP 32, wallets generated independent random keys for each address, and you had to back up each key individually. Losing a backup meant losing funds. HD wallets eliminated that problem entirely.

### BIP 44/49/84/86: Derivation Path Standards

BIP 32 defines the tree structure; these BIPs define which paths to use for which address types:

```
m / purpose' / coin_type' / account' / change / address_index
```

| BIP | Purpose | Address Type | Prefix | Example Path |
|-----|---------|-------------|--------|-------------|
| BIP 44 | 44' | P2PKH (Legacy) | 1... | `m/44'/0'/0'/0/0` |
| BIP 49 | 49' | P2SH-SegWit (Nested) | 3... | `m/49'/0'/0'/0/0` |
| BIP 84 | 84' | P2WPKH (Native SegWit) | bc1q... | `m/84'/0'/0'/0/0` |
| BIP 86 | 86' | P2TR (Taproot) | bc1p... | `m/86'/0'/0'/0/0` |

The path components:
- **purpose'** — Hardened. Identifies the address standard.
- **coin_type'** — Hardened. `0'` for Bitcoin mainnet, `1'` for testnet. (Defined in SLIP 44.)
- **account'** — Hardened. Allows multiple logical accounts under one seed. `0'` is the first account.
- **change** — `0` for receiving addresses, `1` for change addresses. NOT hardened.
- **address_index** — Sequential index within the chain. NOT hardened.

You can verify this in Bitcoin Core with descriptor wallets:

```bash
# Create a descriptor wallet
bitcoin-cli createwallet "test_descriptors" false true "" false true

# List wallet descriptors — shows the derivation paths in use
bitcoin-cli -rpcwallet=test_descriptors listdescriptors

# Example output includes descriptors like:
# wpkh([fingerprint/84h/0h/0h]xpub.../0/*)#checksum  ← BIP 84 receive
# wpkh([fingerprint/84h/0h/0h]xpub.../1/*)#checksum  ← BIP 84 change
# tr([fingerprint/86h/0h/0h]xpub.../0/*)#checksum     ← BIP 86 receive (Taproot)
```

### The xpub, ypub, zpub Confusion

Different extended public key formats signal which address type should be derived:

| Key Prefix | BIP | Address Type | Encoding |
|-----------|-----|-------------|----------|
| xpub | 44 | P2PKH (1...) | Base58 |
| ypub | 49 | P2SH-SegWit (3...) | Base58 |
| zpub | 84 | Native SegWit (bc1q...) | Base58 |

These are encoding conventions, not protocol-level distinctions. The underlying key data is the same — the prefix just tells wallet software which derivation scheme to use. Sparrow Wallet, for example, will show you all four formats for the same seed.

**Warning:** Sharing an xpub/ypub/zpub with anyone allows them to derive ALL your past and future receiving addresses. They can't spend your Bitcoin, but they can see your entire transaction history and balance. Treat extended public keys as sensitive information.

---

## Part IV: Address Types and Wallet Support

Bitcoin has four address formats in active use. Each corresponds to a different output script type (see [Chapter 2](02-script-system.md) for the script-level details).

| Address Type | Prefix | Script | BIP | Introduced | Fee Efficiency |
|-------------|--------|--------|-----|-----------|----------------|
| P2PKH (Legacy) | `1...` | OP_DUP OP_HASH160 ... | — | 2009 (original) | Lowest |
| P2SH-SegWit (Nested) | `3...` | OP_HASH160 ... (wraps SegWit) | 49 | 2017 | Medium |
| P2WPKH (Native SegWit) | `bc1q...` | OP_0 <20-byte hash> | 84 | 2017 | High |
| P2TR (Taproot) | `bc1p...` | OP_1 <32-byte key> | 86 | 2021 | Highest (key-path) |

### Which Address Type Should You Use?

**Native SegWit (bc1q) is the default recommendation** for most users in 2026. It's supported by essentially all wallets and exchanges, has the lowest fees of the non-Taproot types, and is well-understood.

**Taproot (bc1p) is the forward-looking choice.** Better privacy properties (key-path spends look identical regardless of the underlying script complexity), lowest fees for key-path spending, and required for advanced features like MuSig2 key aggregation. Adoption is growing but some older services still don't support sending to bc1p addresses.

**P2SH-SegWit (3...) exists for backward compatibility.** In 2017–2019, many services couldn't send to bc1 addresses. The 3... format wrapped SegWit inside P2SH so it was universally accepted. There's no reason to use it for new wallets today.

**Legacy P2PKH (1...) should be avoided** for new wallets. Higher fees, no SegWit benefits, no Taproot benefits. The only reason to interact with 1... addresses is if you're recovering old wallets.

```bash
# Derive addresses of different types from a descriptor wallet in Bitcoin Core
bitcoin-cli -rpcwallet=test_descriptors deriveaddresses "wpkh([fingerprint/84h/0h/0h]xpub.../0/*)#checksum" "[0,4]"
# Returns 5 native SegWit (bc1q) addresses

bitcoin-cli -rpcwallet=test_descriptors deriveaddresses "tr([fingerprint/86h/0h/0h]xpub.../0/*)#checksum" "[0,4]"
# Returns 5 Taproot (bc1p) addresses
```

### Wallet Compatibility Matrix

Not all wallets support all address types. As of 2026:

| Wallet | P2PKH (1...) | P2SH-SegWit (3...) | Native SegWit (bc1q) | Taproot (bc1p) | Multisig |
|--------|:---:|:---:|:---:|:---:|:---:|
| Sparrow | Yes | Yes | Yes | Yes | Yes |
| Electrum | Yes | Yes | Yes | Yes | Yes |
| Bitcoin Core | Yes | Yes | Yes | Yes | Yes |
| BlueWallet | Yes | Yes | Yes | Yes | Vault |
| ColdCard | Yes | Yes | Yes | Yes | Yes |
| Ledger (Live) | Yes | Yes | Yes | Yes | Limited |
| Trezor (Suite) | Yes | Yes | Yes | Yes | Limited |
| Jade | Yes | Yes | Yes | Yes | Yes |
| Nunchuk | — | — | Yes | Yes | Yes |

---

## Part V: Hardware Wallets — Security Model Comparison

Hardware wallets are the primary tool for self-custody of significant Bitcoin holdings. They all solve the same core problem (keep keys offline) but their security models, trust assumptions, and philosophies differ substantially.

### The Comparison

| Device | Maker | Secure Element | Open Source (Firmware) | Open Source (Hardware) | Air-Gap | Price Range | Key Differentiator |
|--------|-------|:-:|:-:|:-:|:-:|-----------|-------------------|
| **Ledger Nano S+/X** | Ledger (FR) | Yes (ST33) | No | No | No | $80–150 | Widest altcoin support; closed firmware |
| **Ledger Stax/Flex** | Ledger (FR) | Yes (ST33) | No | No | Bluetooth | $280–400 | E-ink touchscreen; consumer-friendly |
| **Trezor Model T/Safe** | SatoshiLabs (CZ) | No* | Yes | Yes | No | $70–180 | First HW wallet; fully open source |
| **ColdCard Mk4/Q** | Coinkite (CA) | Yes (ATECC608B) | Yes | No | Yes (SD/NFC) | $150–240 | Bitcoin-only; paranoid-grade security |
| **Jade** | Blockstream | No (virtual SE) | Yes | Yes | Yes (QR/BLE) | $65 | Cheapest air-gap; no secure element by design |
| **SeedSigner** | Community/DIY | No | Yes | Yes | Yes (QR) | $50–100 (parts) | Stateless; build it yourself |
| **BitKey** | Block/Bitkey | Yes | Partial | Partial | No (NFC) | $150 | 2-of-3 multisig built in; Dorsey-backed |

*Trezor Safe 5 added a secure element (Optiga Trust M); earlier models had none.

### Security Model Deep Dive

**Ledger** uses a secure element (a tamper-resistant chip designed for smartcards and payment systems) running a proprietary real-time OS called BOLOS. The firmware is NOT open source. You are trusting Ledger's internal audit and their secure element vendor (STMicroelectronics) that the chip does what it claims. Ledger's argument: open-sourcing the firmware would let attackers study it; the secure element's tamper resistance is the security boundary.

The counterargument: closed firmware means you can't verify that Ledger isn't extracting your keys. In May 2023, Ledger announced "Ledger Recover," a service that would shard your seed and send encrypted fragments to three custodians (Ledger, Coincover, EscrowTech). The Bitcoin community reacted with outrage — not because the service was mandatory (it wasn't), but because it proved the firmware COULD extract the seed if told to. This confirmed what open-source advocates had always argued: you're trusting Ledger not to push a malicious firmware update.

**Trezor** takes the opposite approach: fully open-source firmware and hardware schematics. No secure element (until recently). The tradeoff: without a secure element, a physical attacker with access to the device can extract keys through voltage glitching or flash memory reads. Trezor relies on your PIN and optional passphrase to protect against physical theft. Against remote attacks, the open-source firmware is auditable — which is a stronger guarantee than Ledger's trust model.

**ColdCard** combines both: a secure element (for key storage) AND open-source firmware (for auditability). It's Bitcoin-only — it intentionally doesn't support any altcoin, which dramatically reduces the attack surface. ColdCard pioneered air-gapped signing via MicroSD card: you export a PSBT to SD, sneaker-net it to the ColdCard, sign, sneaker-net it back. The Mk4 adds NFC for faster air-gapped communication. The ColdCard Q adds a QWERTY keyboard and QR scanner.

**Jade** (Blockstream) makes a radical design choice: no secure element at all. Instead, it uses a "virtual secure element" — the key is split between the device and Blockstream's server (or your own blind oracle server). Neither party alone has the full key. This means a physical attacker who steals your Jade can't extract your key (they'd also need the server PIN). The tradeoff: you need network access to decrypt your key for signing (though the signing itself can be air-gapped via QR). Fully open source. The cheapest entry point for air-gapped Bitcoin storage.

**SeedSigner** is the cypherpunk's dream: you build it yourself from a Raspberry Pi Zero, a camera module, and a small screen (total cost ~$50–100 in parts). It runs Bitcoin-only signing software. It is entirely **stateless** — it doesn't store your keys at all. Each time you use it, you scan your seed (as a QR code from your backup) or enter it manually, sign the PSBT, and the device forgets everything when powered off. There is nothing to steal if someone takes your SeedSigner, because it contains no keys.

**BitKey** (Block Inc., Jack Dorsey's company) is a consumer-grade device that integrates a mobile app, a hardware device, and a server into a 2-of-3 multisig setup. You hold two keys (phone + hardware device), Block holds one. Any two of three can spend. If you lose the hardware device, you and Block can recover. If Block disappears, you have two keys locally. This is the most accessible multisig experience but requires trusting Block's infrastructure for key management.

### Which Hardware Wallet for Whom?

- **"I want maximum paranoia and I'm Bitcoin-only"** → ColdCard Mk4/Q
- **"I want open source and I'll build it myself"** → SeedSigner
- **"I want open source but not DIY, and I'll accept the oracle model"** → Jade
- **"I want the most auditable commercial device"** → Trezor Safe 5
- **"I want maximum altcoin support and don't mind closed firmware"** → Ledger
- **"I want easy multisig with no technical knowledge"** → BitKey
- **"I'm a developer who wants full control"** → SeedSigner + Sparrow

---

## Part VI: Multisig — Eliminating Single Points of Failure

### Why Multisig

Single-signature wallets have a fundamental problem: one key controls everything. If that key is compromised, all funds are gone. If the key is lost, all funds are inaccessible. There's no middle ground.

**Multisig** (multi-signature) solves this by requiring M of N keys to authorize a transaction. The most common setups:

- **2-of-3** — Three keys exist; any two can spend. Lose one key, you can still access funds. One key stolen, attacker can't spend.
- **3-of-5** — Five keys exist; any three can spend. Higher redundancy but more operational complexity.

### How Multisig Works On-Chain

Multisig is implemented using Bitcoin Script (see [Chapter 2](02-script-system.md)). The specific implementation depends on the address type:

**P2SH Multisig (Legacy):**
```
# The redeem script contains all public keys and the M-of-N requirement
OP_2 <pubkey1> <pubkey2> <pubkey3> OP_3 OP_CHECKMULTISIG
# Wrapped in P2SH → 3... address
```

**P2WSH Multisig (Native SegWit):**
```
# Same logic, but in the witness script
# The witness contains: OP_0 <sig1> <sig2> <witness_script>
# bc1q... address (longer than single-sig bc1q due to 32-byte script hash)
```

**P2TR Multisig (Taproot — via MuSig2 or Script Path):**
```
# Option 1: MuSig2 key aggregation (key-path spend)
# All N signers collaborate to produce a single aggregated signature
# On-chain: looks identical to a single-sig spend (maximum privacy)

# Option 2: Taproot script-path
# Multisig logic in a MAST leaf; revealed only when spent
```

MuSig2 (BIP 327) is the most elegant approach: the N parties collaboratively produce a single public key and single signature. Nobody looking at the blockchain can tell it was a multisig. But it requires all N parties to be online and coordinate signing — it's N-of-N by default. For M-of-N (like 2-of-3), you'd use Taproot's script-path with multiple MAST leaves, each containing a different 2-of-3 combination.

```bash
# Create a multisig descriptor wallet in Bitcoin Core
bitcoin-cli createwallet "multisig_test" false true "" false true

# Add a multisig descriptor (2-of-3 with three xpubs)
bitcoin-cli -rpcwallet=multisig_test importdescriptors '[{
  "desc": "wsh(sortedmulti(2,[fp1/84h/0h/0h]xpub1/*,[fp2/84h/0h/0h]xpub2/*,[fp3/84h/0h/0h]xpub3/*))#checksum",
  "timestamp": "now",
  "active": true,
  "range": [0, 100]
}]'

# Derive a multisig address
bitcoin-cli -rpcwallet=multisig_test getnewaddress "" "bech32"
```

### Managed Multisig Services

Setting up multisig correctly is hard. Key distribution, backup coordination, and transaction signing all become more complex. Two companies have built businesses around making multisig accessible:

**Unchained (formerly Unchained Capital):**
- 2-of-3 multisig (you hold 2 keys, Unchained holds 1)
- Hardware wallet agnostic (ColdCard, Trezor, Ledger)
- "Collaborative custody" — Unchained can co-sign but can never spend alone
- Inheritance planning: designated heir receives key access on verified death
- IRA product for Bitcoin in retirement accounts
- Unchained acts as a signing coordinator, not a custodian

**Casa:**
- 2-of-3 (basic) and 3-of-5 (premium) multisig
- Mobile key + hardware key(s) + Casa recovery key
- Health check system (periodically verifies you can still access your keys)
- Inheritance protocol with designated contacts
- More consumer-friendly than Unchained; less technical control

**The key principle:** in both models, the service provider holds ONE key in your multisig. They can facilitate recovery but cannot unilaterally spend your Bitcoin. Even if Unchained or Casa is compromised, seized, or goes bankrupt, the attacker gets at most one key — insufficient to spend.

This is qualitatively different from exchange custody (Coinbase, Kraken), where the exchange holds ALL the keys and you hold a database entry.

---

## Part VII: The Passphrase (25th Word)

BIP 39 includes an optional passphrase (sometimes called the "25th word," though it can be any string, not just a single word). When deriving the seed from the mnemonic, the passphrase is appended to the salt: `PBKDF2(mnemonic, "mnemonic" + passphrase)`.

### What the Passphrase Does

A different passphrase produces a completely different seed — and therefore completely different keys and addresses. There is no "wrong" passphrase; every passphrase produces a valid wallet. The "correct" passphrase is simply the one that derives the wallet containing your Bitcoin.

```
24-word mnemonic + no passphrase  → Wallet A (maybe empty, maybe decoy funds)
24-word mnemonic + "hunter2"      → Wallet B (your actual holdings)
24-word mnemonic + "anything"     → Wallet C (valid but empty)
```

### Use Cases

**Plausible deniability:** If forced to reveal your seed phrase (by a thief, by a government, by a wrench), you can disclose the mnemonic without the passphrase. The attacker sees Wallet A, which might contain a small decoy amount. Your real funds in Wallet B remain hidden because the attacker doesn't know a passphrase exists — let alone what it is.

**Additional security layer:** Even if your 24-word backup is stolen, the attacker can't access funds without the passphrase. This turns single-factor security (something you have — the seed) into two-factor (something you have + something you know).

### Risks

**If you forget the passphrase, your Bitcoin is gone.** There is no recovery. The wallet derived with the correct passphrase is the only wallet your funds exist in. A passphrase of `"hunter2"` and `"Hunter2"` produce different wallets. A trailing space produces a different wallet. Case sensitivity, whitespace, and Unicode normalization all matter.

**Passphrase backups create their own security problem.** If you write the passphrase on the same steel plate as the mnemonic, you've eliminated the benefit. If you memorize it, you've introduced the brain wallet failure mode — human memory is unreliable. The recommended approach: store the passphrase separately from the mnemonic, in a different geographic location, with clear documentation of its purpose.

**Testing is essential.** Before sending any Bitcoin to a passphrase-protected wallet:
1. Set up the wallet with your mnemonic + passphrase
2. Generate a receive address and note it
3. Wipe the device
4. Restore from mnemonic + passphrase
5. Verify the same receive address is generated
6. Only then fund the wallet

---

## Part VIII: Shamir Secret Sharing (SLIP 39)

### What It Is

Shamir Secret Sharing (SSS) is a cryptographic technique that splits a secret into N shares, of which any M are sufficient to reconstruct the original. Trezor implemented this as **SLIP 39**, which replaces BIP 39 mnemonic words with multiple sets of 20 or 33 words (shares).

Example: a 2-of-3 SLIP 39 setup generates three share groups. Any two groups can reconstruct the seed. Each group is a set of words (like BIP 39, but from a different word list).

### Shamir vs. Multisig

These are fundamentally different tools that solve different problems:

| Property | Shamir (SLIP 39) | Multisig |
|----------|:---:|:---:|
| Splits the secret | Yes — at the seed level | No — multiple independent keys |
| Reconstruction required | Yes — must combine shares to sign | No — each key signs independently |
| Single point of failure during signing | Yes — reconstructed seed exists in memory | No — keys never combine |
| On-chain footprint | Single-sig | Multi-sig (or aggregated via MuSig2) |
| Supported wallets | Trezor, limited others | Broad support (Sparrow, Electrum, Core, etc.) |

**When to use Shamir:** backup redundancy. You want geographic distribution of your backup material so that losing one location doesn't destroy your ability to recover. But you're comfortable with single-sig spending.

**When to use multisig:** operational security. You want to ensure that no single device compromise, theft, or coercion can lead to loss of funds. Each key is an independent signing device — they never need to be in the same room.

**The critical difference:** with Shamir, when you reconstruct the seed to sign a transaction, the full seed exists in one device's memory at that moment. With multisig, no single device ever holds enough information to spend. For high-value storage, multisig is strictly superior.

### When Shamir Makes Sense

- You're using single-sig (hardware wallet) and want robust backup distribution
- You want to give shares to family members for inheritance without any one person being able to access funds alone
- You're supplementing multisig (backing up each individual multisig key with Shamir shares)

---

## Part IX: Seed Storage — Physical Security

Your seed phrase is the master key to everything. Its physical storage is the most important operational security decision you'll make.

### Steel Backup Options

Paper degrades. Lamination helps but doesn't survive fire. Steel plates withstand both:

| Product | Method | Fire Rating | Crush Resistant | Price |
|---------|--------|:---:|:---:|-------|
| **Cryptosteel Capsule** | Letter tiles in steel tube | 1400°C / 2500°F | Yes | $90 |
| **Billfodl** | Letter tiles in steel frame | 1100°C / 2000°F | Yes | $65 |
| **SeedPlate** | Center-punch dots on steel | 1500°C+ | Yes | $40 |
| **Blockplate** | Center-punch on steel grid | 1500°C+ | Yes | $50 |
| **DIY washers** | Letter stamps on steel washers | Varies | Yes | $10–20 |

**Recommendation:** center-punch or stamp methods (SeedPlate, Blockplate, DIY washers) are more durable than letter tiles because there are no moving parts that can scatter. Jameson Lopp has published extensive stress tests of seed storage devices — fire, crushing, corrosion, gunshots — and the results strongly favor solid-piece designs.

BIP 39's four-letter prefix property means you only need to stamp the first four characters of each word. `abandon` → `ABAN`. This is unambiguous because no two BIP 39 words share their first four letters.

### Geographic Distribution

A seed stored in one location is vulnerable to that location's risks: fire, flood, burglary, government seizure.

The standard recommendation for significant holdings:

1. **Primary backup** — Home safe or safety deposit box
2. **Secondary backup** — Different geographic location (family member's safe, second safety deposit box in a different bank, attorney's vault)
3. **Tertiary backup** (for multisig) — Third location, ideally in a different city or state

For multisig, each key's backup should be in a different location. A 2-of-3 multisig with all three keys in the same safe offers zero improvement over single-sig.

### Inheritance Planning

Bitcoin inheritance is an unsolved problem at the protocol level. There is no beneficiary designation, no probate process, no account recovery. If you die without your heirs having access to your keys, the Bitcoin is permanently lost.

**Approaches:**

**Letter of instruction:** A sealed document stored with your will or estate attorney. It describes what Bitcoin is, where your hardware wallets are, where your seed backups are, and step-by-step recovery instructions. This is the minimum viable inheritance plan.

**Unchained/Casa inheritance protocols:** Both services offer heir designation. When your death is verified through a defined process, the service co-signs a transaction to transfer funds to your heir's address.

**Timelock-based dead man's switch:** Using `OP_CHECKLOCKTIMEVERIFY` (CLTV), you can create a transaction that becomes valid after a specific block height or date. You periodically refresh the timelock by moving funds to a new CLTV address with a later expiry. If you stop refreshing (because you've died or are incapacitated), the timelock eventually expires and your heir can claim the funds.

```bash
# Conceptual example of a timelock script for inheritance
# This is the spending condition, not a complete transaction
# After block 900000, heir_pubkey can spend; before that, only owner_pubkey can
#
# Script: OP_IF <owner_pubkey> OP_CHECKSIG
#         OP_ELSE <900000> OP_CHECKLOCKTIMEVERIFY OP_DROP <heir_pubkey> OP_CHECKSIG
#         OP_ENDIF
```

**Multisig with a trusted party:** In a 2-of-3 setup, give one key to your heir and one to a trusted service (attorney, Unchained, Casa). In normal operation, you use your two keys. After your death, your heir + the trusted service together control the funds.

**The uncomfortable truth:** every inheritance plan requires trusting someone — either a person, an institution, or a future version of yourself who keeps refreshing a timelock. Bitcoin eliminates counterparty risk for custody but reintroduces it for inheritance. Plan accordingly.

---

## Part X: PSBTs (Partially Signed Bitcoin Transactions)

### The Problem PSBTs Solve

Before BIP 174 (authored by Ava Chow), there was no standard way for multiple devices to collaboratively construct and sign a transaction. Each wallet had its own format. Hardware wallets had proprietary protocols. Multisig coordination was painful.

PSBTs (Partially Signed Bitcoin Transactions) define a universal container format for Bitcoin transactions that aren't yet fully signed. A PSBT can be passed between devices — each one adding its signature — until the transaction has enough signatures to be valid.

### PSBT Workflow

```
Step 1: CREATE (online wallet — Sparrow, Electrum, Bitcoin Core)
        Constructs the unsigned transaction with inputs, outputs, fee
        Encodes as PSBT (base64 string or binary file)
            │
            ▼
Step 2: TRANSFER (air-gap crossing)
        QR code, MicroSD card, USB, or file transfer
            │
            ▼
Step 3: SIGN (hardware wallet — ColdCard, Trezor, Jade, SeedSigner)
        Device displays transaction details for verification
        User confirms; device adds its signature to the PSBT
            │
            ▼
Step 4: (If multisig) TRANSFER to next signer → repeat Step 3
            │
            ▼
Step 5: FINALIZE (any PSBT-compatible wallet)
        Combines all signatures into a complete transaction
            │
            ▼
Step 6: BROADCAST (online wallet or bitcoin-cli)
        Sends the finalized transaction to the network
```

```bash
# PSBT workflow in Bitcoin Core

# Create a PSBT (unsigned transaction)
bitcoin-cli -rpcwallet=my_wallet walletcreatefundedpsbt \
  '[]' \
  '[{"bc1q...recipient_address": 0.01}]' \
  0 \
  '{"changeAddress": "bc1q...change_address"}'
# Returns: { "psbt": "cHNidP8BAH0CAAA...", "fee": 0.00000141, "changepos": 1 }

# Process the PSBT (adds UTXO information for signing)
bitcoin-cli walletprocesspsbt "cHNidP8BAH0CAAA..."

# Analyze a PSBT (check what signatures are present/missing)
bitcoin-cli analyzepsbt "cHNidP8BAH0CAAA..."
# Shows: which inputs need signatures, estimated fee, etc.

# Finalize (combine all signatures into valid transaction)
bitcoin-cli finalizepsbt "cHNidP8BAH0CAAA..."
# Returns: { "hex": "0200000001...", "complete": true }

# Broadcast the finalized transaction
bitcoin-cli sendrawtransaction "0200000001..."
```

### PSBT Version 2 (BIP 370)

BIP 174 (PSBT v0) has a limitation: the transaction must be fully constructed before any signing begins. BIP 370 (PSBT v2, also by Ava Chow) allows inputs and outputs to be added incrementally. This is critical for:

- **CoinJoin** — Multiple parties each add their inputs and outputs to the same transaction
- **PayJoin** — Sender and receiver both contribute inputs
- **Interactive protocols** — Any workflow where the transaction is built collaboratively

### Why PSBTs Matter

PSBTs are the glue that makes modern self-custody work. Without them:
- Air-gapped signing wouldn't have a standard format
- Hardware wallets from different manufacturers couldn't interoperate
- Multisig coordination would require each wallet to understand every other wallet's format
- CoinJoin implementations would each need custom integration

Sparrow Wallet's entire UX is built around PSBTs. When you create a transaction in Sparrow with a connected hardware wallet, what's actually happening under the hood is: Sparrow creates a PSBT → sends it to the hardware wallet → the hardware wallet signs and returns the PSBT → Sparrow finalizes and broadcasts.

---

## Part XI: Common Mistakes and Attack Vectors

### Address Reuse (Privacy Leak)

Every time you reuse an address, you link all transactions involving that address. An observer can track your balance and spending patterns. HD wallets generate a new address for each receive — use that feature. Never manually copy-paste an address you've used before.

Bitcoin Core discourages address reuse by default:

```bash
# Generate a fresh address each time — never reuse
bitcoin-cli -rpcwallet=my_wallet getnewaddress "" "bech32"
```

### Clipboard Malware

A category of malware specifically targeting cryptocurrency users. It monitors the clipboard and, when it detects a Bitcoin address (pattern matching on the prefix and length), silently replaces it with an attacker's address. You paste what you think is the recipient's address; you actually paste the attacker's.

**Defense:** Always visually verify the full address on your hardware wallet's screen before confirming a transaction. This is exactly why hardware wallets have screens — so you can verify the destination address on a device the malware can't compromise.

### Fake Hardware Wallets (Supply Chain Attacks)

Tampered hardware wallets have been documented. Attack vectors:
- **Modified firmware** — Device looks genuine but generates keys the attacker also knows
- **Pre-loaded seed phrases** — Device arrives with a seed phrase card, claiming it's "your" seed. It's actually the attacker's seed. You load Bitcoin; they drain it.
- **Counterfeit devices** — Physical clones with malicious hardware

**Defense:**
1. Buy directly from the manufacturer. Never from Amazon third-party sellers, eBay, or "deals" on forums.
2. Verify the anti-tamper packaging is intact.
3. The device should generate your seed phrase fresh. If it arrives with a pre-filled seed, it's compromised.
4. Verify firmware signatures before first use.
5. ColdCard has a "bag number" system — each device's sealed bag has a unique number you can verify on Coinkite's website.

### SIM Swap Attacks

Attacker convinces your mobile carrier to port your phone number to their SIM. They then use SMS-based 2FA to access your exchange accounts. This doesn't directly affect hardware wallet self-custody, but it's devastating for anyone using:
- SMS 2FA on exchanges
- Phone-number-based recovery on any Bitcoin-related service

**Defense:** Use hardware-based 2FA (YubiKey) or TOTP (Google Authenticator, Authy) — never SMS. For mobile carrier security, add a PIN/passphrase to your account. Consider Google Fi or carriers that support number-lock features.

### The $5 Wrench Attack

The most effective attack on Bitcoin self-custody requires no technical sophistication: someone threatens you with physical violence until you hand over your keys.

**Mitigations:**
- Don't talk about your Bitcoin holdings publicly
- Passphrase wallets for plausible deniability (see Part VII)
- Multisig with geographically distributed keys (attacker can't force you to access keys you physically can't reach)
- Timelock transactions that prevent immediate spending
- Duress wallets: a small amount in an easily-accessible wallet you can surrender

### Phishing and Social Engineering

The most common attack vector is not technical — it's human. Common patterns:
- "Support agent" on Discord/Telegram asks for your seed phrase (no legitimate service will ever ask for this)
- Fake wallet apps in app stores
- Clone websites with slightly different URLs (e.g., `electrurn.org` instead of `electrum.org`)
- "Airdrop" transactions that send dust to your address with a memo pointing to a phishing site

**The rule is simple: NEVER enter your seed phrase into any computer, website, app, or form. The only places your seed should ever exist are: (1) your hardware wallet, (2) your physical backup (steel/paper), and (3) during initial setup/recovery on the hardware wallet's own screen.**

---

## Part XII: Self-Custody Philosophy

### "Not Your Keys, Not Your Coins"

This phrase, attributed to Andreas Antonopoulos, captures the fundamental reality of Bitcoin: if someone else holds your private keys, you don't own Bitcoin. You own an IOU from the entity holding your keys. Whether that entity is an exchange, a bank, a fund, or a government — you have a claim, not a possession.

Satoshi designed Bitcoin to eliminate trusted third parties. The entire point of proof-of-work, of the UTXO model, of the scripting system, is to create money that can be held and transferred without anyone's permission. Using an exchange as a permanent wallet reverses every property that makes Bitcoin different from a bank balance.

### The Graveyard of Custodians

History has demonstrated this lesson with extraordinary consistency:

**Mt. Gox (2014):** The dominant Bitcoin exchange, handling 70% of all Bitcoin transactions at its peak. Lost 850,000 BTC (recovered ~200,000). Users waited TEN YEARS for partial recovery through Japanese bankruptcy proceedings. As of 2024, creditors received approximately 15% of their holdings, denominated in Bitcoin at 2014 prices.

**QuadrigaCX (2019):** Canadian exchange. Founder Gerald Cotten died (or claimed to die) with the only keys to $190 million in customer funds. An investigation revealed the funds had likely been misappropriated long before his death. Users recovered almost nothing.

**Celsius Network (2022):** Crypto lending platform froze withdrawals in June 2022 with $4.7 billion in customer assets. Filed bankruptcy the next month. Customers classified as unsecured creditors — behind lawyers and executives in the repayment queue.

**FTX (2022):** The second-largest cryptocurrency exchange. Customer funds were secretly loaned to Alameda Research (FTX's sister trading firm) and lost. $8+ billion in customer deposits vanished. Founder Sam Bankman-Fried convicted of fraud and sentenced to 25 years. Customers in bankruptcy proceedings as of 2024.

**BlockFi (2022):** Crypto lending platform. Filed bankruptcy in November 2022 after FTX collapse. Customer funds locked for years. Partial recovery through bankruptcy.

The pattern is identical every time:
1. Custodian appears legitimate and trustworthy
2. Custodian takes risks with customer funds (rehypothecation, lending, fraud, or simply poor security)
3. Custodian fails
4. Customers discover they have no Bitcoin — they have a bankruptcy claim

**Every one of these failures was preventable with self-custody.** Not theoretically. Literally. If those users had held their own keys, no management decision, fraud, hack, or death could have touched their Bitcoin.

### The Spectrum of Trust

Self-custody isn't binary. There's a spectrum:

```
Full custodial    ──────────────────────────────────────>    Full self-custody
(exchange holds   (exchange + your    (managed         (single-sig    (multisig,
 all keys)         2FA/withdrawal     multisig —       hardware       own keys,
                   whitelist)         Unchained/Casa)  wallet)        geographic
                                                                      distribution)
    ▲                    ▲                   ▲              ▲              ▲
    │                    │                   │              │              │
  LEAST              MODERATE           GOOD           STRONG        MAXIMUM
  SECURE             SECURITY           SECURITY       SECURITY      SECURITY
```

Move as far right as your technical comfort and operational discipline allow. The minimum for anyone holding more than pocket change: a hardware wallet with a properly backed-up seed phrase.

### The Real Risk Calculus

Self-custody skeptics argue: "Most people will lose their keys. An exchange is safer."

This is like arguing: "Most people will crash their car. Let someone else drive." It's true for some people. But the response isn't to give up driving — it's to learn how to drive.

The realistic threats, ranked by probability of actual Bitcoin loss:

1. **Exchange insolvency/fraud** — Has happened repeatedly, affecting millions of users
2. **Lost seed phrases / forgotten passphrases** — Preventable with proper backup
3. **Phishing / social engineering** — Preventable with education and discipline
4. **Physical theft** — Mitigated by multisig and passphrase wallets
5. **Hardware wallet vulnerability** — Extremely rare; no mass-exploit has occurred
6. **Sophisticated targeted attack** — Only relevant if you're publicly known to hold large amounts

For most people, the exchange risk (item 1) is orders of magnitude higher than the self-custody risks (items 2–6) combined. Self-custody is harder, but the failure modes are all within your control. Exchange custody outsources both the security AND the risk to someone whose incentives may not align with yours.

---

## Part XIII: Practical Self-Custody Checklist

For someone setting up self-custody for the first time:

**Phase 1: Basic (1 hardware wallet)**
- [ ] Buy a hardware wallet directly from the manufacturer
- [ ] Initialize it and write down the 24-word seed phrase
- [ ] Stamp or punch the seed onto a steel backup
- [ ] Test recovery: wipe the device, restore from seed, verify same addresses
- [ ] Send a small test transaction, verify receipt, send back
- [ ] Store the steel backup somewhere physically secure (not the same room as the hardware wallet)

**Phase 2: Robust (passphrase + geographic distribution)**
- [ ] Add a passphrase to create a hidden wallet
- [ ] Document the passphrase separately from the seed
- [ ] Create a second steel backup of the seed; store in a different location
- [ ] Write a letter of instruction for heirs (what it is, where the backups are, how to recover)

**Phase 3: Paranoid (multisig)**
- [ ] Set up 2-of-3 multisig using Sparrow, Nunchuk, or a managed service
- [ ] Use three different hardware wallets from different manufacturers
- [ ] Store each key's backup in a different geographic location
- [ ] Test the full signing workflow: create PSBT → sign on device 1 → sign on device 2 → broadcast
- [ ] Document the multisig configuration (xpubs, derivation paths, quorum) — this is needed for recovery

```bash
# Verify your wallet setup in Bitcoin Core
# Check that your descriptors are correctly imported
bitcoin-cli -rpcwallet=my_wallet getwalletinfo

# List all descriptors and their derivation paths
bitcoin-cli -rpcwallet=my_wallet listdescriptors true

# Verify a specific address belongs to your wallet
bitcoin-cli -rpcwallet=my_wallet getaddressinfo "bc1q..."
```

---

## Key Takeaways

1. **A wallet holds keys, not Bitcoin.** UTXOs live on the blockchain. Keys are what let you spend them. Lose the keys, lose the Bitcoin.

2. **BIP 39 turns entropy into words; BIP 32 turns words into a tree of keys.** Your 12 or 24 words back up an unlimited number of addresses across all four address types (P2PKH, P2SH-SegWit, P2WPKH, P2TR).

3. **Native SegWit (bc1q) is the current default; Taproot (bc1p) is the future.** Both are supported by all major wallets. Legacy addresses cost more in fees and should not be used for new wallets.

4. **Hardware wallets are the baseline for serious self-custody.** ColdCard for maximum paranoia. Trezor for open-source purity. Jade for cheap air-gapping. SeedSigner for stateless DIY. Ledger for altcoin support (with the trust tradeoff).

5. **Multisig eliminates single points of failure.** 2-of-3 is the most common setup. No single device compromise, theft, or coercion can result in loss. Unchained and Casa make multisig accessible.

6. **The passphrase (25th word) adds plausible deniability and a second factor** — but if you forget it, your funds are irretrievable. Test recovery before funding.

7. **Shamir (SLIP 39) splits your backup; multisig splits your signing authority.** They solve different problems. For high-value holdings, multisig is strictly superior because no single device ever holds the complete signing capability.

8. **PSBTs (BIP 174) are the universal standard** for multi-device transaction signing. They enable air-gapped workflows, hardware wallet interoperability, and multisig coordination.

9. **Steel backups, geographic distribution, and a letter of instruction** are the minimum for responsible seed storage. Your backup strategy must survive fire, flood, theft, and your own death.

10. **Self-custody is not optional if you take Satoshi's design seriously.** Mt. Gox, QuadrigaCX, Celsius, FTX — the pattern is always the same. The only Bitcoin you truly own is Bitcoin you hold the keys to. Everything else is a promise.

---

**Previous:** [Chapter 8 — Satoshi's Timeline and Bitcoin Core History](08-satoshi-timeline.md) — Who built Bitcoin, who maintains it now, and who actually controls it.

**Next:** [Chapter 10 — Mining & Hardware](10-mining-hardware.md) — From Satoshi's CPU to 1,160 TH/s hydro-cooled ASICs: the mining arms race and its economics.

**Related:** [Chapter 2 — The Script System](02-script-system.md) — How P2PKH, P2SH, P2WSH, and P2TR scripts work at the opcode level.
