# Chapter 11: The Bitcoin Network

## In Satoshi's Words

> "The design supports letting users just be users. The more burden it is to run a node, the fewer nodes there will be. Those few nodes will be big server farms."
>
> — Satoshi Nakamoto, BitcoinTalk, July 29, 2010

That prediction is half right and half wrong — and the tension between the two halves defines everything about Bitcoin's peer-to-peer layer.

Satoshi was right that running a node would become harder over time. The blockchain is over 600 GB in 2026. Initial block download takes hours to days on consumer hardware. Bandwidth requirements are nontrivial. Most Bitcoin users don't run nodes — they use wallets that connect to someone else's infrastructure.

But Satoshi was wrong about the "server farm" endgame. As of early 2026, roughly 20,000 reachable full nodes operate globally, and an estimated 60,000+ total nodes exist when you count Tor, I2P, and non-listening nodes behind NATs. Many of these run on consumer hardware — Raspberry Pis, old laptops, $200 mini PCs. The community has fought hard — through pruning, compact blocks, assumeUTXO, and relentless optimization — to keep node operation within reach of ordinary people.

This chapter covers the layer that makes all of that possible: the peer-to-peer network. It's the layer most people skip, but it's the layer that makes everything else — consensus, mining, transaction relay, chain validation — actually work in a decentralized system with no central server.

---

## Part I: Node Types

Not all nodes are created equal. The Bitcoin network is a heterogeneous mix of nodes with different capabilities, different storage requirements, and different trust models. Understanding what each type can and cannot verify is fundamental.

### Full Node (Archival)

The gold standard. A full archival node:

- Downloads and stores **every block** since the genesis block (January 3, 2009)
- Validates **every transaction** in every block against the consensus rules
- Maintains the **complete UTXO set** (the set of all unspent transaction outputs)
- Can serve historical blocks to peers who request them
- Can independently verify any transaction in Bitcoin's history

**Storage requirement:** ~600+ GB as of early 2026, growing at roughly 50-80 GB per year.

What it can verify: **everything**. A full archival node trusts no one. It has independently validated every signature, every script, every block header, every difficulty adjustment, and every coinbase reward from block 0 to the present. If you run one, you have mathematically proven to yourself that the chain you're following is valid.

```bash
# Check your node's blockchain info
bitcoin-cli getblockchaininfo
# Look at: "blocks", "headers", "size_on_disk", "pruned"
```

### Pruned Node

A pruned node does everything a full node does — with one difference: after validating old blocks, it **discards** the raw block data to save disk space. It retains only:

- The complete UTXO set (required to validate new transactions)
- Recent blocks (configurable; default is 550 blocks, about 4 days)
- All block headers (80 bytes each — trivial storage)

**Storage requirement:** As low as ~7-10 GB with aggressive pruning settings.

**What it can verify:** Everything a full node can verify — it validates every block during initial sync. The catch is that after pruning, it **cannot serve old blocks** to peers. If someone asks for block 100,000, a pruned node can't deliver it. This limits the node's contribution to the network's data availability but doesn't limit its own security model.

```bash
# Enable pruning (in bitcoin.conf)
# prune=550   # Keep last 550 blocks (~550 MB minimum)

# Or start with pruning:
bitcoind -prune=1000  # Keep ~1 GB of recent blocks
```

The critical point: **pruned nodes are not less secure than archival nodes for the operator.** They validated everything. They just threw away the receipts afterward. The trust model is identical.

### SPV / Lightweight Node

SPV stands for **Simplified Payment Verification**, described by Satoshi in Section 8 of the whitepaper. An SPV node:

- Downloads only **block headers** (80 bytes per block — the entire header chain since genesis is ~60 MB)
- Verifies the proof-of-work chain (headers link together; difficulty adjustments are correct)
- Uses **Bloom filters** (BIP 37) or **compact block filters** (BIP 157/158) to check if specific transactions appear in blocks
- Does **not** validate transactions or scripts independently

**What it can verify:** That a transaction is buried under proof-of-work. That's it. An SPV node cannot verify that a transaction is valid according to consensus rules — only that miners included it in a block. It trusts miners to have validated correctly.

**What it cannot verify:**
- Whether a transaction's inputs actually existed and were unspent
- Whether the scripts were satisfied correctly
- Whether the block reward was correct
- Whether any inflation bugs were exploited

SPV is the trust model used by most mobile wallets (Electrum on mobile, BlueWallet in SPV mode, BRD). It's vastly better than trusting a centralized server, but it's not trustless. It's "trust miners and assume honest majority."

### Mining Node

A mining node is a full node (usually archival) plus:

- **Block template construction** — assembles candidate blocks from mempool transactions, ordered by fee rate
- **Stratum protocol support** — connects to mining pools or manages ASICs directly
- **getblocktemplate** — the RPC call that returns a candidate block for miners to hash against

```bash
# Get a block template (what a miner would work on)
bitcoin-cli getblocktemplate '{"rules": ["segwit"]}'
# Returns: transactions to include, previous block hash, target difficulty, coinbase value
```

Mining nodes must be full nodes because they need to validate the blocks they produce. A miner who builds on an invalid block wastes energy and risks their block being rejected by the network. The economic incentive to run a full node is strongest for miners — invalid blocks mean lost revenue.

In practice, most individual miners don't run their own full nodes anymore. They connect to mining pools (Foundry, Antpool, ViaBTC, F2Pool), which operate the full nodes and distribute work to miners via the Stratum protocol. The centralization risk here is real: a pool operator who validates incorrectly can cause all their connected miners to waste work.

### Archival Node with Transaction Index

A full archival node with `-txindex=1` enabled maintains an additional index that maps **every transaction ID** to its location on disk. Without txindex, a node can only look up transactions that are in the UTXO set (unspent) or in recent blocks. With txindex, you can query any transaction in Bitcoin's history by its txid.

