# Chapter 10: Mining & Hardware

## In Satoshi's Words

> "At first, most users would run network nodes, but as the network grows beyond a certain point, it would be left more and more to specialists with server farms of specialized hardware."
>
> — Satoshi Nakamoto, Bitcoin whitepaper mailing list, November 3, 2008

Satoshi predicted industrial mining before Bitcoin had a single user. Before there was a genesis block, before there was a price, before Hal Finney ran the second node — Satoshi already knew that CPU mining was a bootstrapping mechanism, not an endgame. The whitepaper mailing list message is unambiguous: specialist hardware, server farms, economies of scale. This wasn't a corruption of the original vision. It *was* the original vision.

The romance of "one CPU, one vote" lasted about eighteen months. What replaced it is an arms race that has consumed billions of dollars in capital, reshaped global energy markets, and produced the most powerful computational network in human history — one that now processes over 800 exahashes per second. This chapter covers how we got here, what the hardware landscape looks like today, and why the economics of mining are far more nuanced than "plug in a machine and print money."

---

## Part I: The Mining Evolution

### Era 1: CPU Mining (January 2009 – Mid 2010)

When Satoshi launched Bitcoin on January 3, 2009, you mined with whatever processor your computer had. Satoshi's own mining setup was almost certainly a modest desktop PC — he mined approximately 1.1 million BTC (the Patoshi pattern blocks) on what appears to have been a single machine, possibly throttled to avoid dominating the network too aggressively.

In this era:
- **Hardware**: Intel/AMD desktop CPUs. A Core 2 Duo might produce 2–5 MH/s (megahashes per second).
- **Who did it**: Satoshi, Hal Finney, a handful of cypherpunks and early adopters.
- **Difficulty**: Started at 1 (the minimum). By May 2010, it had only climbed to about 24.
- **Economics**: Electricity cost was irrelevant — you were mining bitcoin worth fractions of a cent per coin. The real cost was the opportunity cost of caring about this at all.

Satoshi himself acknowledged the transitional nature of CPU mining on BitcoinTalk:

> "The current system where every user is a network node is not the intended configuration for large scale."
> — Satoshi Nakamoto, BitcoinTalk, July 14, 2010

By mid-2010, GPU mining was already making CPU mining obsolete. The "one CPU, one vote" era lasted less than two years.

### Era 2: GPU Mining (Mid 2010 – 2012)

On July 18, 2010, a developer known as ArtForz is generally credited with creating the first OpenCL-based GPU miner. The logic was straightforward: Bitcoin's SHA-256 hashing algorithm is massively parallelizable, and GPUs have thousands of simple cores designed for parallel computation. A single high-end GPU could outperform dozens of CPUs.

In this era:
- **Hardware**: AMD Radeon HD 5870, 5970, 6990. Nvidia GPUs were used but AMD had a significant advantage due to their architecture being better suited to integer math.
- **Hashrate**: A Radeon HD 5870 could produce ~400 MH/s — roughly 100x faster than a high-end CPU.
- **Who did it**: Early technical adopters, gamers repurposing hardware, the first dedicated mining "rigs" with multiple GPUs.
- **Profitability**: Enormously profitable at first. A $350 GPU could mine dozens of BTC per day in late 2010. By late 2011, rising difficulty made it harder but still viable.
- **When it died**: GPU mining remained profitable through most of 2012 for those with cheap electricity and efficient setups. By mid-2013, ASICs made GPUs uncompetitive for SHA-256 mining. Some GPU miners pivoted to Litecoin (Scrypt) and later Ethereum (Ethash).

The GPU era established two patterns that persist today:
1. **Hardware arms races favor whoever adopts next-generation technology first** — early GPU miners made fortunes; late GPU miners barely broke even.
2. **Mining centralizes toward whoever has access to cheap electricity and better hardware** — the meritocracy of "anyone with a laptop" ended permanently.

### Era 3: FPGA Mining (Mid 2011 – Early 2013)

Field-Programmable Gate Arrays (FPGAs) are chips that can be configured at the logic-gate level to perform specific computations. Several companies produced FPGA-based Bitcoin miners starting in mid-2011.

In this era:
- **Hardware**: Xilinx Spartan-6, Altera Cyclone IV-based boards. Products like the BFL (Butterfly Labs) Single, Icarus, and ModMiner Quad.
- **Hashrate**: 100–800 MH/s per unit, comparable to GPUs.
- **Key advantage**: Not raw speed but **power efficiency**. An FPGA could deliver GPU-level hashrate at a fraction of the wattage, often under 10 watts per 100 MH/s compared to 200+ watts for an equivalent GPU.
- **Who did it**: Technically sophisticated miners, small companies specializing in FPGA design.
- **When it died**: Late 2012 to early 2013, when the first ASICs arrived and immediately made FPGAs noncompetitive.

The FPGA era was short — barely 18 months — and is often treated as a footnote. But it was significant as a proof of concept: custom silicon for Bitcoin mining was viable and profitable. The companies and engineers who learned FPGA design during this period became the talent pipeline for the ASIC era.

### Era 4: ASIC Mining (2013 – Present)

Application-Specific Integrated Circuits (ASICs) are chips designed to do one thing and one thing only: compute SHA-256 hashes. They cannot run an operating system, browse the web, or do anything else. They are the most efficient possible implementation of the SHA-256 algorithm in silicon.

