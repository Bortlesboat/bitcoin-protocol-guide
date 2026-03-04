# Satoshi Nakamoto: Primary Sources Reference

Verbatim quotes from the white paper and Satoshi's public writings, organized by protocol topic.
All quotes are sourced from bitcoin.org/bitcoin.pdf and nakamotoinstitute.org (the Nakamoto Institute's
curated archive of Satoshi's posts and emails — the canonical reference for researchers).

Satoshi's BitcoinTalk profile (575 posts, last active December 13, 2010):
https://bitcointalk.org/index.php?action=profile;u=3

---

## 1. UTXO Model / Transactions / Chain of Ownership

### White Paper — Section 2 (Transactions)

> "We define an electronic coin as a chain of digital signatures. Each owner transfers the coin to the
> next by digitally signing a hash of the previous transaction and the public key of the next owner
> and adding these to the end of the coin. A payee can verify the signatures to verify the chain of
> ownership."

**Source:** Bitcoin white paper, Section 2 "Transactions" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** This is the foundational definition of how value moves in Bitcoin. "Chain of digital
signatures" is Satoshi's own framing — not "chain of ownership of coins" but ownership verified
through a cryptographic chain. The UTXO model flows directly from this: a coin IS its history of
signatures, not a balance in an account.

---

### White Paper — Section 9 (Combining and Splitting Value)

> "Although it would be possible to handle coins individually, it would be unwieldy to make a
> separate transaction for every cent in a transfer. To allow value to be split and combined,
> transactions contain multiple inputs and outputs. Normally there will be either a single input from
> a larger previous transaction or multiple inputs combining smaller amounts, and at most two
> outputs: one for the payment, and one returning the change, if any, back to the sender."

**Source:** Bitcoin white paper, Section 9 "Combining and Splitting Value" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** Satoshi explicitly designed multi-input/multi-output transactions from day one.
The "at most two outputs" pattern (payment + change) is the original description of what every
basic Bitcoin transaction does — and why UTXO dust and change management matter.

---

### P2P Foundation Forum — Introducing Bitcoin (Feb 11, 2009)

> "One of the fundamental building blocks for such a system is digital signatures. A digital coin
> contains the public key of its owner. To transfer it, the owner signs the coin together with the
> public key of the next owner. Anyone can check the signatures to verify the chain of ownership.
> It works well to secure ownership, but leaves one big problem unsolved: double-spending."

**Source:** P2P Foundation forum post, "Bitcoin open source implementation of P2P currency"
February 11, 2009
https://satoshi.nakamotoinstitute.org/posts/p2pfoundation/1/

**Why it matters:** Satoshi's public introduction of Bitcoin to the P2P research community. He leads
with the digital signature / chain of ownership explanation before addressing the double-spend
problem — showing that the UTXO model was always the conceptual starting point, not an
implementation detail.

---

## 2. Script System

### BitcoinTalk — "Transactions and Scripts: DUP HASH160 ... EQUALVERIFY CHECKSIG"

> "The nature of Bitcoin is such that once version 0.1 was released, the core design was set in stone
> for the rest of its lifetime. Because of that, I wanted to design it to support every possible
> transaction type I could think of. The problem was, each thing required special support code and
> data fields whether it was used or not, and only covered one special case at a time. It would have
> been an explosion of special cases. The solution was script, which generalizes the problem so
> transacting parties can describe their transaction as a predicate that the node network evaluates.
> The nodes only need to understand the transaction to the extent of evaluating whether the
> sender's conditions are met."

**Source:** BitcoinTalk, "Re: Transactions and Scripts: DUP HASH160 ... EQUALVERIFY CHECKSIG"
June 17, 2010
https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/69/

**Why it matters:** This is Satoshi's clearest explanation of WHY Script exists and why it is stack-
based and predicate-oriented. He explicitly chose a generalized scripting approach over hard-coded
special cases to keep the design extensible without changing the core protocol. The phrase "set in
stone for the rest of its lifetime" is his own framing of Bitcoin's immutability rationale.

---

> "The script is actually a predicate. It's just an equation that evaluates to true or false. Predicate
> is a long and unfamiliar word so I called it script."

**Source:** BitcoinTalk, "Re: Transactions and Scripts: DUP HASH160 ... EQUALVERIFY CHECKSIG"
June 17, 2010
https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/69/

**Why it matters:** Satoshi's own etymology. He chose the word "script" as an accessible label for
what is formally a boolean predicate evaluator. This matters for understanding why Bitcoin Script is
intentionally not Turing-complete — it evaluates to true or false, full stop.

---

> "The design supports a tremendous variety of possible transaction types that I designed years ago.
> Escrow transactions, bonded contracts, third party arbitration, multi-party signature, etc. If
> Bitcoin catches on in a big way, these are things we'll want to explore in the future, but they all
> had to be designed at the beginning to make sure they would be possible later."

**Source:** BitcoinTalk, "Re: Transactions and Scripts: DUP HASH160 ... EQUALVERIFY CHECKSIG"
June 17, 2010
https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/69/

**Why it matters:** Satoshi anticipated multisig, escrow, and smart contract-like constructs from
the very start. Taproot and Lightning were not departures from his vision — they were what he
was pointing at when he said "explore in the future."

---

## 3. Block Size / Scaling / SPV

### White Paper — Section 8 (Simplified Payment Verification)

> "It is possible to verify payments without running a full network node. A user only needs to keep
> a copy of the block headers of the longest proof-of-work chain, which he can get by querying
> network nodes until he's convinced he has the longest chain, and obtain the Merkle branch linking
> the transaction to the block it's timestamped in. He can't check the transaction for himself, but
> by linking it to a place in the chain, he can see that a network node has accepted it, and blocks
> added after it further confirm the network has accepted it."

**Source:** Bitcoin white paper, Section 8 "Simplified Payment Verification" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** Satoshi built SPV (light clients) into the white paper from the beginning. He
explicitly did NOT expect all users to run full nodes. This is the canonical source for the node
security model debate.

---

### BitcoinTalk — Scalability and Transaction Rate (July 2010)

> "The current system where every user is a network node is not the intended configuration for
> large scale. That would be like every Usenet user runs their own NNTP server. The design
> supports letting users just be users. The more burden it is to run a node, the fewer nodes there
> will be. Those few nodes will be big server farms. The rest will be client nodes that only do
> transactions and don't generate."

**Source:** BitcoinTalk, "Re: Scalability and transaction rate"
July 2010
https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/127/

**Why it matters:** Satoshi's explicit prediction that at scale, full nodes would be run by
server farms, not ordinary users. He was not describing this as a failure mode — it was his
anticipated end state. This quote is central to the block size debate: Satoshi did not assume
every participant would run a full node.

---

### BitcoinTalk — [PATCH] increase block size limit (October 2010)

> "It can be phased in, like:
>
> if (blocknumber > 115000)
>     maxblocksize = largerlimit
>
> It can start being in versions way ahead, so by the time it reaches that block number and goes
> into effect, the older versions that don't have it are already obsolete.
>
> When we're near the cutoff block number, I can put an alert to old versions to make sure they
> know they have to upgrade."

**Source:** BitcoinTalk, "Re: [PATCH] increase block size limit"
October 4, 2010
https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/233/

**Why it matters:** Satoshi explicitly described a mechanism to raise the block size limit via a
scheduled hard fork — showing he viewed 1MB as a temporary anti-spam measure, not a
permanent cap. He also said in the same thread: "We can phase in a change later if we get closer
to needing it." This is the key Satoshi quote in the block size wars.

---

### White Paper — Section 7 (Reclaiming Disk Space)

> "A block header with no transactions would be about 80 bytes. If we suppose blocks are generated
> every 10 minutes, 80 bytes * 6 * 24 * 365 = 4.2MB per year. With computer systems typically
> selling with 2GB of RAM as of 2008, and Moore's Law predicting current growth of 1.2GB per year,
> storage should not be a problem even if the block headers must be kept in memory."

**Source:** Bitcoin white paper, Section 7 "Reclaiming Disk Space" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** Satoshi's own scaling analysis. He assumed Moore's Law would handle growth.
His 4.2MB/year figure for headers-only storage shows he was thinking explicitly about long-term
node resource requirements from day one.

---

## 4. Privacy

### White Paper — Section 10 (Privacy) — Full Section

> "The traditional banking model achieves a level of privacy by limiting access to information to the
> parties involved and the trusted third party. The necessity to announce all transactions publicly
> precludes this method, but privacy can still be maintained by breaking the flow of information in
> another place: by keeping public keys anonymous. The public can see that someone is sending an
> amount to someone else, but without information linking the transaction to anyone. This is similar
> to the level of information released by stock exchanges, where the time and size of individual
> trades, the 'tape', is made public, but without telling who the parties were.
>
> As an additional firewall, a new key pair should be used for each transaction to keep them from
> being linked to a common owner. Some linking is still unavoidable with multi-input transactions,
> which necessarily reveal that their inputs were owned by the same owner. The risk is that if the
> owner of a key is revealed, linking could reveal other transactions that belonged to the same owner."

**Source:** Bitcoin white paper, Section 10 "Privacy" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** This is the complete Bitcoin privacy model from the white paper. Satoshi's
design was pseudonymous, not anonymous — public keys serve as pseudonyms. He explicitly
acknowledged the common-input-ownership heuristic as a known privacy leak. The phrase
"additional firewall" for fresh keys per transaction is the origin of Bitcoin's address reuse guidance.

---

## 5. Data in Transactions / Non-Financial Use

### BitcoinTalk — "Suggestion: Allow short messages to be sent together with bitcoins?" (October 2010)

> "ECDSA can't encrypt messages, only sign signatures.
>
> It would be unwise to have permanently recorded plaintext messages for everyone to see. It would
> be an accident waiting to happen.
>
> If there's going to be a message system, it should be a separate system parallel to the bitcoin
> network. Messages should not be recorded in the block chain. The messages could be signed with
> the bitcoin address keypairs to prove who they're from."

**Source:** BitcoinTalk, "Re: Suggestion: Allow short messages to be sent together with bitcoins?"
October 23, 2010
https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/239/

**Why it matters:** Satoshi explicitly opposed storing non-financial data on-chain. He viewed
the blockchain as a transaction ledger, not a general-purpose data store. This is the primary
canonical Satoshi quote in the Ordinals/inscription debate — he called permanently recorded
messages "an accident waiting to happen."

---

## 6. Mining Incentives / Fee Transition

### White Paper — Section 6 (Incentive)

> "By convention, the first transaction in a block is a special transaction that starts a new coin
> owned by the creator of the block. This adds an incentive for nodes to support the network, and
> provides a way to initially distribute coins into circulation, since there is no central authority to
> issue them. The steady addition of a constant of amount of new coins is analogous to gold miners
> expending resources to add gold to circulation. In our case, it is CPU time and electricity that
> is expended."

**Source:** Bitcoin white paper, Section 6 "Incentive" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** The gold-mining analogy is Satoshi's own. He framed block rewards not as
"printing money" but as the cost of securing the network (CPU + electricity), analogous to
the cost of gold mining. This is the origin of the "digital gold" framing.

---

> "The incentive can also be funded with transaction fees. If the output value of a transaction is
> less than its input value, the difference is a transaction fee that is added to the incentive value
> of the block containing the transaction. Once a predetermined number of coins have entered
> circulation, the incentive can transition entirely to transaction fees and be completely inflation free."

**Source:** Bitcoin white paper, Section 6 "Incentive" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** Satoshi explicitly designed the subsidy-to-fees transition. Bitcoin was never
intended to have permanent block rewards — he expected fees to eventually replace subsidies entirely.
The phrase "completely inflation free" is his own description of the post-subsidy endgame.

---

## 7. Network Security / 51% Attack / Honest Majority

### White Paper — Section 6 (Incentive) — 51% Game Theory

> "The incentive may help encourage nodes to stay honest. If a greedy attacker is able to assemble
> more CPU power than all the honest nodes, he would have to choose between using it to defraud
> people by stealing back his payments, or using it to generate new coins. He ought to find it more
> profitable to play by the rules, such rules that favour him with more new coins than everyone else
> combined, than to undermine the system and the validity of his own wealth."

**Source:** Bitcoin white paper, Section 6 "Incentive" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** This is Satoshi's game-theoretic argument for why a rational 51% attacker
would NOT attack. The argument is economic: an attacker with majority hash power earns MORE
from honest mining than from attacking, and attacking would destroy the value of their own
holdings. This is not a cryptographic guarantee — it is an incentive argument.

---

### White Paper — Abstract

> "The system is secure as long as honest nodes collectively control more CPU power than any
> cooperating group of attacker nodes."

**Source:** Bitcoin white paper, Abstract (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** The "honest majority" assumption stated in the very first paragraph. Satoshi
never claimed Bitcoin was unconditionally secure — he stated a precise security condition.
Everything in the white paper's security model flows from this single assumption.

---

### White Paper — Conclusion (Section 12)

> "The network is robust in its unstructured simplicity. Nodes work all at once with little
> coordination. They do not need to be identified, since messages are not routed to any particular
> place and only need to be delivered on a best effort basis. Nodes can leave and rejoin the network
> at will, accepting the proof-of-work chain as proof of what happened while they were gone. They
> vote with their CPU power, expressing their acceptance of valid blocks by working on extending
> them and rejecting invalid blocks by refusing to work on them. Any needed rules and incentives
> can be enforced with this consensus mechanism."

**Source:** Bitcoin white paper, Section 12 "Conclusion" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** "Vote with their CPU power" — Satoshi's description of how consensus works.
The conclusion is also the source of the "robust in its unstructured simplicity" framing. Note the
explicit statement that nodes can leave and rejoin freely — designed for intermittent connectivity.

---

## 8. The Genesis Block Message

> "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks"

**Source:** Coinbase transaction of Block 0 (the genesis block), mined January 3, 2009
Block hash: 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f

**Why it matters:** This message, encoded in the scriptSig of the genesis block's coinbase
transaction, is simultaneously a timestamp proof (the block could not have been mined before
this date, as it references that day's Times headline), a political statement (the banking system
bailout was the context for creating Bitcoin), and an aesthetic act. It is the only "message"
Satoshi definitively wrote into the blockchain itself.

The hexadecimal encoding in the raw transaction:
`5468652054696d65732030332f4a616e2f323030392043...`
decodes directly to this ASCII string.

The genesis block is unique: its 50 BTC coinbase reward is unspendable by protocol design —
there is no prior block for the coinbase to reference.

---

## 9. Satoshi's Last Known Public Communications

### Second-to-Last Forum Post — PC World Article on Bitcoin (December 11, 2010)

> "It would have been nice to get this attention in any other context. WikiLeaks has kicked the
> hornet's nest, and the swarm is headed towards us."

**Source:** BitcoinTalk, "Re: PC World Article on Bitcoin"
December 11, 2010, 11:39:16 PM
https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/259/

**Why it matters:** This was Satoshi's second-to-last public post. WikiLeaks had just begun
accepting Bitcoin donations after being cut off by Visa/Mastercard/PayPal. Satoshi was alarmed
that high-profile attention — especially from governments targeting WikiLeaks — would focus
scrutiny on Bitcoin before it was mature enough to survive it. The "hornet's nest" quote is his
most widely cited expression of concern about Bitcoin's political exposure.

---

### Final Public Forum Post — Version 0.3.19 / DoS Limits (December 12, 2010)

> "There's more work to do on DoS, but I'm doing a quick build of what I have so far in case it's
> needed, before venturing into more complex ideas. The build for this is version 0.3.19."

**Source:** BitcoinTalk, "Added some DoS limits, removed safe mode (0.3.19)"
December 12, 2010 (last active: December 13, 2010)
https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/260/

**Why it matters:** This technical release note was Satoshi's final public post. He did not write a
farewell. He simply stopped posting. His last words on the public forum were about software
reliability. His BitcoinTalk profile shows last active: December 13, 2010.

---

### Final Known Private Communication — Email to Mike Hearn (April 23, 2011)

> "I've moved on to other things. It's in good hands with Gavin and everyone."

**Source:** Private email to developer Mike Hearn, April 23, 2011
(Hearn has publicly confirmed and quoted this email)

**Why it matters:** This is the closest thing to a farewell Satoshi ever sent. When Hearn asked
whether Satoshi might return to the project, this was the reply. No dramatics, no manifesto —
just a handoff. The phrase "good hands with Gavin" refers to Gavin Andresen, who Satoshi had
designated as the lead maintainer. After this email, no further confirmed Satoshi communications
have been authenticated.

Note: In March 2014, a message appeared on the P2P Foundation forum from Satoshi's account
stating "I am not Dorian Nakamoto" in response to a Newsweek article. Its authenticity is disputed
— the account may have been compromised.

---

## 10. White Paper — Key Section Quotes

### Abstract (Complete)

> "A purely peer-to-peer version of electronic cash would allow online payments to be sent directly
> from one party to another without going through a financial institution. Digital signatures provide
> part of the solution, but the main benefits are lost if a trusted third party is still required to
> prevent double-spending. We propose a solution to the double-spending problem using a
> peer-to-peer network. The network timestamps transactions by hashing them into an ongoing chain
> of hash-based proof-of-work, forming a record that cannot be changed without redoing the
> proof-of-work. The longest chain not only serves as proof of the sequence of events witnessed,
> but proof that it came from the largest pool of CPU power. As long as a majority of CPU power is
> controlled by nodes that are not cooperating to attack the network, they'll generate the longest
> chain and outpace attackers. The network itself requires minimal structure. Messages are
> broadcast on a best effort basis, and nodes can leave and rejoin the network at will, accepting
> the longest proof-of-work chain as proof of what happened while they were gone."

**Source:** Bitcoin white paper, Abstract (October 2008)
https://bitcoin.org/bitcoin.pdf

---

### Section 5 — Network Steps (Complete)

> "The steps to run the network are as follows:
> 1) New transactions are broadcast to all nodes.
> 2) Each node collects new transactions into a block.
> 3) Each node works on finding a difficult proof-of-work for its block.
> 4) When a node finds a proof-of-work, it broadcasts the block to all nodes.
> 5) Nodes accept the block only if all transactions in it are valid and not already spent.
> 6) Nodes express their acceptance of the block by working on creating the next block in the
>    chain, using the hash of the accepted block as the previous hash."