```bash
# Requires txindex=1 in bitcoin.conf
# Look up any historical transaction by its ID
bitcoin-cli getrawtransaction f4184fc596403b9d638783cf57adfe4c75c605f6356fbc91338530e9831e9e16 2
# This is the first Bitcoin transaction — Satoshi to Hal Finney, block 170
```

**Storage overhead:** The txindex adds roughly 30-40 GB on top of the full blockchain. It's used by block explorers, analytics services, and anyone who needs to look up arbitrary historical transactions.

### Summary Table

| Node Type | Validates Everything | Stores Full Chain | Serves Old Blocks | Trust Model |
|-----------|---------------------|-------------------|-------------------|-------------|
| Full (archival) | Yes | Yes | Yes | Trustless |
| Pruned | Yes | No (discards old) | No | Trustless |
| SPV / Lightweight | No (headers only) | No | No | Trusts miners |
| Mining | Yes | Usually | Usually | Trustless |
| Archival + txindex | Yes | Yes + index | Yes | Trustless |

---

## Part II: Peer Discovery — The Bootstrap Problem

When a brand-new Bitcoin Core node starts for the first time, it faces a cold-start problem: it doesn't know any peers. It has no IP addresses, no connections, and no way to download blocks. How does it find the network?

### DNS Seeds

Bitcoin Core ships with a hardcoded list of **DNS seed nodes** — domain names that resolve to IP addresses of known, reliable Bitcoin nodes. These are maintained by long-standing community members.

When your node starts, it queries these DNS seeds:

```
seed.bitcoin.sipa.be          — Pieter Wuille
dnsseed.bluematt.me            — Matt Corallo
dnsseed.bitcoin.dashjr-list-of-p2sh-hierarchical-deterministic-wallets.org — Luke Dashjr
seed.bitcoinstats.com          — Christian Decker
seed.bitcoin.jonasschnelli.ch  — Jonas Schnelli
seed.btc.petertodd.net         — Peter Todd
seed.bitcoin.sprovoost.nl      — Sjors Provoost
dnsseed.emzy.de                — Stephan Oeste
seed.bitcoin.wiz.biz           — Jason Maurice
```

Each of these domains returns a set of IP addresses for currently-reachable Bitcoin nodes. Your node connects to several of them, requests more peer addresses, and gradually builds up its address book.

### Hardcoded Seed Nodes

If DNS resolution fails (censored network, DNS outage), Bitcoin Core falls back to a **hardcoded list of seed IP addresses** compiled at release time. These are baked into the binary as a last resort. They're less reliable than DNS seeds (IPs change over time), but they provide a minimum bootstrap capability even in hostile network environments.

### The addr and addrv2 Messages

Once connected to at least one peer, your node participates in **address gossip**. Nodes periodically share the addresses of other nodes they know about:

```
┌─────────┐    addr/addrv2     ┌─────────┐
│  Node A  │ ───────────────→  │  Node B  │
│          │                   │          │
│ "I know  │                   │ Adds to  │
│ about    │                   │ address  │
│ Node C,  │                   │ book     │
│ Node D"  │                   │          │
└─────────┘                    └─────────┘
```

The original `addr` message (v1) only supported IPv4 and IPv6 addresses. **addrv2** (BIP 155, implemented in Core 22.0) extended this to support:

- **Tor v3** (.onion) addresses (56 characters)
- **I2P** addresses
- **CJDNS** addresses
- Any future network protocol with addresses up to 512 bytes

This was essential for privacy-focused node operation. Without addrv2, Tor and I2P nodes couldn't be discovered through normal gossip.

### Anchor Connections

Starting in Bitcoin Core 0.21.0, the node persists its **anchor connections** — the peers it was connected to at shutdown — and reconnects to them on startup. This provides continuity across restarts and makes it harder for an attacker to isolate the node (see Eclipse Attacks below).

```bash
# See your current peers
bitcoin-cli getpeerinfo

# Key fields to examine:
# "addr"        — peer's IP address and port
# "network"     — ipv4, ipv6, onion, i2p, cjdns
# "subver"      — peer's user agent string (e.g., "/Satoshi:27.0.0/")
# "synced_headers" — how many headers they've sent us
# "synced_blocks"  — how many blocks they've sent us
# "connection_type" — inbound, outbound-full-relay, block-relay-only, feeler, addr-fetch
```

### Connection Types

Bitcoin Core distinguishes several connection types, each serving a specific purpose:

- **Outbound full-relay** (8 by default) — Your node initiates these; they relay both transactions and blocks
- **Outbound block-relay-only** (2 by default) — Only relay blocks, not transactions; reduces fingerprinting risk
- **Inbound** (up to 125 by default) — Other nodes connect to you; only if you're listening (port 8333)
- **Feeler connections** — Short-lived connections to test if addresses in your address book are reachable
- **Addr-fetch connections** — One-time connections to seed nodes to populate your address book

The diversity of connection types is deliberate. Block-relay-only connections, for example, exist specifically to make eclipse attacks harder: an attacker who controls all your transaction-relaying peers still can't isolate you from the block chain if you have independent block-relay-only connections.

---

## Part III: Block Propagation

Block propagation is a race. When a miner finds a valid block, they want the rest of the network to know about it as fast as possible — because every second of delay is a second during which another miner might find a competing block at the same height, causing an orphan race and wasted work.

### The Old Way: Full Block Relay

In Bitcoin's early years, block propagation was simple and slow:

```
Miner finds block
       │
       ▼
Sends full block (~1-2 MB) to all peers
       │
       ▼
Each peer validates and forwards to their peers
       │
       ▼
Propagation time: 10-40 seconds across the network
```

This was inefficient because every node in the network already had most of the transactions in the block sitting in their mempool. Sending the full block meant redundantly transmitting data that the receiving node already possessed.