**The first Bitcoin ASIC**: Avalon shipped the first commercial Bitcoin ASIC miner in January 2013 — the Avalon1, producing approximately 66 GH/s (gigahashes per second). It was roughly 50x more efficient than the best GPU and marked the point of no return for general-purpose mining hardware.

The early ASIC era was chaotic:
- **Butterfly Labs** sold ASIC miners on pre-order, then failed to deliver for months while mining with customers' machines. They were eventually shut down by the FTC for fraud.
- **KnC Miner** (Sweden) produced competitive ASICs but went bankrupt by 2016.
- **Avalon** (Canaan Creative, China) shipped first but couldn't maintain manufacturing leadership.
- **Bitmain** (China), founded in 2013 by Jihan Wu and Micree Zhan, rapidly became the dominant manufacturer. Their Antminer S-series became the de facto standard for Bitcoin mining hardware.

**Why ASICs centralized manufacturing**: Designing and fabricating a competitive ASIC requires:
1. Multi-million dollar tape-out costs (the process of turning a chip design into physical silicon)
2. Access to leading-edge semiconductor foundries (TSMC, Samsung)
3. Deep expertise in chip architecture, power delivery, and thermal management
4. Supply chain relationships for components, enclosures, and power supplies

These barriers to entry mean that as of 2026, essentially three companies produce the vast majority of Bitcoin mining hardware globally. This is a centralization pressure that the protocol itself cannot address — it's a manufacturing and capital markets problem.

| Era | Period | Hardware | Typical Hashrate | Power Efficiency | Key Players |
|-----|--------|----------|-----------------|------------------|-------------|
| CPU | 2009–2010 | Desktop processors | 2–5 MH/s | ~1,000 J/MH | Satoshi, Hal Finney, hobbyists |
| GPU | 2010–2012 | AMD Radeon 5870/6990 | 300–800 MH/s | ~2.5 J/MH | ArtForz, early miners |
| FPGA | 2011–2013 | Xilinx Spartan-6 | 100–800 MH/s | ~0.3 J/MH | BFL, Icarus, ModMiner |
| ASIC (early) | 2013–2016 | 28nm–16nm chips | 1–14 TH/s | 50–100 J/TH | Avalon, Bitmain, KnC |
| ASIC (modern) | 2017–2023 | 7nm–5nm chips | 14–255 TH/s | 15–30 J/TH | Bitmain, MicroBT |
| ASIC (current) | 2024–2026 | 5nm–3nm chips | 200–1,160 TH/s | 8–18 J/TH | Bitmain, MicroBT, Auradine |

---

## Part II: Current Hardware Landscape (2025–2026)

The mining hardware market in 2025–2026 is defined by three manufacturers, a push toward sub-10 J/TH efficiency, and the emergence of liquid/hydro cooling as the differentiator between mid-tier and top-tier operations.

### The Big Three Manufacturers

**Bitmain (Antminer S-series)** remains the market leader, with an estimated 50–60% market share of new ASIC sales. Their Antminer S21 series represents the current generation.

**MicroBT (Whatsminer M-series)** is the primary competitor, with roughly 25–35% market share. Founded in 2016 by Yang Zuoxing, a former Bitmain chip designer (the subject of an extended legal dispute), MicroBT has consistently produced competitive alternatives.

**Auradine (Teraflux)** is the newest entrant — a US-based company founded in 2022 and backed by significant venture capital. Their Teraflux line targets the premium efficiency segment and represents the first serious American challenge to the Chinese duopoly.

### Hardware Comparison Table (2025–2026 Models)

| Model | Manufacturer | Hashrate (TH/s) | Efficiency (J/TH) | Cooling | Approx. Price (USD) |
|-------|-------------|------------------|--------------------|---------|---------------------|
| **Antminer S21** | Bitmain | 200 | 17.5 | Air | $3,000–3,500 |
| **Antminer S21 Pro** | Bitmain | 234 | 15.0 | Air | $4,500–5,500 |
| **Antminer S21+ (Hydro)** | Bitmain | 319 | 16.0 | Hydro | $6,000–7,500 |
| **Antminer S21 XP** | Bitmain | 270 | 13.5 | Air | $5,500–7,000 |
| **Antminer S21 XP (Hydro)** | Bitmain | 473 | 12.0 | Hydro | $9,000–12,000 |
| **Antminer S21 Hyd.** | Bitmain | 335 | 16.0 | Hydro | $7,000–8,500 |
| **Whatsminer M60S** | MicroBT | 186 | 18.5 | Air | $2,500–3,000 |
| **Whatsminer M60S+** | MicroBT | 200 | 17.0 | Air | $3,000–3,500 |
| **Whatsminer M66S** | MicroBT | 298 | 16.5 | Air | $5,000–6,000 |
| **Whatsminer M66S+ (Hydro)** | MicroBT | 380 | 14.5 | Hydro | $8,000–10,000 |
| **Teraflux AT2880** | Auradine | 290 | 16.0 | Air | $5,500–7,000 |
| **Teraflux (Hydro)** | Auradine | ~375 | ~9.5 | Hydro | $10,000–13,000 |
| **Antminer S21 Hyd. (1160T)** | Bitmain | 1,160 | 11.5 | Immersion/Hydro | Custom pricing |

