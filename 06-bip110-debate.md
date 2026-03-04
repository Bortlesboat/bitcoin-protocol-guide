# Chapter 6: The BIP-110 Debate

## Bitcoin's "Spam War" — Both Sides

BIP-110 (also called BIP-444 / Reduced Data Temporary Softfork) is the most contentious proposal in Bitcoin since the block size wars. It proposes to limit arbitrary data in transactions, directly targeting Ordinals/Inscriptions. Understanding both sides requires everything from the previous chapters.

## What Satoshi Said

Both sides of this debate claim Satoshi's vision. Here are his actual words:

**Pro-restriction camp cites:**
> "Messages should not be recorded in the block chain."
> — [BitcoinTalk](https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/239/) (October 23, 2010)

**Anti-restriction camp cites:**
> "The incentive can transition entirely to transaction fees and be completely inflation free."
> — White paper, [Section 6](https://bitcoin.org/bitcoin.pdf)

And on Bitcoin's resilience:
> "The network is robust in its unstructured simplicity... Any needed rules and incentives can be enforced with this consensus mechanism."
> — White paper, [Section 12: Conclusion](https://bitcoin.org/bitcoin.pdf)

The tension is real: Satoshi opposed non-financial data, but also designed a permissionless fee market as Bitcoin's long-term security model.

## What BIP-110 Does

**Author:** Dathon Ohm, backed by Luke Dashjr (Bitcoin Knots maintainer)
**Proposed:** December 2025
**Status:** Draft, enforced by Bitcoin Knots nodes

### Specific restrictions:

| Rule | Current limit | BIP-110 limit | Effect |
|------|--------------|---------------|--------|
| Output data size | No limit | 34 bytes | Kills non-standard outputs |
| OP_RETURN data | 80 bytes (Core default) | 83 bytes | Slightly more permissive than pre-v30 Core |
| Data push size | 520 bytes | 256 bytes | Breaks inscription chunks |
| Witness element size | No limit | 256 bytes | Kills large inscriptions |
| Witness versions | v0, v1 active | Restricts future versions | Prevents new witness tricks |

### Key detail: Temporary

BIP-110 is designed as a **1-year soft fork**. After expiration, the restrictions lift unless renewed. This is unusual — most soft forks are permanent.

### Activation mechanism

Requires **55% hashrate signaling** over a defined period. As of early 2026, support is around **3% of nodes** (mostly Knots) and one mining pool (Ocean).

## The "Anti-Spam" Case (Pro BIP-110)

### 1. Block space is for financial transactions

> "Bitcoin was designed for peer-to-peer electronic cash. Using block space for JPEGs and meme tokens is a misuse of a scarce resource."

The argument: Bitcoin's value proposition is censorship-resistant money. Every byte used for inscriptions is a byte not available for payments. When blocks are full, inscriptions compete with financial transactions, driving up fees for everyone.

### 2. The witness discount was not meant for data storage

SegWit's 4:1 discount was designed to make *signatures* cheap, not to subsidize on-chain data storage. Inscriptions exploit an unintended consequence.

```
Design intent:     Actual use:
Cheaper sigs  →    Cheap data storage
More TXs/block →   Fewer TXs/block (data fills witness)
```

### 3. UTXO set bloat

While inscriptions themselves don't bloat the UTXO set (they're in witness data), the *ecosystem around them* creates dust UTXOs. BRC-20 in particular creates many small UTXOs that every node must track.

### 4. Precedent for "anything goes"

If arbitrary data is acceptable, what's next? Full movies? Malware? CSAM? The argument is that *some* content policy is necessary, and limiting data size is a neutral way to do it.

### 5. Luke Dashjr's position

Luke (Bitcoin Knots lead) has consistently argued that non-financial data in transactions is spam. He views BIP-110 as restoring Bitcoin's original purpose and fixing a policy mistake (the removal of OP_RETURN limits in Bitcoin Core v30).

## The "Free Market" Case (Anti BIP-110)

### 1. Paid-for block space is not spam

> "If someone pays the market fee rate, they have as much right to block space as anyone else. Bitcoin is permissionless."

The argument: Bitcoin's fee market is the mechanism for allocating block space. Inscriptions pay fees — often *very high* fees. Calling paid transactions "spam" is a value judgment that contradicts Bitcoin's permissionless nature.

### 2. Filters don't work — they make things worse

Historical precedent from Bitcoin Core's OP_RETURN limits:

| Era | Policy | What happened |
|-----|--------|---------------|
| Pre-2014 | No OP_RETURN | Data encoded as fake addresses (UTXO bloat) |
| 2014+ | 40-byte OP_RETURN | Data moved to OP_RETURN (cleaner) |
| 2023+ | Inscriptions | Data in witness (cheapest, doesn't bloat UTXO set) |

Every restriction pushes data embedding into *worse* methods. If you ban witness data, inscriptions will move to multisig scripts, fake outputs, or other creative workarounds — all of which are MORE harmful to the network.

### 3. Miner revenue matters

Inscriptions generate significant fee revenue for miners. Post-halving, the block subsidy keeps decreasing. Bitcoin's long-term security depends on fee revenue replacing the subsidy. Banning a major fee source is counterproductive.

```
Block subsidy schedule:
2024: 3.125 BTC/block → ~$312K/block at $100K
2028: 1.5625 BTC/block → needs fees to maintain security
2032: 0.78125 BTC/block → fees become dominant
```

### 4. Soft fork risk

Any contentious soft fork risks a chain split. If BIP-110 activates with 55% hashrate but significant economic nodes reject it, you get:
- Chain A: BIP-110 blocks (restrictive)
- Chain B: Non-BIP-110 blocks (permissive)
- Confusion, potential value loss on both chains

Adam Back (Blockstream CEO, HashCash inventor) has explicitly warned that BIP-110 risks damaging Bitcoin's credibility.

### 5. The slippery slope

Who decides what's "legitimate" use of block space? Today it's inscriptions. Tomorrow it might be privacy transactions (CoinJoin), time-locked contracts, or specific smart contract patterns. Establishing the precedent that Bitcoin transactions can be censored by content opens a dangerous door.

### 6. Jameson Lopp's analysis

Lopp (Casa CTO, long-time Bitcoin developer) called BIP-110 "misguided," arguing:
- Technical restrictions are trivially circumvented
- The proposal conflates policy preferences with protocol rules
- Bitcoin's value comes from being *neutral infrastructure*

## The Nuanced Middle

Some thoughtful positions that don't fit neatly into either camp:

### "Fix the discount, not the data"

Instead of restricting data, adjust the witness discount. If witness data cost 2 WU/byte instead of 1, inscriptions would cost 2× more. This preserves permissionlessness while reducing the subsidy for non-signature witness data.

### "Separate the concerns"

- **Consensus level**: Allow anything that pays fees
- **Relay policy**: Individual nodes can choose what to relay (already possible — miners can filter their own mempools)
- **Application level**: Build better fee estimation and block space markets

### "Wait for the market"

Inscription activity naturally ebbs and flows with market cycles. During quiet periods, blocks have plenty of room. The "crisis" may be self-correcting.

## Current State (Early 2026)

- Bitcoin Core v30 (Oct 2025) removed the 83-byte OP_RETURN limit
- Bitcoin Knots enforces BIP-110 restrictions
- Ocean pool (backed by Jack Dorsey) mines some BIP-110-compliant blocks
- ~3% of network nodes enforce BIP-110
- The 55% hashrate threshold has not been met
- The debate continues

## The Deeper Question

The BIP-110 debate is really about Bitcoin's identity:

**Is Bitcoin neutral infrastructure** (like TCP/IP — it doesn't care what data you send) **or is it purpose-built money** (like the Federal Reserve wire system — only for financial transactions)?

Your answer to this question determines your position on BIP-110, and probably every future debate about Bitcoin's scope.

## Key Takeaways

1. BIP-110 proposes temporary (1-year) restrictions on data sizes in transactions and witness
2. Pro: Block space should be for payments, witness discount wasn't meant for data, UTXO bloat concerns
3. Anti: Paid fees = legitimate use, restrictions don't work (data finds other paths), miner revenue, chain split risk
4. The real question: Is Bitcoin neutral infrastructure or purpose-built money?
5. Currently ~3% of nodes, far from the 55% hashrate threshold needed

---

**Next:** [Chapter 7 — Node Setup](07-node-setup.md) — Run your own node and verify everything yourself.