### Headers-First Sync (BIP 130)

Introduced in Bitcoin Core 0.10.0, **headers-first sync** changed how nodes synchronize with the chain. Instead of downloading blocks sequentially and validating as they go, nodes now:

1. **Download all block headers first** (80 bytes each — fast)
2. **Validate the proof-of-work chain** (headers contain nonce and difficulty target)
3. **Identify the best chain** (most cumulative proof-of-work)
4. **Request and download blocks in parallel** from multiple peers

```
┌──────────────────────────────────────────────────┐
│              Headers-First Sync                   │
│                                                   │
│  Phase 1: Download headers                        │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ... ┌──┐              │
│  │H0│→│H1│→│H2│→│H3│→│H4│→...→│Hn│  (~60 MB)    │
│  └──┘ └──┘ └──┘ └──┘ └──┘     └──┘              │
│                                                   │
│  Phase 2: Validate PoW chain                      │
│  (Can we trust that this chain has real work?)     │
│                                                   │
│  Phase 3: Request blocks in parallel              │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐            │
│  │Blk 0 │ │Blk 1 │ │Blk 2 │ │Blk 3 │  ...      │
│  │Peer A│ │Peer B│ │Peer C│ │Peer A│            │
│  └──────┘ └──────┘ └──────┘ └──────┘            │
│                                                   │
│  Phase 4: Validate full blocks, update UTXO set   │
└──────────────────────────────────────────────────┘
```

This matters enormously for initial block download (IBD). A new node can download headers from the fastest peer, verify the PoW chain in seconds, and then request blocks from multiple peers in parallel — dramatically reducing sync time compared to the old sequential approach.

The `sendheaders` message (BIP 130) also changed ongoing sync: instead of using `inv` messages to announce new blocks, peers send `headers` messages directly, allowing the receiving node to immediately validate the PoW and request the full block.

### Compact Blocks (BIP 152)

**Compact blocks** are the single most impactful optimization to Bitcoin's block propagation. Proposed by Matt Corallo in BIP 152, implemented in Bitcoin Core 0.13.0 (2016).

The insight: when a new block arrives, the receiving node already has ~99% of the transactions in its mempool. Sending the full block is almost entirely redundant. Instead:

1. The sending node transmits a **compact block** containing:
   - The block header
   - **Short transaction IDs** (6 bytes each, derived from txids using SipHash)
   - Any transactions the sender suspects the receiver might not have (called "prefilled transactions" — typically just the coinbase)
2. The receiving node **reconstructs the full block** from its mempool using the short IDs
3. If any transactions are missing, the receiver sends a `getblocktxn` request for just those specific transactions
4. The sender responds with a `blocktxn` message containing the missing transactions

```
┌─────────────────────────────────────────────────────────┐
│                 Compact Block Flow                       │
│                                                          │
│  Sender                              Receiver            │
│    │                                    │                │
│    │  ── cmpctblock ──────────────→     │                │
│    │     (header + short IDs            │                │
│    │      + prefilled coinbase)         │                │
│    │                                    │                │
│    │                        Reconstruct from mempool     │
│    │                        Missing 2 of 3000 txs?       │
│    │                                    │                │
│    │  ←── getblocktxn ────────────      │                │
│    │      (request 2 missing txs)       │                │
│    │                                    │                │
│    │  ── blocktxn ─────────────────→    │                │
│    │     (2 missing transactions)       │                │
│    │                                    │                │
│    │                        Block fully reconstructed    │
│    │                        Validate and relay           │
└─────────────────────────────────────────────────────────┘
```

**Bandwidth savings:** A typical block contains ~3,000 transactions. Sending full: ~1.5 MB. Sending compact: ~15-25 KB (header + short IDs). That's a **~98-99% reduction** in block relay bandwidth.

### High-Bandwidth vs Low-Bandwidth Modes

BIP 152 defines two compact block modes:

- **High-bandwidth mode:** The sender pushes compact blocks **immediately** upon receiving them, without waiting for the receiver to request them via `inv`. The receiver tells up to 3 peers to operate in this mode using `sendcmpct(1)`. This minimizes latency at the cost of some redundant data (you might receive the same compact block from multiple high-bandwidth peers).

- **Low-bandwidth mode:** The sender announces the new block via `inv`, waits for the receiver to request it with `getdata(CMPCT_BLOCK)`, and then sends the compact block. Lower bandwidth, higher latency.

Most nodes use high-bandwidth mode with their 3 fastest peers and low-bandwidth mode with everyone else. The goal is to minimize the time between "block found" and "block validated" — because in that window, the node is vulnerable to working on a stale chain tip.

### FIBRE — Fast Internet Bitcoin Relay Engine

**FIBRE** was a project by Gregory Maxwell designed for the most latency-sensitive use case: relay between miners. While compact blocks reduced bandwidth, FIBRE attacked **latency** directly.

FIBRE used **forward error correction (FEC)** — borrowed from satellite communication — to transmit block data over UDP. The sender encodes the compact block into a stream of FEC-coded packets. Even if some packets are lost, the receiver can reconstruct the full message without retransmission. This eliminated the round-trip delay of requesting missing transactions.

FIBRE was operated as a network of relay nodes on dedicated infrastructure, primarily connecting major mining pools. At its peak, it could relay blocks across the globe in under a second.

The project was eventually deprecated as compact blocks became more efficient and mining pool infrastructure matured. But its contribution to reducing orphan rates — and thus reducing centralization pressure from mining — was significant. The faster blocks propagate, the less advantage large miners have over small ones (because large miners are more likely to be the first to hear about their own blocks).

---

## Part IV: Transaction Relay

Every Bitcoin transaction begins its life as a message gossiped through the peer-to-peer network. The journey from "user broadcasts transaction" to "miner includes it in a block" passes through the transaction relay layer.