**Source:** Bitcoin white paper, Section 5 "Network" (October 2008)
https://bitcoin.org/bitcoin.pdf

**Why it matters:** This is the simplest complete description of how Bitcoin works. Six steps.
No mention of miners, pools, fees, wallets, or addresses — just the raw protocol. Step 5 is
particularly important: validity is a prerequisite for acceptance. This is the foundation of the
full node's role.

---

## 11. Additional High-Value Quotes

### Cryptography Mailing List — Announcing the White Paper (October 31, 2008)

> "I've been working on a new electronic cash system that's fully peer-to-peer, with no trusted
> third party."

**Source:** Cryptography Mailing List, "Bitcoin P2P e-cash paper"
October 31, 2008, 18:10:00 UTC
https://satoshi.nakamotoinstitute.org/emails/cryptography/1/

**Why it matters:** Bitcoin's first public announcement. Satoshi's opening line. The date is
Halloween 2008 — two months before the genesis block, two months after the financial crisis
Lehman moment. The phrase "no trusted third party" is the core thesis in six words.

---

### P2P Foundation — On the Root Problem with Conventional Currency (February 2009)

> "The root problem with conventional currency is all the trust that's required to make it work.
> The central bank must be trusted not to debase the currency, but the history of fiat currencies
> is full of breaches of that trust. Banks must be trusted to hold our money and transfer it
> electronically, but they lend it out in waves of credit bubbles with barely a fraction in reserve."

