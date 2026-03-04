# Chapter 8: Satoshi's Timeline and Bitcoin Core History

## In Satoshi's Words

> "I've moved on to other things. It's in good hands with Gavin and everyone."
>
> — Satoshi Nakamoto, [private email to Mike Hearn](https://satoshi.nakamotoinstitute.org/emails/mike-hearn/), April 23, 2011

That seven-word sentence — "good hands with Gavin and everyone" — is one of the most consequential handoffs in the history of open-source software. A person who had just invented a new form of money, mined the first block, sent the first transaction, and quietly shepherded a global network into existence, disappeared. No press conference. No farewell post. Just a clean pass to the next runner.

This chapter is about that handoff, everyone who came after it, and what "who controls Bitcoin" actually means.

---

## Part I: The Satoshi Timeline (2007–2011)

### Before Bitcoin Was Bitcoin

Satoshi later told BitcoinTalk that he had been working on the design since 2007:

> "Since 2007. At some point I became convinced there was a way to do this without any trust required at all and couldn't resist to keep thinking about it. Much more of the work was designing than coding."

The design-to-code ratio matters. Bitcoin wasn't thrown together — it was architected. The system had unusual conceptual coherence because the economics, the game theory, and the cryptography had been worked out before a line of code was written.

By mid-2008, he had a working implementation and a draft whitepaper. Before going public, he quietly reached out to two people whose prior work he wanted to cite and who he respected:

- **Adam Back** (HashCash, the proof-of-work function Bitcoin uses) — contacted sometime before August 2008
- **Wei Dai** (b-money, a 1998 proposal for anonymous distributed electronic cash) — emailed August 22, 2008

Satoshi's email to Wei Dai is one of the few pre-launch primary sources we have:

> "I was very interested to read your b-money page. I'm getting ready to release a paper that expands on your ideas into a complete working system."

He needed the citation date for b-money. That's it. Professional, direct, no fanfare.

---

### The Complete Satoshi Timeline

| Date | Event | Notes |
|------|-------|-------|
| 2007 | Satoshi begins designing Bitcoin | His own account: "much more designing than coding" |
| Jul–Aug 2008 | Adam Back contacted about HashCash | Pre-launch private correspondence |
| **Aug 18, 2008** | bitcoin.org domain registered | Registered by Satoshi and Martti Malmi (Sirius) |
| **Aug 22, 2008** | Email to Wei Dai | Requests b-money citation date; shares pre-release draft |
| **Oct 31, 2008** | White paper published | Posted to the Cryptography Mailing List (metzdowd.com) |
| Nov 2008 | Code shared privately | Shared with a small number of developers before public launch |
| **Jan 3, 2009** | Genesis block mined | Block 0; coinbase embeds "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks" |
| **Jan 8, 2009** | Bitcoin v0.1 released | Announced on the Cryptography Mailing List |
| **Jan 9, 2009** | Network goes live | Software released publicly; block 1 mined |
| Jan 10, 2009 | Email to Wei Dai confirming launch | "I think it achieves nearly all the goals you set out to solve in your b-money paper." |
| **Jan 12, 2009** | First transaction | Block 170; Satoshi sends 10 BTC to Hal Finney |
| Feb 11, 2009 | P2P Foundation post | First broad public announcement of Bitcoin |
| Oct 2009 | First price established | New Liberty Standard: $0.0008/BTC |
| May 2010 | Bitcoin Pizza Day (nearby) | Laszlo Hanyecz pays 10,000 BTC for two pizzas |
| Jun 2010 | Satoshi grants commit access | Gavin Andresen, Martti Malmi, Laszlo Hanyecz, and others added |
| Jun 17, 2010 | Defends Script system on BitcoinTalk | "The nature of Bitcoin is such that once version 0.1 was released, the core design was set in stone" |
| Jul 2010 | Scalability thread | Predicts server farm nodes for large scale |
| Aug 2010 | Value overflow incident | Critical 0.1 BTC inflation bug fixed within hours by Satoshi; first emergency patch |
| Oct 4, 2010 | Block size discussion | Describes mechanism to raise size limit via scheduled hard fork |
| Oct 23, 2010 | Messages on the blockchain | "Messages should not be recorded in the block chain. The messages could be signed with the bitcoin address keypairs to prove who they're from." |
| **Dec 11, 2010** | WikiLeaks post | "WikiLeaks has kicked the hornet's nest, and the swarm is headed towards us." — second-to-last public post |
| **Dec 12, 2010** | Final public forum post | "There's more work to do on DoS, but I'm doing a quick build of what I have so far in case it's needed." (v0.3.19 release) |
| Dec 13, 2010 | Last login on BitcoinTalk | Profile shows last active this date — never returns |
| **Apr 23, 2011** | Final known private email — to Mike Hearn | "I've moved on to other things. It's in good hands with Gavin and everyone." |
| **Apr 26, 2011** | Final known email — to Gavin Andresen | "I wish you wouldn't keep talking about me as a mysterious shadowy figure... Maybe instead make it about the open-source project and give more credit to your dev contributors; it helps motivate them." |
| Mar 2014 | P2P Foundation account posts | "I am not Dorian Nakamoto" — authenticity disputed; account may have been compromised |

---

### The Genesis Block: A Political Statement Embedded in Code

Block 0 is unique in multiple ways. Its coinbase transaction contains this ASCII string in the scriptSig:

```
The Times 03/Jan/2009 Chancellor on brink of second bailout for banks
```

This is simultaneously:
1. **A timestamp proof** — the block cannot have been mined before January 3, 2009, because that newspaper headline didn't exist yet
2. **A political statement** — the UK government was bailing out banks that had created the 2008 financial crisis. Bitcoin's genesis was an explicit response to that
3. **An aesthetic act** — Satoshi chose to write something permanent into the first block of his system

The genesis block's 50 BTC coinbase reward is also permanently unspendable — a quirk of the protocol design that means Satoshi's "founder's reward" from block 0 can never be moved.

You can verify the message yourself:

```bash
bitcoin-cli getblock 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f 2
# Look for the coinbase transaction's scriptSig (hex) and decode it
```

---

### The First Transaction (Block 170)

On January 12, 2009 — nine days after the genesis block — Satoshi sent 10 BTC to Hal Finney, a cryptographer and the only other person running the Bitcoin software at the time. Finney had responded enthusiastically to the whitepaper announcement.

Block 170 contains what is generally considered the first Bitcoin transaction between two people. Finney ran a node, participated in the early network, and later wrote:

> "When Satoshi announced Bitcoin on the cryptography mailing list, he got a skeptical reception at best. Cryptographers have seen too many grand schemes by clueless noobs. They tend to have a knee-jerk reaction."

Finney was an exception. He saw what Satoshi was building and engaged seriously from day one.

You can decode this transaction yourself:

```bash
bitcoin-cli getrawtransaction f4184fc596403b9d638783cf57adfe4c75c605f6356fbc91338530e9831e9e16 2
```

---

### Why Did Satoshi Leave?

He never explained. The WikiLeaks post from December 11, 2010 provides a hint — he was alarmed at the political attention Bitcoin was attracting before it was robust enough to survive it. WikiLeaks had just begun accepting Bitcoin donations after being cut off by Visa, Mastercard, and PayPal. Government attention was incoming.

But there are other factors the community has noted:

- **The CIA visit announcement** — In late April 2011, Gavin Andresen announced he would give a presentation about Bitcoin at CIA headquarters. Satoshi's final email to Gavin — days later — conspicuously requested that Gavin stop describing him as a "mysterious shadowy figure." Whether the CIA presentation contributed to Satoshi's departure is speculation, but the timing is noted.
- **Bitcoin's political exposure** — The system was explicitly designed to resist state-level interference. A de facto leader who could be subpoenaed, threatened, or revealed undermined the whole premise.
- **Decentralization completed** — Satoshi had distributed keys, handed off maintenance, and built a system that didn't need him anymore.

His final email to Gavin ends with the request to give more credit to dev contributors. Not "please continue my vision." Not a farewell. A PR note about attribution. That's consistent with someone who had simply moved on and wanted the project to stand on its own.

---

### Satoshi's Coins: The Sleeping Giant

Analysis by Sergio Lerner (Patoshi pattern) and others suggests Satoshi mined approximately **1.1 million BTC** in Bitcoin's early days (blocks 1 through ~21,000). These coins have never moved. As of early 2026, they represent approximately 5% of the total Bitcoin supply.

The community's relationship with these coins is one of Bitcoin's most fascinating ongoing dynamics:
- They serve as a kind of decentralization proof — Satoshi could have sold long ago and didn't
- Any movement of these coins would be a market and philosophical event simultaneously
- Multiple people have claimed to be Satoshi; none have proven it by moving coins from known early addresses

---

## Part II: Bitcoin Core Maintainers — The History

### Understanding "Maintainer"

Before going through the history, it's important to understand what a Bitcoin Core maintainer actually is — and isn't.

A **maintainer** has merge access to the Bitcoin Core GitHub repository. They can:
- Merge pull requests that have been reviewed and approved by other contributors
- Sign official release binaries (so users can verify they're authentic)
- Coordinate the release process

A maintainer cannot:
- Force changes into the protocol without community consensus
- Override economic nodes (exchanges, businesses, wallets)
- Make Bitcoin do anything users don't accept

Jameson Lopp described it well: "Acting as a Bitcoin Core maintainer is often referred to as **janitorial work** because maintainers don't actually have the power to make decisions that run contrary to the consensus of contributors or of the users."

The misconception that "whoever controls Core controls Bitcoin" is wrong in a specific, important way. Bitcoin Core is software that nodes choose to run. If Core's developers made changes that economic nodes didn't accept, those nodes would fork or switch implementations. The maintainers serve the network; they don't govern it.

---

### The Maintainer Succession

```
Satoshi Nakamoto
     │  (2009–2010)
     │
     ▼
Gavin Andresen
     │  (2010–2014)
     │  Lead maintainer; also hands out keys to others
     │
     ▼
Wladimir van der Laan (laanwj)
     │  (2014–2022/2023)
     │  Longest-serving lead maintainer
     │  Steps down gradually; Craig Wright lawsuits accelerate decentralization
     │
     ▼
No single "lead maintainer"
     │  (2022–present)
     │  Collective of 5–6 trusted key holders
     ▼
Current: Marco Falke*, Hennadii Stepanov, Ava Chow (achow101),
         Ryan Ofsky, TheCharlatan
         *Falke stepped down 2023; others joined
```

---

### Satoshi → Gavin Andresen (2010–2014)

Gavin Andresen is a software developer from Amherst, Massachusetts — an alum of Princeton, with early-career experience at Silicon Graphics. He discovered Bitcoin in 2010 and almost immediately made himself useful.

His first notable contribution was the **Bitcoin Faucet**: a website that gave away free Bitcoin to anyone who solved a CAPTCHA. Gavin personally funded it with 10,000 BTC ($50 at the time). This was marketing and adoption infrastructure that no one had thought to build. It worked — it rapidly distributed Bitcoin to new users and showed Satoshi that Gavin understood the growth problem.

Satoshi gave Gavin escalating levels of trust through 2010. When Satoshi disappeared in late 2010 / early 2011, Gavin found himself as the de facto project lead by default. Crucially, **Satoshi never explicitly asked Gavin if he wanted the job**.

As Mike Hearn recalled: "Satoshi never actually asked Gavin if he wanted the job, and in fact he didn't. So the first thing Gavin did was grant four other developers access to the code as well."

Gavin's practical response was exactly right: he immediately distributed commit access to prevent a single-point-of-failure. The original five-person core team was assembled quickly — "essentially, whoever was around and making themselves useful at the time."

Gavin also founded the **Bitcoin Foundation** in 2012, a nonprofit aimed at funding development and promoting Bitcoin. The Foundation became controversial through no direct fault of Gavin's — board members Charlie Shrem and Mark Karpeles (Mt. Gox) were later implicated in legal scandals.

**The CIA Visit (2011)**: In late April 2011, Gavin announced on BitcoinTalk that he had accepted an invitation to speak about Bitcoin to the US intelligence community at CIA headquarters. He argued, reasonably, that engaging with government was better than being ignored by it. Many Bitcoin developers disagreed strongly. Whether or not this was the right call, it created tension that never fully resolved. Satoshi's final email to Gavin — written just days later — was notably pointed about how Gavin was representing Bitcoin in public.

**The Block Size Wars**: As Bitcoin grew, Gavin became increasingly convinced that the 1MB block size limit needed to be raised — urgently. He backed **BIP 101**, which would have increased the limit to 8MB immediately and continued doubling. When this was rejected by the Core contributors, he collaborated with Mike Hearn to launch **Bitcoin XT** in 2015, a fork of Core that implemented BIP 101. Bitcoin XT never gained enough economic support to succeed and was eventually abandoned.

**The Craig Wright Debacle (2016)**: In May 2016, Craig Wright — an Australian academic with a history of Bitcoin-adjacent claims — publicly claimed to be Satoshi Nakamoto. He provided demonstrations to a small group, including Gavin, in private sessions. Gavin published a blog post stating:

> "I believe Craig Steven Wright is the person who invented Bitcoin. After spending time with him I am convinced beyond a reasonable doubt."

The Bitcoin Core team immediately revoked Gavin's commit access to the repository, citing concerns he had been "hacked" — but the real concern was that he had endorsed a claim that the rest of the community found technically inconsistent. Wright's "proof" was quickly dissected and found wanting. Gavin later expressed doubt about his own conclusion. In 2024, a UK High Court ruled definitively: Craig Wright is not Satoshi Nakamoto. He had fabricated evidence, committed perjury, and lied to multiple courts across multiple jurisdictions over years.

Gavin's last commit to Bitcoin Core was in February 2016. He effectively retired from active development after the Wright controversy. His contributions during the critical 2010–2014 period, when Bitcoin could easily have died from abandonment or incompetence, were genuine and important.

---

### Gavin → Wladimir van der Laan (2014–2022/2023)

On April 7, 2014, Gavin stepped down from the lead maintainer role and nominated Wladimir J. van der Laan (GitHub: `laanwj`) as his successor. Van der Laan, a Dutch developer based in Eindhoven with a background in computer science and embedded systems, had been contributing to Core since 2011 and had become the most consistently active technical contributor.

The transition was quiet. By 2014, as Gavin was spending more time on Bitcoin Foundation work and less on actual code review, van der Laan had already been doing most of the actual maintainer tasks. Gavin's colleagues had noticed: as Wladimir told CoinDesk, "not only was [Gavin] not writing code, he wasn't discussing on the developer IRC, nor on GitHub, nor reviewing code."

**Van der Laan's tenure** (2014–2022/2023) is the longest sustained period of stewardship Bitcoin Core has had under any single developer. His technical contributions were substantial: he oversaw SegWit activation, Taproot, numerous security fixes, and the ongoing hardening of the codebase. His salary was funded by MIT's Digital Currency Initiative and, later, other Bitcoin development funding organizations.

What made van der Laan's position unusual was the personal cost. Being lead maintainer meant being the public face of Bitcoin's development — a target for:
- Intense political pressure during the block size wars
- Personal attacks from both sides of every major protocol debate
- Legal harassment from Craig Wright, who named van der Laan as a defendant in copyright suits claiming ownership of the Bitcoin whitepaper
- A 2020 Twitter controversy over variable naming conventions that nearly led to his resignation

The Craig Wright lawsuits in particular shaped his thinking about decentralization. Van der Laan began advocating for spreading commit access more widely — specifically so that no single person could be sued into compliance. This initiative succeeded and became the current multi-maintainer model.

Van der Laan announced he would begin delegating his tasks on **January 21, 2021**. He gradually stepped back over the next two years. In **February 2023**, he removed his own merge privileges — formally ending his tenure as lead maintainer. His stated hope was that Bitcoin Core would become more decentralized, with maintainer responsibility spread across more people and institutions.

---

### The Current Model: No Lead Maintainer (2022–Present)

Bitcoin Core no longer has a single "lead maintainer." Instead, a small group of developers hold **Trusted Keys** — cryptographic signing keys that authorize merges and sign release binaries.

**Current maintainers as of early 2026:**

| Name | GitHub | Area of Focus | Trusted Key Since |
|------|--------|---------------|-------------------|
| Hennadii Stepanov | hebasto | Build system, GUI, general | 2021 |
| Ava Chow | achow101 | Wallet, descriptors | 2021 |
| Ryan Ofsky | ryanofsky | Multiprocess architecture | 2023 |
| TheCharlatan | TheCharlatan | Validation logic, reproducibility | Jan 2026 |

**Recent departures:**
- **Marco Falke** (marcofalke) — Trusted Key 2016–2023; most prolific contributor in Core's history (2,000+ commits); stepped down in 2023, citing personal reasons. Still contributes actively but without merge access.
- **Gloria Zhao** (glozow) — Trusted Key 2022–2026; mempool and transaction relay specialist; stepped down early 2026 following sustained personal attacks during the OP_RETURN debate. See Chapter 6.

**Note:** Pieter Wuille (sipa) held commit/merge access from 2011 to 2022 and is one of the most impactful contributors in Bitcoin's history, though he stepped back from maintainer duties in 2022 alongside the broader transition.

---

## Part III: Key Contributors (Not All Maintainers)

These people shaped Bitcoin's protocol in ways that go beyond their formal role titles.

### Pieter Wuille (sipa)

Belgian software engineer; PhD in computer science from the University of Leuven; former Google SRE in Switzerland. Joined Bitcoin Core in May 2011 and held commit access until 2022. Currently at Chaincode Labs.

By commit count, Wuille is the closest to Satoshi in codebase impact. His contributions include:

- **libsecp256k1** — A highly optimized elliptic curve library for Bitcoin's ECDSA signatures. Before this, Bitcoin used OpenSSL for signature verification, which was slow and had a larger attack surface. Sipa's library made signature verification massively faster and more auditable.
- **BIP 32** (Hierarchical Deterministic Wallets) — The reason you can back up all your Bitcoin addresses with a single 12- or 24-word seed phrase. Wuille invented the HD wallet derivation scheme.
- **BIP 66** (Strict DER encoding) — Closed a consensus-critical security bug in signature encoding
- **Segregated Witness (SegWit)** — Co-primary author and implementer of the most significant protocol upgrade in Bitcoin's history (see [Chapter 3](03-segwit.md))
- **Schnorr/Taproot** — Co-authored the MuSig multi-signature scheme; major contributor to Taproot (see [Chapter 4](04-taproot.md))
- **Bech32 addresses** — The `bc1...` address format
- **MiniScript** — A language for writing and analyzing Bitcoin spending conditions
- **ultraprune** — Minimized the disk space required to sync a full node

The Human Rights Foundation's Finney Prize described Wuille's secp256k1 as "based on experimental code originally published by Hal Finney, which made signing and verifying signatures massively faster and more secure." That detail — that Wuille built on Finney's work — creates a quiet lineage from Bitcoin's first user to one of its most important technical improvements.

### Gregory Maxwell (gmaxwell)

One of the most intellectually formidable and personally controversial figures in Bitcoin's history. Maxwell was a Core developer from 2011 to 2018, co-founder and former CTO of Blockstream.

Before Bitcoin, he was a Mozilla developer who co-authored the Opus audio codec (RFC 6716) and contributed to the Daala video codec. He brought that deep systems-engineering background to Bitcoin.

His contributions include:

- **CoinJoin** — The fundamental technique for combining multiple users' transactions into one for privacy
- **Confidential Transactions** — Cryptographic technique to hide transaction amounts while proving they balance
- **BIP 32 co-work** — Contributed to homomorphic key derivation
- **Taproot original proposal** — Maxwell originally proposed Taproot (the MAST-based approach to hiding script complexity); Wuille and others developed it further
- **Compact blocks** — Improved block propagation speed significantly
- **FIBRE** (Fast Internet Bitcoin Relay Engine) — Reduced orphan rates dramatically
- **Block size debate intellectual leadership** — Maxwell was perhaps the most effective articulator of why large block sizes posed existential risks to decentralization

His controversies were real. He gave up his maintainer role in 2017, likely in part due to sustained personal pressure during the block size wars. His online style could be abrasive. He had disputes with other developers that became public. But on technical substance, even his critics frequently acknowledged the quality of his work.

### Marco Falke (marcofalke)

The single most prolific committer in Bitcoin Core history: over 2,000 individual commits, primarily in testing infrastructure. During three of his seven years as a trusted key holder, his work was funded by OKCoin and Paradigm.

Falke's focus was **test coverage** — regression tests, functional tests, fuzz tests. Consensus-critical software that controls real money needs a level of testing rigor that most software never approaches. Falke spent years ensuring that Bitcoin Core's test suite was comprehensive enough that bugs could be caught before they reached production.

He stepped down in 2023: "I remain passionate about open source and Bitcoin and I am positive about the future, however being a maintainer is no longer a good fit for me personally."

### Ava Chow / Andrew Chow (achow101)

Wallet maintainer and Trusted Key holder since 2021. Chow's primary focus is Bitcoin Core's wallet and key management system. Key contributions include:

- **Descriptor wallets** — A new wallet format that makes it explicit what scripts are being watched and controlled, replacing the older, less transparent approach
- **Output Script Descriptors (BIP 380–386)** — The formal specification for describing Bitcoin output scripts
- **PSBT** (Partially Signed Bitcoin Transactions, BIP 174) — The standard format for multi-party transaction signing, now used by hardware wallets, multisig coordinators, and CoinJoin implementations everywhere

### Gloria Zhao (glozow)

Trusted Key holder 2022–2026; the first publicly known female maintainer of Bitcoin Core. Zhao's domain was the **mempool** — the waiting room where transactions sit before being included in blocks — and transaction relay.

Her major contributions:

- **Package relay (BIP 331)** — Allows related transactions (packages) to be relayed together, which is critical for Lightning Network fee bumping
- **TRUC / BIP 431** (Topologically Restricted Until Confirmation) — A new type of transaction designed to prevent mempool pinning attacks
- **RBF improvements** — More predictable replace-by-fee behavior

Zhao stepped down in early 2026, following years of personal attacks that intensified during the OP_RETURN debate in 2025. She had deleted her X (Twitter) account in 2025 after a livestream in which another developer questioned her credentials. Her departure was mourned by many and celebrated by others — a reflection of how polarized Bitcoin development discourse had become. See [Chapter 6](06-bip110-debate.md) for the context.

---

## Part IV: Who Controls Bitcoin?

This is the question everyone asks and almost everyone answers incorrectly.

### The Wrong Answer

"Bitcoin Core developers control Bitcoin."

Why it's wrong: Core developers can only merge code. They cannot force anyone to run that code. Every node operator chooses which software to run. Every miner chooses what to mine. Every exchange chooses what to accept.

### The Right Mental Model

Bitcoin is a protocol that many parties implement. The parties who ultimately define what "Bitcoin" means are:

1. **Economic nodes** — full nodes run by exchanges, businesses, large holders. They define the rules they'll accept and reject.
2. **Miners** — choose which transactions to include and which chain tip to extend
3. **Users** — choose which software to run and which chain to transact on

Bitcoin Core developers can propose changes. But a change only "happens" if economic nodes adopt it. This is the key insight.

### The BIP Process

The **Bitcoin Improvement Proposal (BIP)** process is how changes are formally proposed and tracked. It was created by Amir Taaki in 2011 (BIP 1), modeled on Python's PEP (Python Enhancement Proposal) process.

**How a BIP works:**

```
Idea                          → Discussed informally (mailing list, IRC, GitHub)
                                        ↓
Draft BIP                     → Formal document: motivation, specification, backwards compatibility
                                        ↓
Community review              → Open feedback; author revises
                                        ↓
Reference implementation      → Code exists and can be tested
                                        ↓
Consensus reached             → Rough consensus (not a vote, but lack of serious objection)
                                        ↓
Activation                    → Soft fork: nodes enforce new rules; miners signal readiness
```

There are three BIP types:
- **Standards Track** — Protocol changes (consensus rules, P2P, wallet formats)
- **Process** — Changes to the BIP process itself
- **Informational** — Non-binding documentation or guidelines

Important: **No BIP has ever been implemented via a hard fork.** Every consensus change in Bitcoin's history has been a soft fork — backward compatible with older nodes (though older nodes see less of the picture).

### How Soft Forks Actually Activate

Before SegWit, most soft forks used **BIP 9** (miner-activated, also called MASF — miner-activated soft fork). Miners signal readiness by setting version bits in their block headers. Once 95% of miners signal over a defined window, the soft fork activates.

The problem with BIP 9: it gave miners de facto veto power. They could block protocol upgrades that had broad community support simply by not signaling.

SegWit demonstrated this problem: miners — primarily large Chinese mining pools with commercial interests in competing settlement layers — delayed SegWit signaling for over a year despite overwhelming developer and user support.

### The UASF Precedent (2017)

The **User-Activated Soft Fork (UASF)** is the most important governance event in Bitcoin's history. It established, once and for all, who actually controls Bitcoin.

Background: By 2017, SegWit had clear technical and community support but miners were refusing to signal. Developer Shaolin Fry proposed **BIP 148**: a flag-day activation where, on August 1, 2017, all nodes running BIP 148 would simply start rejecting any block that didn't signal SegWit readiness. No miner permission required.

What this meant: if enough economic nodes ran BIP 148, miners would face a choice:
- Mine SegWit-signaling blocks and stay on the economically dominant chain
- Mine non-SegWit blocks and mine into a worthless chain

The UASF movement grew. Exchange support grew. Wearing a "UASF" hat at Bitcoin conferences became a political statement. Miners read the situation correctly.

**Before August 1, 2017 arrived**, miners capitulated and signaled SegWit. BIP 148 never actually had to execute. But the threat was credible enough that SegWit activated in August 2017 via a more conventional path (BIP 91 → BIP 141).

The lesson: **users and economic nodes, not miners, have final authority over protocol rules.** The UASF precedent now exists as proof. Any future attempt by miners to block changes that economic nodes strongly want will face the same counter: a credible threat that nodes will simply stop accepting the miners' blocks.

Bitcoin Knots' enforcement of BIP 110 restrictions (Chapter 6) operates on the same principle, just from the other direction: a minority of nodes enforcing stricter rules, hoping to gather enough economic weight that miners follow.

### The "No One Controls Bitcoin" Statement

This claim is sometimes dismissed as naive idealism. Here's why it's technically accurate:

| Who | What they control | What they don't control |
|-----|-------------------|------------------------|
| Miners | Which transactions go in blocks, which chain tip to extend | Which chain economic nodes accept, consensus rules |
| Core maintainers | Which code gets merged into Bitcoin Core | Whether nodes run that code, consensus rules |
| Economic nodes | Which rules they enforce, which chain they transact on | What code other nodes run, what miners mine |
| Large holders | How much BTC they buy/sell, market price | Protocol rules, who mines what |
| Governments | Exchanges within their jurisdiction, legal pressure on developers | Protocol itself, global nodes |

No single party controls all of these simultaneously. Bitcoin has no CEO, no board of directors, no kill switch, and no mailing address to serve a court order to.

The closest thing to "control" that exists is the collective of economic nodes. If they converge on a rule, that rule is Bitcoin. If they diverge, you get a chain split (like Bitcoin Cash in 2017). Even that split shows the system working as designed: those who disagreed could leave.

---

## The Unanswered Question

Who is Satoshi Nakamoto?

After 17 years, we don't know. The serious candidates proposed over the years (Hal Finney, Nick Szabo, Wei Dai, Adam Back) have all denied it or lack sufficient evidence. Craig Wright spent years claiming it, fabricated evidence across multiple court cases, and was definitively ruled not to be Satoshi by a UK High Court in 2024.

The most honest answer: it may not matter anymore. Satoshi designed Bitcoin to work without him. It does. Whether Satoshi was one person or a small group, whether they are alive or dead, whether they will ever return — Bitcoin's protocol has been running continuously since January 3, 2009. The coinbase transactions have kept coming. The blocks have kept getting mined. The keys have passed from hand to hand.

The system works. That was the goal.

---

## Key Takeaways

1. Satoshi worked on Bitcoin's design for two years before publishing the whitepaper (Oct 31, 2008)
2. The genesis block (Jan 3, 2009) embedded a political statement; the first transaction (Jan 12, 2009) went to Hal Finney
3. Satoshi's last public post was Dec 12, 2010; his last known private communication was April 26, 2011 — no farewell, just a pass to Gavin and the team
4. The maintainer lineage: Satoshi → Gavin Andresen (2010–2014) → Wladimir van der Laan (2014–2022) → collective model (2022–present)
5. "Maintainer" means merge access and release signing — not protocol authority
6. The 2017 UASF proved that economic nodes, not miners, have final say over protocol rules
7. Bitcoin Core's trusted key group as of 2026: Hennadii Stepanov, Ava Chow, Ryan Ofsky, TheCharlatan (plus others with active merge access)
8. Nobody controls Bitcoin. The UTXO set runs as long as nodes run. The nodes run as long as the incentives hold.

---

**Previous:** [Chapter 7 — Node Setup](07-node-setup.md) — Run your own node and verify everything yourself.

**Related:** [Satoshi's Words](satoshi-quotes.md) — Verbatim quotes from the white paper and BitcoinTalk, with context.