### The Message Flow

Transaction relay follows a three-step protocol designed to minimize redundant data transmission:

```
┌─────────┐                    ┌─────────┐
│  Node A  │                   │  Node B  │
│          │                   │          │
│ Has tx   │  ── inv ────→     │ "I have  │
│ abc123   │  (I have txid     │ txid     │
│          │   abc123)         │ abc123?" │
│          │                   │          │
│          │  ←── getdata ──   │ "I don't │
│          │  (send me abc123) │  have it,│
│          │                   │  send it"│
│          │                   │          │
│          │  ── tx ────────→  │ Validates│
│          │  (full tx data)   │ and adds │
│          │                   │ to       │
│          │                   │ mempool  │
└─────────┘                    └─────────┘
```

1. **`inv` (inventory)** — Node A announces "I have transaction with this ID." This is a 32-byte hash, not the full transaction.
2. **`getdata`** — Node B checks its mempool. If it doesn't already have the transaction, it requests the full data.
3. **`tx`** — Node A sends the complete serialized transaction.

This three-step dance prevents redundant transmission. If Node B already has the transaction (because another peer already relayed it), it simply ignores the `inv` and never requests the data.

**Erlay (BIP 330)** is a proposed improvement that replaces the `inv` flood with **set reconciliation** — nodes periodically compare their mempool contents using sketches (based on Minisketch), only exchanging the differences. This can reduce transaction relay bandwidth by ~40%. It was merged into Bitcoin Core in stages and is expected to be fully enabled in a near-future release.

### Mempool Policy vs Consensus Rules

This distinction is critical and widely misunderstood.

**Consensus rules** define what is valid in a block. If a transaction violates consensus rules, any block containing it is invalid and will be rejected by all full nodes. Consensus rules are enforced universally and changing them requires a soft fork or hard fork.

**Mempool policy** defines what a node is willing to relay and keep in its mempool. Policy can be **stricter** than consensus. A node's mempool is its own business — it can accept or reject whatever it wants.

Examples of policy-only rules (not consensus):

| Policy Rule | What It Does | Consensus? |
|-------------|-------------|------------|
| Minimum relay fee | Ignore transactions below 1 sat/vB | No — a miner could include a 0-fee tx in a block |
| Standard transaction types | Only relay known script templates | No — any valid script can appear in a block |
| OP_RETURN size limits | Core 27.0 limited OP_RETURN data to 83 bytes | No — consensus allows up to block size limit |
| Maximum transaction size | Policy limits individual tx to 400 kWU | No — consensus limit is the block weight limit |
| Dust limit | Reject outputs below ~546 satoshis | No — consensus permits any amount above 0 |

This distinction became politically explosive during the OP_RETURN debate (see [Chapter 6](06-bip110-debate.md)). When Bitcoin Core changed the default OP_RETURN relay limit, it changed **policy**, not consensus. Miners were always free to include larger OP_RETURN outputs; they just wouldn't propagate through default Core mempools.

The mempool is an economic buffer, not a consensus mechanism. It exists for convenience and fee estimation, not for rule enforcement.

### Transaction Packages and Package Relay (BIP 331)

One of Gloria Zhao's major contributions. The problem: in traditional relay, each transaction is evaluated independently. If Transaction B depends on Transaction A (B spends A's outputs), and Transaction A has a low fee, nodes might reject A for being below their minimum fee — even if B pays a very high fee that would make the package profitable for miners.

This is especially problematic for **Lightning Network** channels. When a channel is force-closed, the commitment transaction might have an outdated fee rate. The child transaction (CPFP — Child Pays for Parent) needs to be relayed alongside its parent, or the parent might never propagate.

**Package relay** (BIP 331) allows nodes to submit and evaluate groups of related transactions as a single unit:

```
┌────────────────────────────────────────────┐
│          Package Relay                      │
│                                             │
│  Parent tx: 1 sat/vB fee  (too low alone)   │
│       │                                     │
│       ▼                                     │
│  Child tx: 100 sat/vB fee (CPFP)            │
│                                             │
│  Package fee rate: (parent + child fees)    │
│                    / (parent + child vsize) │
│                  = high enough to relay     │
└────────────────────────────────────────────┘
```

Without package relay, the parent might be rejected before the child ever has a chance to "pay" for it. With package relay, the pair is evaluated together, and the combined fee rate determines acceptance.

This is complemented by **TRUC transactions** (BIP 431 — Topologically Restricted Until Confirmation), which restrict package topology to prevent pinning attacks. TRUC transactions limit how many unconfirmed children a transaction can have, making fee bumping more predictable.

### Fee Filters (BIP 133)

Nodes have limited mempool capacity (default: 300 MB). When the mempool is full, the lowest-fee transactions get evicted. BIP 133 introduced the **feefilter** message: a node tells its peers "don't send me transactions below X sat/vB."

```bash
# Check your node's mempool status
bitcoin-cli getmempoolinfo
# "size"         — number of transactions
# "bytes"        — total size in bytes
# "usage"        — memory usage
# "mempoolminfee" — current minimum fee rate to enter mempool
# "minrelaytxfee" — minimum fee rate to relay (static, from config)
```

When mempool congestion is high, feefilter values go up across the network. Peers stop relaying low-fee transactions to each other because no one has room for them. This is an emergent, distributed fee market operating at the relay layer.

---

## Part V: Network Topology and Eclipse Attacks

### What Is an Eclipse Attack?

An **eclipse attack** isolates a target node from the honest network by controlling all of its inbound and outbound connections. If an attacker succeeds, the victim node sees only the attacker's version of the blockchain.