**Source:** P2P Foundation forum, "Bitcoin open source implementation of P2P currency"
February 11, 2009
https://satoshi.nakamotoinstitute.org/posts/p2pfoundation/1/

**Why it matters:** Satoshi's most politically explicit statement. He names monetary debasement,
fractional reserve banking, and the trust model as the problems Bitcoin solves. This is the
foundational text for understanding Bitcoin as a political/economic project, not just a technical one.

---

### BitcoinTalk — On Lost Coins (June 21, 2010)

> "Lost coins only make everyone else's coins worth slightly more. Think of it as a donation to
> everyone."

**Source:** BitcoinTalk, "Re: Dying bitcoins"
June 21, 2010, 05:48:26 PM UTC
https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/71/

**Why it matters:** Satoshi's elegantly concise take on the economic effect of lost coins under a
fixed supply. He reframes a perceived flaw (coins being permanently destroyed) as a feature that
benefits all remaining holders via mild deflation.

---

### BitcoinTalk — On Bitcoin's Design History (2010)

> "Since 2007. At some point I became convinced there was a way to do this without any trust
> required at all and couldn't resist to keep thinking about it. Much more of the work was
> designing than coding."

**Source:** BitcoinTalk (quoted in "The Quotable Satoshi," Nakamoto Institute)
https://satoshi.nakamotoinstitute.org/quotes/bitcoin-design/