*Note: Prices fluctuate with Bitcoin price and difficulty. The 1,160 TH/s unit is Bitmain's flagship immersion model — a single rack-mounted unit producing hashrate that would have represented a meaningful fraction of the entire network just a few years ago.*

### Cooling Is the New Battleground

Air-cooled miners top out around 270 TH/s with current chip technology. To push beyond that, manufacturers have turned to liquid cooling:

- **Direct-to-chip hydro cooling**: Coolant flows through cold plates mounted directly on the ASIC chips. This is what Bitmain's "Hydro" models use. Allows higher clock speeds and denser rack deployment.
- **Single-phase immersion cooling**: The entire miner is submerged in a dielectric fluid. Companies like LiquidStack and GRC provide the tanks. Eliminates fans entirely, reduces noise to near-zero, and enables heat reuse.
- **Two-phase immersion cooling**: The dielectric fluid boils on contact with the chips, and the vapor condenses on a heat exchanger. The most efficient cooling method, but also the most complex and expensive to deploy.

Industrial mining operations increasingly treat cooling infrastructure as the primary capital investment — the ASICs themselves are almost commoditized. The ability to remove heat determines power density, which determines cost per hash.

---

## Part III: Mining Economics

Mining is not a technology problem. It's an energy arbitrage problem with a hardware component. The miners who survive are the ones who secure the cheapest electricity, not necessarily the ones with the newest machines.

### Hashprice: The Universal Mining Metric

**Hashprice** is the daily revenue earned per unit of hashrate, typically expressed as USD per petahash per day ($/PH/day). It captures the combined effect of Bitcoin's price, network difficulty, transaction fees, and the block subsidy in a single number.

As of early 2026 (post-fourth halving):
- **Block subsidy**: 3.125 BTC per block (halved from 6.25 in April 2024)
- **Average transaction fees**: 0.2–0.5 BTC per block (highly variable; can spike to 2+ BTC during congestion)
- **Network hashrate**: ~800 EH/s
- **Hashprice**: Approximately $45–55/PH/day