```
┌──────────────────────────────────────────────┐
│               Normal Operation                │
│                                               │
│    ┌──┐   ┌──┐   ┌──┐   ┌──┐   ┌──┐         │
│    │P1│───│P2│───│  │───│P4│───│P5│         │
│    └──┘   └──┘   │  │   └──┘   └──┘         │
│                   │ME│                        │
│    ┌──┐   ┌──┐   │  │   ┌──┐   ┌──┐         │
│    │P6│───│P7│───│  │───│P8│───│P9│         │
│    └──┘   └──┘   └──┘   └──┘   └──┘         │
│                                               │
│  (Connected to diverse, honest peers)         │
└──────────────────────────────────────────────┘

┌──────────────────────────────────────────────┐
│                Eclipse Attack                 │
│                                               │
│    ┌──┐   ┌──┐   ┌──┐   ┌──┐   ┌──┐         │
│    │A1│───│A2│───│  │───│A3│───│A4│         │
│    └──┘   └──┘   │  │   └──┘   └──┘         │
│                   │ME│                        │
│    ┌──┐   ┌──┐   │  │   ┌──┐   ┌──┐         │
│    │A5│───│A6│───│  │───│A7│───│A8│         │
│    └──┘   └──┘   └──┘   └──┘   └──┘         │
│                                               │
│  (ALL connections controlled by attacker)     │
│  Attacker can: feed fake blocks, hide txs,    │
│  double-spend against the victim              │
└──────────────────────────────────────────────┘
```

**What an eclipse attacker can do:**

- **Double-spend against the victim** — show the victim a fake chain while spending the same coins on the real chain
- **Delay or hide transactions** — the victim never sees certain transactions
- **Waste a miner's hashpower** — trick an eclipsed miner into extending a stale chain
- **Selfish mining amplification** — combine eclipse attacks with selfish mining strategies

### Countermeasures in Bitcoin Core

The defense against eclipse attacks is a multi-layered strategy:

1. **Diverse peer selection** — Bitcoin Core uses a bucketed address table with randomization. Addresses are bucketed by source network group (/16 for IPv4), ensuring connections are drawn from diverse network ranges rather than a single attacker-controlled subnet.

2. **Outbound connection priority** — Your node initiates 8 outbound full-relay connections and 2 block-relay-only connections. Outbound connections are harder to attack than inbound because the node chooses whom to connect to.

3. **Anchor connections** — On restart, your node reconnects to the peers it had before shutdown, preventing an attacker from filling all connection slots during the restart window.

4. **Block-relay-only connections** — These 2 connections don't relay transactions, making them invisible to transaction-graph analysis. An attacker who monitors transaction relay patterns to identify your connections won't detect these.

5. **Feeler connections** — Periodic short-lived connections to random addresses in the address book, testing whether they're reachable. This keeps the address book fresh and makes it harder for an attacker to fill it with stale addresses.

6. **Tor and I2P connections** — Running connections over multiple network layers forces an attacker to control infrastructure on all of them simultaneously.

7. **Stale tip detection** — If your node hasn't received a new block in an unusually long time, it opens additional outbound connections to check if it's being eclipsed.

```bash
# Check your network connection diversity
bitcoin-cli getnetworkinfo
# "networks" array shows: ipv4, ipv6, onion, i2p, cjdns
# "connections" — total peer count
# "connections_in" — inbound connections
# "connections_out" — outbound connections
```

### Sybil Resistance

Eclipse attacks are a specific form of **Sybil attack** — where an adversary creates many fake identities to gain disproportionate influence. In Bitcoin's P2P network, the cost of running many nodes is:

- Hardware/bandwidth for each node
- IP addresses (IPv4 addresses are increasingly scarce and expensive)
- Tor/I2P identities (cheaper, but less effective at eclipsing clearnet nodes)

Bitcoin's connection limits (8 outbound + 2 block-relay-only = 10 controlled connections out of a potentially large network) mean an attacker needs to control a significant fraction of reachable nodes. With ~20,000 reachable nodes, that's expensive. But for a state-level attacker with access to a large IP range, it's not impossible — hence the layered defenses.

---

## Part VI: Privacy in Transaction Relay

Bitcoin transactions are pseudonymous — addresses are not linked to real identities. But the P2P network can leak identity information through **timing analysis**: if you can observe when a node first broadcasts a transaction, you can infer that the node's operator likely created that transaction.

This is not a theoretical concern. Chain analysis companies (Chainalysis, Elliptic) and intelligence agencies operate networks of listening nodes specifically to deanonymize transaction origins.

### Tor Support

Bitcoin Core has included built-in Tor support since version 0.12.0 (2016). Running Bitcoin over Tor:

- Hides your node's IP address from peers
- Prevents your ISP from seeing that you're running a Bitcoin node
- Makes it harder to correlate your node with your real-world identity

```bash
# bitcoin.conf settings for Tor
proxy=127.0.0.1:9050          # Route all connections through Tor SOCKS proxy
listen=1                       # Accept incoming connections
bind=127.0.0.1                 # Bind to localhost
onion=127.0.0.1:9050          # Use Tor for .onion connections

# Or for a Tor-only node (no clearnet at all):
onlynet=onion
proxy=127.0.0.1:9050
```

Bitcoin Core can also create a **Tor hidden service** automatically, giving your node a `.onion` address that other Tor-enabled nodes can connect to. This allows inbound connections without exposing your IP.

### I2P Support

Added in Bitcoin Core 22.0 (2021), **I2P** (Invisible Internet Project) is an alternative anonymity network. Unlike Tor, which is designed for accessing the regular internet anonymously, I2P is designed as a self-contained darknet — optimized for services hosted within the I2P network itself.

```bash
# bitcoin.conf settings for I2P
i2psam=127.0.0.1:7656         # Connect to local I2P SAM bridge
```

Running Bitcoin over both Tor and I2P simultaneously provides defense in depth: if one anonymity network is compromised, the other still protects your connections.