**Why it matters:** Satoshi worked on Bitcoin's design for at least two years before the white paper.
The ratio of design to coding work ("much more designing than coding") explains why the protocol
had such unusual conceptual coherence at launch.

---

### BitcoinTalk — On the 20-Year Horizon (2010)

> "I'm sure that in 20 years there will either be very large transaction volume or no volume."

**Source:** BitcoinTalk (quoted in multiple sources, original post 2010)
https://bitcointalk.org/index.php?topic=400623.0

**Why it matters:** Satoshi was not naive about Bitcoin's prospects. He explicitly acknowledged
it could fail entirely. This binary framing — massive adoption or zero — shows he understood
Bitcoin needed network effects to survive; there was no stable middle ground.

---

## Source Index

| Source | URL | Notes |
|--------|-----|-------|
| Bitcoin white paper | https://bitcoin.org/bitcoin.pdf | The canonical text; 9 pages |
| Nakamoto Institute — all forum posts | https://satoshi.nakamotoinstitute.org/posts/ | Complete archive, searchable by thread |
| Nakamoto Institute — emails | https://satoshi.nakamotoinstitute.org/emails/ | Cryptography list + bitcoin-list |
| Nakamoto Institute — Quotable Satoshi | https://satoshi.nakamotoinstitute.org/quotes/ | Curated by topic |
| BitcoinTalk profile | https://bitcointalk.org/index.php?action=profile;u=3 | 575 posts, last active Dec 13 2010 |
| P2P Foundation post | https://satoshi.nakamotoinstitute.org/posts/p2pfoundation/1/ | Feb 11, 2009 intro post |
| Cryptography list announcement | https://satoshi.nakamotoinstitute.org/emails/cryptography/1/ | Oct 31, 2008 |

---

## A Note on Authenticity

All quotes in this document are sourced from:
1. The white paper PDF at bitcoin.org (unchanged since publication)
2. The Nakamoto Institute's archive (nakamotoinstitute.org), which curates Satoshi's writings
   from primary sources including the Cryptography Mailing List archives and BitcoinTalk
3. BitcoinTalk forum posts under account `satoshi` (user ID 3, the first registered user)

Quotes attributed to Satoshi from secondary sources (media articles, Reddit posts) that do not
appear in the above primary sources should be treated with skepticism. The Nakamoto Institute
archive is the gold standard reference.

The Mike Hearn email (April 2011) is authenticated by Hearn's public statements but is a private
communication — no primary text URL exists.