For context, hashprice peaked above $400/PH/day during the 2021 bull market (when BTC was $60K+ and difficulty hadn't caught up). After the 2024 halving, it dropped below $50 and has largely stayed in that range.

### Difficulty Adjustments: The Thermostat

Bitcoin's difficulty adjustment is one of its most elegant mechanisms. Every 2,016 blocks (approximately every two weeks), the protocol recalculates the mining difficulty target:

```
new_difficulty = old_difficulty × (2016 blocks × 10 minutes) / (actual time for last 2016 blocks)
```

If blocks came too fast (miners added hashrate), difficulty goes up. If blocks came too slow (miners dropped off), difficulty goes down. The maximum adjustment per period is capped at 4x in either direction — a safety valve Satoshi added to prevent instability.

This creates a self-regulating system:
1. Bitcoin price rises → mining becomes more profitable → miners add hardware → hashrate increases → difficulty increases → profitability normalizes
2. Bitcoin price falls → mining becomes unprofitable for marginal miners → they shut off → hashrate decreases → difficulty decreases → remaining miners become profitable again

The difficulty adjustment is why Bitcoin's block time has remained remarkably close to 10 minutes for seventeen years, despite hashrate increasing by a factor of roughly 10^14 from the first block to today.

### Electricity Cost Breakeven Analysis

The single most important variable in mining economics is the cost of electricity. Here's a breakeven analysis for a modern air-cooled miner (Antminer S21 Pro, 234 TH/s, 15 J/TH) at different electricity rates, assuming a Bitcoin price of $90,000 and network difficulty as of early 2026:

| Electricity Rate ($/kWh) | Daily Power Cost | Daily Revenue | Daily Profit | Monthly Profit | Status |
|--------------------------|-----------------|---------------|-------------|----------------|--------|
| $0.02 | $1.68 | $11.50 | $9.82 | $295 | Highly profitable |
| $0.04 | $3.37 | $11.50 | $8.13 | $244 | Profitable |
| $0.06 | $5.05 | $11.50 | $6.45 | $194 | Profitable |
| $0.08 | $6.74 | $11.50 | $4.76 | $143 | Marginally profitable |
| $0.10 | $8.42 | $11.50 | $3.08 | $92 | Thin margins |
| $0.12 | $10.11 | $11.50 | $1.39 | $42 | Barely viable |
| $0.14 | $11.79 | $11.50 | -$0.29 | -$9 | Unprofitable |

*Assumes continuous operation; excludes hosting costs, maintenance, hardware depreciation, and cooling overhead. Real margins are 10-20% lower than shown.*

Key observations:
- Below $0.04/kWh is the "printing money" zone. This is where flared gas, stranded hydro, and curtailed renewable sites operate.
- $0.06–0.08/kWh is where well-run industrial operations live.
- Above $0.10/kWh, you're fighting for scraps. Residential electricity rates in most developed countries ($0.10–0.30/kWh) make home mining a money-losing hobby.
- At $0.14/kWh and above, even efficient hardware loses money. This is where older-generation machines (S19 series, <100 J/TH) have already been forced offline.

### The Halving Cycle and Profitability

Bitcoin's block subsidy halves every 210,000 blocks (approximately every four years). This is the single most predictable shock to mining economics:

| Halving | Date | Block Subsidy | Total Daily Issuance | Effect on Miners |
|---------|------|--------------|---------------------|-----------------|
| 0 (genesis) | Jan 2009 | 50 BTC | 7,200 BTC | N/A — no market |
| 1st | Nov 2012 | 25 BTC | 3,600 BTC | GPU miners pushed offline; ASICs arriving |
| 2nd | Jul 2016 | 12.5 BTC | 1,800 BTC | Older ASICs unprofitable; efficiency becomes critical |
| 3rd | May 2020 | 6.25 BTC | 900 BTC | S9-era machines (100+ J/TH) pushed offline |
| **4th** | **Apr 2024** | **3.125 BTC** | **450 BTC** | S17/S19 non-Pro models struggling; sub-25 J/TH required |
| 5th (est.) | ~2028 | 1.5625 BTC | 225 BTC | Transaction fees must become a larger share of revenue |

Each halving cuts the block subsidy in half overnight. Miners who were operating on thin margins are immediately pushed into negative territory. The survivors are those with:
1. The cheapest power
2. The most efficient hardware
3. The lowest overhead (cooling, rent, staff, financing)
4. Access to capital to upgrade hardware before the halving hits

The fourth halving in April 2024 was particularly brutal because it came during a period of already-elevated difficulty. Many operators running S19j Pro models (approximately 29.5 J/TH) found themselves unprofitable unless their power cost was below $0.05/kWh.

---

## Part IV: The Pool Landscape

### Why Pools Exist

Solo mining a Bitcoin block in 2026 is a lottery ticket. With network hashrate at ~800 EH/s, a single Antminer S21 Pro (234 TH/s) has a probability of finding a block of approximately:

```
234 TH/s / 800,000,000 TH/s = 0.00000029 (0.000029%)
```

At 144 blocks per day, that miner would expect to find one block approximately every **9,500 days** — about 26 years. When it does find one, it earns ~3.125 BTC plus fees (roughly $300,000 at current prices). But the variance is extreme: you might find one in a week, or you might never find one.

Mining pools solve this problem through statistical aggregation. Thousands of miners contribute hashrate to a pool, the pool finds blocks at a predictable rate, and rewards are distributed proportionally. You earn less per block, but you earn consistently.

### Current Pool Distribution

| Pool | Approx. Hashrate Share (2025–2026) | Headquarters | Notes |
|------|-------------------------------------|--------------|-------|
| **Foundry USA** | ~30% | Rochester, NY | A subsidiary of Digital Currency Group (DCG). Largest pool globally. |
| **AntPool** | ~18% | Beijing, China | Operated by Bitmain. Second largest. |
| **ViaBTC** | ~12% | Shenzhen, China | Third largest; also operates CoinEx exchange. |
| **F2Pool** | ~10% | Beijing, China | One of the oldest pools (founded 2013). |
| **Marathon (MARA Pool)** | ~5% | Fort Lauderdale, FL | Publicly traded miner (MARA) running their own pool. |
| **Binance Pool** | ~4% | Global | Exchange-operated pool. |
| **Ocean** | ~2% | Decentralized | Stratum V2 early adopter; non-custodial payouts via BOLT 12. |
| **Others** | ~19% | Various | Includes SpiderPool, Luxor, EMCD, SBI Crypto, Braiins, etc. |

**The concentration problem is severe.** The top 4 pools (Foundry, AntPool, ViaBTC, F2Pool) control approximately 70% of total network hashrate. This is not mining centralization in the traditional sense — the individual miners who point hashrate at these pools can switch at any time — but it creates a different, arguably more dangerous form of centralization: **block template centralization**.

### How Pools Work: Shares, Payouts, and the Trust Problem

A mining pool operates by distributing work units to its members. Each work unit is a candidate block template — a partially completed block header that the miner needs to complete by finding a valid nonce. Here's the flow:

1. **Pool constructs a block template** — selects transactions from the mempool, builds the Merkle tree, sets the coinbase transaction (with the pool's payout address)
2. **Pool distributes work** — sends each miner a unique portion of the nonce space to search
3. **Miners hash** — trying nonces until they find one that produces a hash below the pool's "share difficulty" (much easier than the network difficulty)
4. **Miners submit shares** — each valid share proves the miner did work, even if it wasn't a full block solution
5. **Occasionally, a share IS a block solution** — the pool broadcasts it to the network and earns the block reward
6. **Pool distributes rewards** — to all miners proportionally based on shares submitted

### Payout Methods

| Method | Full Name | How It Works | Risk to Miner | Risk to Pool |
|--------|-----------|-------------|---------------|--------------|
| **PPS** | Pay Per Share | Fixed payment per share, regardless of whether the pool finds blocks. Pool absorbs all variance. | None — guaranteed income per share | High — pool eats the variance |
| **FPPS** | Full Pay Per Share | Like PPS, but also includes a pro-rata share of estimated transaction fees | None | Highest — pool guarantees both subsidy and fee income |
| **PPLNS** | Pay Per Last N Shares | Payment based on shares submitted in the window before a block is found. Rewards vary. | Moderate — income varies with pool luck | Low — pool distributes what it earns |
| **PPS+** | PPS Plus | Block subsidy paid as PPS (fixed), but transaction fees paid as PPLNS (variable) | Low — subsidy is guaranteed, fees vary slightly | Medium |

FPPS has become the dominant payout method at major pools because miners prefer predictable income. The pool effectively acts as an insurance company, absorbing the variance of block-finding in exchange for keeping a small fee (typically 1–3%).

### Pool Hopping

Pool hopping is the practice of switching between pools to maximize short-term returns — mining at a PPLNS pool right after it finds a block (when few shares are in the window, meaning each new share earns a larger proportion) and leaving before the next block is found. Modern pools have implemented anti-hopping measures (larger N in PPLNS, score-based systems), and the dominance of PPS/FPPS has made hopping largely irrelevant. If you're paid per share regardless of blocks found, there's nothing to hop to.

---

## Part V: The Centralization Problem — Who Builds the Blocks?

This is the most important and least understood issue in Bitcoin mining today.

When you point your ASIC at Foundry USA or AntPool, you are not choosing which transactions go into blocks. You're not even choosing the block structure. You are hashing a template that the pool operator constructed. You are a dumb compute node executing someone else's decisions.

The pool operator decides:
- **Which transactions to include** (and which to exclude or censor)
- **How to order transactions** (relevant for MEV — Miner Extractable Value — and for Ordinals/BRC-20 sorting)
- **Whether to include OP_RETURN data, Ordinals inscriptions, or other "controversial" transaction types**
- **The coinbase message** (the arbitrary data in the coinbase transaction)

This means that even though "hashrate is distributed" across thousands of individual miners worldwide, the actual power to construct blocks — to decide what Bitcoin's blockchain contains — is concentrated in a handful of pool operators. Foundry USA alone constructs ~30% of all Bitcoin blocks. Add AntPool, and two entities construct roughly half.

This is not a theoretical concern. It has practical implications:

1. **Transaction censorship**: A pool operator could refuse to include transactions from sanctioned addresses. In 2022, Foundry USA and Marathon briefly experimented with OFAC-compliant block templates that excluded transactions involving sanctioned addresses. The backlash was intense, and both reversed course, but the capability exists.

2. **MEV extraction**: As on-chain activity becomes more complex (Ordinals, BRC-20 tokens, Runes), the ordering of transactions within a block has economic value. Pool operators can extract this value. Miners see none of it.

3. **Government coercion**: A government that wants to censor specific Bitcoin transactions doesn't need to control 51% of hashrate. It needs to pressure the pool operators who construct 51% of block templates. If Foundry USA (US-based) and AntPool (China-based) both complied with their respective governments, over 48% of block templates would be censored.

4. **Empty block mining**: Some pools have been observed mining empty blocks (containing only the coinbase transaction) immediately after receiving a new block, before they've had time to validate the full block and construct a new template. This is rational behavior for the pool (they earn the block subsidy without waiting) but harmful to the network (it wastes block space and delays transaction confirmation).

The fix is not to complain about pool centralization. The fix is to change the protocol so that miners — not pool operators — construct their own templates.

---

## Part VI: Stratum V2 — The Fix for Template Centralization

### The Problem with Stratum V1

The original mining pool protocol, Stratum V1, was introduced by Slush Pool (now Braiins) in 2012. It's simple and effective: the pool sends work to miners, miners return shares. But it has a fundamental design flaw: **the pool constructs the block template, and the miner has no say in what that template contains.**

Under Stratum V1, the miner is a hashing machine. Nothing more. It doesn't see the transactions, doesn't choose the block structure, and has no way to resist censorship by the pool operator.

### What Stratum V2 Changes

Stratum V2 is a complete protocol redesign, co-developed by Braiins, Square Crypto (now Spiral), and others. It introduces three key improvements:

1. **Job Declaration Protocol**: The miner can construct its own block template and submit it to the pool. The pool validates that the template is valid (correct difficulty, valid transactions) but cannot force the miner to use the pool's template. This is the critical change — it shifts template construction authority from pool operators back to individual miners.

2. **Encrypted connections**: Stratum V1 sends all communication in plaintext. Anyone between the miner and the pool (ISP, government, man-in-the-middle) can see which pool the miner is using and what work it's doing. Stratum V2 uses AEAD (Authenticated Encryption with Associated Data) to encrypt the connection.

3. **Bandwidth reduction**: Binary framing (instead of JSON over TCP) reduces bandwidth by roughly 50%, which matters for miners in areas with poor internet connectivity.

### Job Declaration: Why It Matters

With Job Declaration enabled:
- The **miner** selects transactions from its own mempool
- The **miner** builds the Merkle tree
- The **miner** decides what goes in the block
- The **pool** just validates and distributes the reward

This fundamentally changes the centralization dynamic. Even if 30% of hashrate points at Foundry USA, if those miners are running Stratum V2 with Job Declaration, Foundry can't censor transactions. Each miner constructs its own template. The pool is reduced to an accounting layer — a payout coordinator, not a block builder.

### Adoption Status

**Ocean** (founded by Jack Dorsey and Luke Dashjr) was the first major pool to implement Stratum V2 with Job Declaration support. Ocean also pioneered non-custodial payouts — miners receive their share of the block reward directly in the coinbase transaction rather than trusting the pool to pay them later.

**Braiins Pool** (formerly Slush Pool) has supported Stratum V2 since 2023, though full Job Declaration adoption among its miners has been slow.

**DEMAND**: The firmware side matters too. Braiins OS and other aftermarket firmware packages support Stratum V2 on compatible hardware. Bitmain's stock firmware does not natively support Stratum V2 Job Declaration as of early 2026 — a notable gap given Bitmain's hardware market share.

The honest assessment: Stratum V2 is technically ready and demonstrably superior. But adoption is slow because (a) most miners are economically indifferent to template construction — they care about payout reliability and low fees — and (b) pool operators have no incentive to give up template authority, which is a source of both economic value and influence.

---

## Part VII: Solo Mining vs. Pool Mining

### The Math

Solo mining makes economic sense only when your hashrate is large enough that the expected time between blocks is short relative to your financial planning horizon.

| Your Hashrate | Expected Time to Find a Block | Variance Assessment |
|--------------|-------------------------------|---------------------|
| 234 TH/s (1 S21 Pro) | ~26 years | Absurd — you'll go bankrupt first |
| 2.34 PH/s (10 S21 Pros) | ~2.6 years | Still extreme variance — could be 10+ years |
| 23.4 PH/s (100 S21 Pros) | ~95 days | High variance but beginning to be viable for well-capitalized operators |
| 234 PH/s (1,000 S21 Pros) | ~9.5 days | Reasonable — this is a small-to-medium mining farm |
| 2.34 EH/s (10,000 S21 Pros) | ~23 hours | Low variance — finding blocks almost daily |

### When Solo Mining Makes Sense

1. **You operate at scale**: If you have 100+ PH/s, you'll find blocks regularly enough that variance smooths out naturally. Major public miners like Marathon and CleanSpark solo-mine or run proprietary pools.

2. **You want maximum sovereignty**: Solo mining means you construct your own block template, select your own transactions, and answer to no pool operator. This is philosophically aligned with Bitcoin's design, even if economically suboptimal for small operations.

3. **You're running a lottery miner**: Some hobbyists run a single old-generation ASIC as a solo miner, fully aware they're unlikely to ever find a block. The expected value is approximately the same as pool mining (minus pool fees), but the payout distribution is binary: 0 or ~$300,000. For someone who can afford the electricity and treats it as entertainment, this is fine. It's not a business strategy.

### When Solo Mining Doesn't Make Sense

For virtually everyone else. If you're operating fewer than 50 PH/s and depend on mining income to cover costs, pool mining is the rational choice. The guaranteed, predictable revenue from FPPS payouts lets you plan, budget, and service debt — all things that are impossible when your income arrives in random ~$300,000 lump sums separated by months or years of nothing.

---

## Part VIII: The AI Pivot — Mining Facilities as Data Centers

One of the most significant trends in the mining industry since 2023 is the pivot toward high-performance computing (HPC) and artificial intelligence workloads. Several publicly traded Bitcoin miners have announced or begun converting portions of their mining infrastructure to GPU-based AI/ML compute.

### Why Mining Facilities Are Perfect for AI

Bitcoin mining facilities and AI data centers have nearly identical infrastructure requirements:

| Requirement | Bitcoin Mining | AI/HPC Data Center |
|-------------|---------------|---------------------|
| Cheap electricity | Yes — the primary input cost | Yes — GPU clusters are extremely power-hungry |
| High power density | 30–50+ kW per rack | 40–100+ kW per rack (Nvidia H100/B200 clusters) |
| Cooling infrastructure | Required (air, liquid, immersion) | Required (same or more demanding) |
| Remote/rural location OK | Yes — proximity to users irrelevant | Partially — latency matters for inference, less for training |
| Grid interconnection | Already negotiated and built | Would take 2–3 years to build from scratch |
| Permitting | Already obtained | Would take 1–3 years for new construction |

The key insight: **power infrastructure is the bottleneck for AI scaling, and Bitcoin miners already have it.** A mining facility with 100 MW of grid capacity, cooling, and permitting can repurpose that infrastructure for AI training or inference workloads faster than a hyperscaler can build new capacity from scratch.

### The Major Players

**Marathon Digital (MARA)**: The largest publicly traded Bitcoin miner by market cap. Marathon has announced plans to allocate portions of its 1+ GW pipeline to AI/HPC hosting. Their Garden City, Texas facility is being partially converted.

**IREN (formerly Iris Energy)**: Australian-founded, US-listed miner with operations in British Columbia and Texas. IREN has been the most aggressive AI pivoter, with Nvidia GPU clusters already deployed alongside Bitcoin ASICs. They've secured significant contracts for AI inference hosting.

**CleanSpark**: Georgia-based miner that has expanded rapidly through acquisition of distressed mining sites. More cautious on the AI pivot than peers, but has explored colocation arrangements.

**Hut 8**: Canadian miner that merged with USBTC in 2023. Has explicitly targeted AI/HPC as a diversification strategy, with GPU compute offerings.

**Core Scientific**: Emerged from bankruptcy in early 2024 and immediately pivoted toward AI hosting, signing a 200 MW deal with CoreWeave (a GPU cloud provider).

### The Economics of the Pivot

The math is compelling. Bitcoin mining generates approximately $30–60 per MWh consumed (depending on hashprice). AI/HPC hosting can generate $100–200+ per MWh consumed. The capital expenditure to convert is significant — GPU servers cost far more than ASICs, and the networking/storage requirements are different — but the revenue per megawatt is substantially higher.

The catch: AI hosting requires higher reliability (SLAs, redundancy, uptime guarantees) than Bitcoin mining. If a Bitcoin miner goes offline for an hour, you lose an hour of expected revenue. If an AI training cluster goes offline for an hour, you might corrupt a multi-million-dollar training run. The operational discipline required is different.

### Strategic Tension

There's a philosophical tension in the mining industry about this pivot. Bitcoin maximalists view it as miners abandoning their core function — securing the network — in pursuit of higher margins. The pragmatists view it as rational capital allocation that actually strengthens Bitcoin mining: AI revenue cross-subsidizes Bitcoin mining operations, allowing miners to keep hashrate online even during low-profitability periods.

The most likely outcome is hybrid facilities: some racks running ASICs, some running GPUs, with the allocation shifting dynamically based on relative profitability. Bitcoin miners become energy-monetization platforms rather than single-purpose hashing operations.

---

## Part IX: The Environmental Debate

Bitcoin mining's energy consumption is one of the most polarized topics in the space. The annual energy consumption of the Bitcoin network is estimated at 150–200 TWh (terawatt-hours) as of early 2026 — comparable to the energy consumption of a mid-sized country like Poland or Argentina. Both sides of the debate have legitimate points.

### The Case Against Bitcoin Mining's Energy Use

1. **Absolute consumption is enormous**: 150+ TWh per year is a lot of energy by any measure. Even if it's "only" 0.1% of global energy production, that's still enough to power tens of millions of homes.

2. **Carbon footprint depends on the energy mix**: Mining operations that run on coal-fired power (as many Chinese miners did before the 2021 ban, and as some operations in Kazakhstan and parts of the US still do) have a real and significant carbon footprint.

3. **The "useful work" question**: Unlike AI training, scientific computing, or industrial processes, Bitcoin mining's computational output has no direct productive use beyond securing the network. Every hash is discarded except the winning one. Critics argue this is wasteful by definition.

4. **E-waste**: ASICs have a useful life of 3–5 years before being replaced by more efficient models. Millions of obsolete miners end up in landfills. They contain valuable metals but are rarely recycled economically.

### The Case For Bitcoin Mining's Energy Use

1. **Energy mix is better than critics claim**: The Bitcoin Mining Council (an industry group, so take with appropriate skepticism) estimates that approximately 60% of Bitcoin mining uses sustainable energy sources. Cambridge's independent estimates are lower (roughly 40–50%) but still substantial. Many mining operations are specifically sited at renewable energy sources.

2. **Stranded energy monetization**: Bitcoin mining is uniquely suited to consuming energy that would otherwise be wasted:
   - **Flared natural gas**: Oil wells produce associated gas that is often flared (burned off) because there's no pipeline to transport it. Companies like Crusoe Energy and Giga Energy place mining containers at wellheads to monetize this gas, converting a waste product into economic value while reducing methane emissions (since the generators burn more completely than open flares).
   - **Stranded hydro**: Hydroelectric dams in remote areas (especially in Sichuan, Canada, Scandinavia, and Ethiopia) produce more power than the local grid can consume. Mining absorbs the surplus.
   - **Curtailed renewables**: Wind and solar farms sometimes produce more power than the grid can absorb, leading to curtailment (energy wasted). Mining can absorb curtailed power, improving the economics of renewable projects.

3. **Grid stabilization (demand response)**: Large mining operations can serve as controllable load — ramping down instantly when the grid needs power (during heat waves, cold snaps, or peak demand) and ramping up when there's surplus. ERCOT (the Texas grid operator) has embraced this model: Bitcoin miners in Texas are significant participants in demand response programs, earning payments for curtailment that supplement their mining revenue.

4. **Methane capture**: Methane (CH4) is 80x more potent as a greenhouse gas than CO2 over a 20-year period. Mining operations that capture methane from landfills, abandoned coal mines, or agricultural waste — and use it to generate electricity for mining — are converting a potent greenhouse gas into a much less potent one (CO2 from combustion). The net effect is a *reduction* in warming impact.

5. **Subsidizing renewable development**: By providing a buyer of last resort for renewable energy, mining improves the financial return of wind, solar, and hydro projects, potentially accelerating their construction. The marginal economics of a solar farm that can sell 100% of its output (including curtailed power, to miners) are better than a solar farm that curtails 15–20% of production.

### The Honest Assessment

Both sides are partially right. Bitcoin mining does consume a lot of energy, and some of that energy comes from fossil fuels. But the industry's energy mix is genuinely improving, the stranded-energy use case is real and growing, and the demand-response model creates genuine value for grid operators.

The strongest argument for Bitcoin mining's energy use is not environmental — it's economic: the network secures hundreds of billions of dollars in value, and the energy cost is the mechanism that makes that security trustless. Whether that security is "worth" the energy depends on whether you think a non-sovereign, censorship-resistant monetary network has value. For people living under authoritarian governments, currency controls, or hyperinflation, the answer is unambiguously yes.

---

## Part X: Selfish Mining

### The Theory

Selfish mining is a theoretical attack strategy described in a 2013 paper by Ittay Eyal and Emin Gun Sirer ("Majority is Not Enough: Bitcoin Mining is Vulnerable"). The core idea:

A miner who finds a valid block does not immediately broadcast it. Instead, they continue mining on top of their secret block, attempting to build a private chain that is longer than the public chain. If they succeed, they can release their private chain all at once, orphaning the honest miners' blocks and claiming all the rewards.

### How It Works

1. Selfish miner finds block N. Does not broadcast it.
2. Selfish miner immediately starts mining block N+1 on top of their secret block N.
3. Honest network is still mining on block N-1 (the previous public tip).
4. **Case A**: Honest network finds block N first. Selfish miner immediately releases their block N, creating a race. If the selfish miner's block propagates faster (because they have good network connectivity), honest miners build on the selfish miner's chain.
5. **Case B**: Selfish miner finds block N+1 before the honest network finds block N. Selfish miner now has a 2-block lead. They release both blocks, orphaning any honest block N that was in progress.

### The Math

Eyal and Sirer showed that selfish mining is profitable for miners with more than approximately 33% of network hashrate (under certain network propagation assumptions), not the 50% that naive analysis would suggest. With favorable network positioning (high connectivity, low latency), the threshold could be even lower.

The key insight: **selfish mining doesn't require 51% hashrate to be profitable.** It requires roughly 25–33% (depending on the attacker's network connectivity advantage, often parameterized as "gamma" — the fraction of the honest network the attacker can reach first during a block race).

### Why It Hasn't Happened (Probably)

1. **The honest mining assumption holds**: Miners are economically rational, and selfish mining is risky. If the selfish miner's private chain is outpaced by the honest chain, they lose all the mining revenue from those blocks — a catastrophic cost for a large operation.

2. **Detection**: Anomalous orphan rates and suspicious block timing would be detectable by network observers. A selfish miner would be identified relatively quickly, and the social/economic consequences (exclusion from pools, loss of business relationships) could outweigh the gains.

3. **No miner has 33% of hashrate**: As of 2026, no single mining entity has 33% of network hashrate. Pools control more than that, but the pool itself doesn't own the hashrate — individual miners do, and they can switch pools. A pool attempting to selfish mine would lose its members overnight.

4. **Opportunity cost**: The capital required to acquire 33% of hashrate would be in the billions of dollars. The expected return from selfish mining is only marginally better than honest mining. The risk-adjusted return is likely negative when you factor in detection, social costs, and the possibility of protocol-level responses.

### Protocol-Level Mitigations

Several defenses have been proposed:

- **Uniform tie-breaking**: Instead of miners always building on the first block they receive during a race, they randomly choose between competing blocks. This reduces the selfish miner's advantage from network positioning.
- **Timestamp analysis**: Detecting blocks with suspiciously old timestamps (suggesting they were withheld).
- **Publish-or-perish**: Penalizing blocks that arrive significantly after they could have been mined.

None of these have been implemented in Bitcoin Core, largely because the attack hasn't been observed at scale and the proposed mitigations introduce their own complexity and potential for abuse.

---

## Key Takeaways

1. **Mining evolved from CPU to ASIC in four years** (2009–2013). Each hardware generation rendered the previous one obsolete within 12–18 months. Satoshi predicted this trajectory before Bitcoin had a single user.

2. **Three companies dominate hardware manufacturing**: Bitmain (~55% share), MicroBT (~30%), and Auradine (emerging challenger). Modern ASICs hit 200–1,160 TH/s at 9–18 J/TH efficiency. Cooling technology (hydro, immersion) is the key differentiator for top-tier units.

3. **Electricity cost is the only variable that matters long-term.** Below $0.04/kWh is highly profitable; above $0.10/kWh is marginal at best. The difficulty adjustment ensures that mining profitability always regresses toward the cost of the cheapest available electricity.

4. **Pool centralization is real and dangerous.** The top 4 pools control ~70% of hashrate. More importantly, pool operators — not individual miners — construct block templates and decide which transactions are included. This is the most underappreciated centralization vector in Bitcoin.

5. **Stratum V2 with Job Declaration is the fix.** It returns template construction authority to individual miners, making pools into payout coordinators rather than block builders. Adoption is still early (Ocean, Braiins) but the protocol is production-ready.

6. **The halving cycle drives hardware obsolescence.** Every four years, the block subsidy halves and marginal miners are pushed offline. The fourth halving (April 2024) forced a fleet refresh to sub-20 J/TH hardware.

7. **Bitcoin miners are pivoting to AI/HPC.** Their existing power infrastructure, cooling capacity, and grid interconnections make mining facilities natural candidates for GPU compute hosting. This is reshaping the public miner landscape (MARA, IREN, Core Scientific).

8. **The environmental picture is nuanced.** 150+ TWh/year is a lot of energy. But 40–60% is renewable, stranded energy monetization is real, methane capture reduces emissions, and demand response provides genuine grid value. The strongest defense of mining's energy use is the security model it provides.

9. **Selfish mining is a real theoretical vulnerability** that is profitable above ~33% of hashrate, but has never been observed in practice due to detection risk, capital requirements, and the absence of any single entity with sufficient hashrate.

10. **The endgame is that transaction fees must replace the block subsidy.** With each halving, fees become a larger proportion of miner revenue. By the late 2030s, the block subsidy will be negligible. Bitcoin's long-term security depends on a robust fee market — a topic that connects directly to the block size debate, mempool policy, and the Ordinals/BRC-20 activity that has reinvigorated on-chain fee revenue.

---

**Previous:** [Chapter 8 — Satoshi's Timeline and Bitcoin Core History](08-satoshi-timeline.md) — Who created Bitcoin, who maintains it now, and who actually controls the protocol.

**Next:** [Chapter 11 — The Bitcoin Network](11-bitcoin-network.md) — How the P2P layer actually works: node types, block propagation, transaction relay, and why ~20,000 nodes keep Bitcoin decentralized.