### Dandelion++ (BIP Proposal)

**Dandelion++** is a transaction relay protocol designed to hide which node originally broadcast a transaction. In standard relay, a node broadcasts its transaction to all peers simultaneously — creating a "fluff" pattern that propagates outward from the source. Timing analysis can trace this pattern back to the origin.

Dandelion++ adds a **stem phase** before the fluff phase:

```
┌─────────────────────────────────────────────────────────┐
│              Dandelion++ Protocol                        │
│                                                          │
│  STEM PHASE (private):                                   │
│  Tx passes through a random chain of nodes               │
│  Each node forwards to exactly ONE random peer            │
│                                                          │
│  [Origin] → [Node A] → [Node B] → [Node C]              │
│                                         │                │
│                                         ▼                │
│  FLUFF PHASE (public):                                   │
│  Node C broadcasts normally to all its peers              │
│                                                          │
│  Observers see Node C as the apparent origin,             │
│  not the actual origin                                    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

During the stem phase, the transaction is forwarded through a random chain of nodes, each passing it to exactly one peer. After a random number of hops (determined probabilistically), the transaction enters the fluff phase and is broadcast normally. Network observers see the fluff origin (Node C) as the apparent broadcaster — but the real origin could be any node in the stem.

Dandelion++ has been implemented in some altcoins (Zcash, Grin) but has not yet been merged into Bitcoin Core. It remains a BIP proposal with ongoing discussion about implementation complexity and interaction with other relay optimizations. Some privacy-focused Bitcoin node implementations (like Bitcoin Knots) have experimented with it.

### Why Transaction Origin Privacy Matters

A concrete scenario: you live in a country where Bitcoin is restricted or surveilled. You broadcast a transaction from your home node. A government-operated listening node observes that your IP address was the first to relay that transaction. Now your IP is linked to that transaction — and potentially to your real identity.

Even in jurisdictions where Bitcoin is legal, origin-IP data combined with blockchain analysis can:
- Link pseudonymous addresses to real identities
- Reveal spending patterns (timing, amounts, destinations)
- Enable targeted phishing or physical attacks ("$5 wrench attacks")

Running over Tor/I2P, using Dandelion++ (when available), and connecting through diverse network paths are all defenses against this surveillance vector.

---

## Part VII: AssumeUTXO

### The Sync Time Problem

Syncing a new Bitcoin full node from scratch — initial block download (IBD) — means downloading and validating every block since January 3, 2009. On consumer hardware, this takes **6-24 hours** depending on CPU, disk speed, and bandwidth. That's a significant barrier to running a full node.

### The AssumeUTXO Solution (Bitcoin Core 28.0+)

**AssumeUTXO** lets a new node start validating immediately by loading a **UTXO snapshot** — a serialized copy of the UTXO set at a specific block height. The node:

1. Downloads the UTXO snapshot (a few GB, much smaller than the full blockchain)
2. Verifies the snapshot's hash matches a value hardcoded in the Bitcoin Core binary
3. Begins validating new blocks **from the snapshot height** — immediately usable
4. In the background, downloads and validates all historical blocks from genesis to the snapshot height
5. When background validation completes, verifies that the UTXO set it computed matches the snapshot

```
┌───────────────────────────────────────────────────────────┐
│                   AssumeUTXO Flow                          │
│                                                            │
│  Step 1: Load snapshot (block 840,000)                     │
│  ┌──────────────────────────────────┐                      │
│  │  UTXO Set @ block 840,000        │  Verify hash         │
│  │  ~170M UTXOs, ~7 GB              │  matches hardcoded   │
│  └──────────────────────────────────┘  value                │
│                                                            │
│  Step 2: Start validating from block 840,001               │
│  ┌──────────────────────────────────────────────────┐      │
│  │  840,001 → 840,002 → ... → current tip           │      │
│  │  (Node is USABLE — can verify new transactions)   │      │
│  └──────────────────────────────────────────────────┘      │
│                                                            │
│  Step 3: Background validation (IBD in background)         │
│  ┌──────────────────────────────────────────────────┐      │
│  │  Block 0 → Block 1 → ... → Block 840,000         │      │
│  │  (Validates full history; proves snapshot honest)  │      │
│  └──────────────────────────────────────────────────┘      │
│                                                            │
│  Step 4: Compare — computed UTXO set == snapshot?          │
│  If yes: full validation complete, trustless from genesis   │
│  If no: snapshot was dishonest, node rejects it            │
└───────────────────────────────────────────────────────────┘
```

### How It Differs from Trusting a Checkpoint

This is the key question. "Isn't AssumeUTXO just trusting a checkpoint?"

No, and here's why:

- A **checkpoint** says "this block hash is valid — don't bother checking anything before it." It permanently skips validation. If the checkpoint is wrong, the node is permanently wrong.
- **AssumeUTXO** says "assume this UTXO set is valid **for now** — but verify it in the background." The node eventually validates everything from genesis. If the snapshot was dishonest, background validation will detect the mismatch and the node will reject it.

The trust model during background validation is temporarily weaker — you're trusting the hardcoded hash for the first few hours/days until background sync completes. But the hash is hardcoded in the same binary you're already trusting to validate blocks correctly. And the trust is time-bounded: once background validation finishes, you're fully trustless.

```bash
# Load a UTXO snapshot (Bitcoin Core 28.0+)
bitcoin-cli loadtxoutset /path/to/utxo-snapshot.dat

# Check background validation progress
bitcoin-cli getblockchaininfo
# "background_validation" field shows sync progress
```

### Practical Impact

AssumeUTXO reduces the time from "install Bitcoin Core" to "usable full node" from hours to **minutes**. The node can validate incoming transactions, check balances, and participate in relay almost immediately. Background validation catches up over the next several hours without blocking the user.

This directly addresses Satoshi's concern about the burden of running a node. If syncing takes 20 minutes instead of 20 hours, more people will run nodes.

---

## Part VIII: Compact Block Filters (BIP 157/158)

### The SPV Privacy Problem

Satoshi's original SPV design used **Bloom filters** (BIP 37) for lightweight clients. The idea: a light client sends a Bloom filter to a full node, specifying which addresses/scripts it cares about. The full node applies the filter to each block and returns only the matching transactions.

The problem: **Bloom filters leak privacy catastrophically.** The filter reveals which addresses the client is interested in. A malicious full node can analyze the filter to determine exactly which addresses the client holds. Worse, the client has no way to verify that the server is returning all relevant transactions — the server could lie by omission.

### Neutrino: Client-Side Filtering

**Compact block filters** (BIP 157/158), also known as the **Neutrino protocol**, flip the model:

1. The full node computes a **compact filter** for each block — a deterministic summary of all the scripts/addresses in that block
2. The light client downloads these filters (small — roughly 20 KB per filter for GCS filters)
3. The client checks each filter locally to see if it matches any of its own addresses
4. If a filter matches, the client downloads the **full block** and extracts its transactions

```
┌─────────────────────────────────────────────────────────┐
│           Bloom Filter (BIP 37) — OLD WAY                │
│                                                          │
│  Light client → sends filter to server                   │
│  Server → applies filter, returns matching txs           │
│                                                          │
│  PROBLEMS:                                               │
│  • Server sees your filter → knows your addresses        │
│  • Server can lie by omission (hide transactions)        │
│  • Server can DoS you by returning false positives       │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│        Compact Block Filters (BIP 157/158) — NEW WAY     │
│                                                          │
│  Full node → computes filter for each block              │
│  Light client → downloads filters                        │
│  Light client → checks filters locally (private!)        │
│  If match → downloads full block from ANY peer           │
│                                                          │
│  ADVANTAGES:                                             │
│  • Server never learns what addresses you care about     │
│  • Client can verify filter against full block           │
│  • Client downloads full blocks — can verify txs         │
└─────────────────────────────────────────────────────────┘
```

### Why This Matters for Lightning

The Neutrino protocol is the default light client backend for **LND** (Lightning Labs' Lightning Network implementation). Lightning wallets need to monitor the blockchain for:

- Channel open/close transactions
- Breach remediation (justice transactions if a counterparty tries to cheat)
- HTLC timeout transactions

Using Bloom filters for this would leak which channels you have open. Compact block filters keep that information private while still allowing the wallet to react quickly to on-chain events.

### Filter Types

BIP 158 defines the **basic** filter type, which covers all scriptPubKeys (output scripts) and all scripts being spent in each block. This is sufficient for wallet operations. The filter uses a **Golomb-Rice Coded Set (GCS)** — a space-efficient probabilistic data structure similar to a Bloom filter but deterministic and more compact.

```bash
# Get a compact block filter (if your node serves them)
# Requires blockfilterindex=1 in bitcoin.conf
bitcoin-cli getblockfilter "000000000000000000024bead8df69990852c202db0e0097c1a12ea637d7e96d"
# Returns: {"filter": "<hex>", "header": "<hex>"}
```

---

## Part IX: Current Network Statistics

### Reachable Nodes

As of early 2026, according to Bitnodes and similar monitoring services:

| Metric | Approximate Value |
|--------|------------------|
| Reachable full nodes (IPv4/IPv6) | ~17,000-20,000 |
| Estimated total nodes (incl. Tor, non-listening) | ~60,000-80,000 |
| Tor nodes (reachable) | ~3,000-4,000 |
| I2P nodes (reachable) | ~500-800 |
| CJDNS nodes | ~50-100 |

The "reachable" count only captures nodes that accept inbound connections on port 8333 (or equivalent for Tor/I2P). Many node operators run behind NATs or firewalls and don't accept inbound connections. These nodes are fully functional for their operators but invisible to network crawlers.

### Geographic Distribution

Bitcoin node distribution is heavily concentrated in wealthy, internet-connected countries:

| Country | Approximate % of Reachable Nodes |
|---------|----------------------------------|
| United States | ~25-28% |
| Germany | ~12-15% |
| France | ~6-8% |
| Canada | ~4-5% |
| Netherlands | ~3-5% |
| United Kingdom | ~3-4% |
| Singapore | ~2-3% |
| Japan | ~2-3% |

This geographic concentration is a real centralization concern. If a single jurisdiction's government ordered ISPs to block Bitcoin P2P traffic (port 8333), a significant portion of the network's connectivity would be disrupted. Tor and I2P provide some mitigation, but a coordinated multi-country crackdown could meaningfully impact the network.

### Network Protocol Breakdown

```bash
# Check your node's network breakdown
bitcoin-cli getnetworkinfo
```

Typical output includes:

| Network | Reachable | Connections |
|---------|-----------|-------------|
| IPv4 | Yes | Majority |
| IPv6 | Yes | Growing |
| Tor (onion) | Depends on config | Significant minority |
| I2P | Depends on config | Small but growing |
| CJDNS | Depends on config | Tiny |

### Software Versions

The network runs a mix of Bitcoin Core versions and alternative implementations:

- **Bitcoin Core 27.x-28.x** — The large majority of nodes
- **Bitcoin Core 25.x-26.x** — Older but still common
- **Bitcoin Core <25.0** — Outdated; lacks recent security fixes
- **Bitcoin Knots** — Luke Dashjr's fork with stricter policy defaults (see [Chapter 6](06-bip110-debate.md))
- **btcd** — Go implementation (Decred team, used by LND for some functions)
- **libbitcoin** — Alternative C++ implementation
- **Bitcoin Unlimited, Bitcoin Classic** — Legacy big-block forks; nearly extinct on the network

The dominance of Bitcoin Core is itself a centralization concern — a critical bug in Core could affect the vast majority of nodes simultaneously. Alternative implementations provide some resilience, but in practice, Core's review process and test coverage make it the safest option by a wide margin. The mono-culture risk is real but difficult to solve without lowering the quality bar.

---

## Part X: Network Commands and Diagnostics

Bitcoin Core provides extensive RPC commands for inspecting network state. These are essential for diagnosing connectivity issues, verifying peer diversity, and understanding what your node sees.

### Essential Commands

```bash
# Network overview
bitcoin-cli getnetworkinfo
# Returns: version, subversion, protocol version, connections, networks, relay fee,
#          local addresses, warnings

# Detailed peer information
bitcoin-cli getpeerinfo
# For each peer: addr, services, lastsend, lastrecv, bytessent, bytesrecv,
#                conntime, pingtime, version, subver, inbound/outbound,
#                synced_headers, synced_blocks, minfeefilter

# How many connections of each type
bitcoin-cli getconnectioncount
# Simple integer — total connections

# Add a peer manually
bitcoin-cli addnode "192.168.1.100:8333" "add"

# Disconnect a specific peer
bitcoin-cli disconnectnode "192.168.1.100:8333"

# Ban a misbehaving peer
bitcoin-cli setban "192.168.1.100" "add" 86400  # Ban for 24 hours

# List banned peers
bitcoin-cli listbanned

# Check blockchain sync status
bitcoin-cli getblockchaininfo
# "chain", "blocks", "headers", "verificationprogress", "pruned",
# "size_on_disk", "initialblockdownload"

# Mempool statistics
bitcoin-cli getmempoolinfo
# "size" (tx count), "bytes", "usage", "total_fee",
# "mempoolminfee", "minrelaytxfee"
```

### Diagnosing Common Issues

**Node not syncing:**
```bash
# Check if you're in IBD (Initial Block Download)
bitcoin-cli getblockchaininfo | grep initialblockdownload
# true = still syncing; false = caught up

# Check peer count — if 0, you have no connections
bitcoin-cli getconnectioncount

# Check if network is reachable
bitcoin-cli getnetworkinfo
# Look at "networkactive" — should be true
```

**High orphan rate (for miners):**
```bash
# Check the chaintips — multiple tips suggest reorgs
bitcoin-cli getchaintips
# "status": "active" = current best chain
# "status": "valid-fork" = valid alternative chain
# "status": "invalid" = invalid chain (rejected)
```

**Peer misbehavior:**
```bash
# Check for peers with high ban scores
bitcoin-cli getpeerinfo
# Look at "banscore" (deprecated in newer versions, but concept persists)
# Peers that send invalid data get disconnected automatically
```

---

## The Design Philosophy

The Bitcoin P2P network is not optimized for speed, bandwidth efficiency, or user experience. It is optimized for **resilience**. Every design decision — the redundant connections, the gossip-based propagation, the lack of a central relay, the support for multiple transport networks — exists to make the system as hard to shut down as possible.

A government can block port 8333. Bitcoin runs on Tor. A government can block Tor. Bitcoin runs on I2P. An ISP can deep-packet-inspect Bitcoin traffic. The v2 transport protocol (BIP 324, implemented in Core 26.0) encrypts peer-to-peer connections, making traffic indistinguishable from random noise.

The network doesn't need to be fast. It needs to be **unkillable**. Ten minutes per block is plenty of time for a block to propagate to every node on Earth, even through multiple layers of anonymity networks. The P2P layer is engineered for that design constraint.

Satoshi understood this from the beginning. The decision to use a gossip protocol rather than a centralized relay was not laziness — it was the only architecture compatible with a system that has no administrator, no coordinator, and no authority that can be coerced.

---

## Key Takeaways

1. **Full nodes are the security backbone** — they validate everything and trust no one. Pruned nodes are equally secure for their operators; they just can't serve historical data to peers.

2. **SPV is not trustless** — lightweight clients trust miners to validate correctly. Compact block filters (BIP 157/158) fix SPV's privacy leak but don't fix the trust model.

3. **Peer discovery bootstraps from DNS seeds** and then relies on address gossip (addr/addrv2). Multiple connection types (full-relay, block-relay-only, feeler) provide defense against eclipse attacks.

4. **Compact blocks (BIP 152) reduce block relay bandwidth by ~99%** — instead of sending full blocks, nodes send short transaction IDs and reconstruct blocks from their mempools.

5. **Mempool policy is not consensus** — nodes can enforce stricter relay rules than the consensus allows. This distinction matters enormously for protocol politics.

6. **Package relay (BIP 331) fixes CPFP propagation** — critical infrastructure for Lightning Network fee bumping.

7. **Eclipse attacks are the primary P2P threat** — Bitcoin Core uses diverse connection types, anchor peers, bucketed address tables, and multi-network support to resist them.

8. **Tor and I2P integration protect node operator privacy** — hiding your IP address from peers and your ISP. Dandelion++ (not yet in Core) would further hide transaction origin.

9. **AssumeUTXO (Core 28.0+) reduces sync time from hours to minutes** — load a UTXO snapshot, validate immediately, verify the snapshot in the background. Not a checkpoint — trust is time-bounded and self-verifying.

10. **The network is designed for resilience, not speed** — gossip propagation, multi-network support, encrypted transport (BIP 324), and no central coordinator make Bitcoin's P2P layer extremely difficult to disrupt.

---

**Previous:** [Chapter 10 — Mining & Hardware](10-mining-hardware.md) — From Satoshi's CPU to 1,160 TH/s hydro-cooled ASICs: the mining arms race and its economics.

**Next:** [Chapter 12 — The Future of Bitcoin](12-future-bitcoin.md) — CTV, OP_CAT, covenants, Silent Payments, BitVM, RGB, Ark, and the ossification debate.
