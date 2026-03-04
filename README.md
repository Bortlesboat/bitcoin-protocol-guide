# Bitcoin Protocol Guide

A deep dive into Bitcoin's protocol internals — from UTXOs to Taproot to the Ordinals debate. Written from a "confused → understanding" perspective, with real transaction examples you can verify yourself.

No price talk. No trading. Pure protocol.

## Table of Contents

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 01 | [UTXO Model](01-utxo-model.md) | How Bitcoin tracks ownership without accounts |
| 02 | [Script System](02-script-system.md) | Bitcoin's stack-based programming language |
| 03 | [SegWit](03-segwit.md) | Witness data, weight units, and why it changed everything |
| 04 | [Taproot](04-taproot.md) | Schnorr signatures, MAST, and privacy upgrades |
| 05 | [Ordinals & Inscriptions](05-ordinals-inscriptions.md) | How NFTs and tokens work on Bitcoin |
| 06 | [The BIP-110 Debate](06-bip110-debate.md) | The "spam war" — what's at stake |
| 07 | [Node Setup](07-node-setup.md) | Running Bitcoin Knots/Core on Windows |
| 08 | [Satoshi's Timeline & Core History](08-satoshi-timeline.md) | From genesis block to current maintainers — who built Bitcoin and who stewards it now |
| 09 | [Wallets & Self-Custody](09-wallets-self-custody.md) | Key generation, hardware wallets, multisig, PSBTs, seed storage, and inheritance planning |
| 10 | [Mining & Hardware](10-mining-hardware.md) | From CPU to 1,160 TH/s ASICs — mining economics, pools, Stratum V2, and the AI pivot |
| 11 | [The Bitcoin Network](11-bitcoin-network.md) | P2P layer: node types, block propagation, compact blocks, eclipse attacks, Tor/I2P |
| 12 | [The Future of Bitcoin](12-future-bitcoin.md) | CTV, OP_CAT, covenants, Silent Payments, BitVM, RGB, Ark, and the ossification debate |

### Bonus

- **[Satoshi's Words](satoshi-quotes.md)** — 19 verified, verbatim quotes from the white paper and BitcoinTalk forums
- **[CLI Commands](cli-commands.md)** — Useful `bitcoin-cli` commands for exploring the blockchain
- **[Examples](examples/)** — Real decoded transactions from mempool.space
  - [The Pizza Transaction](examples/pizza-tx.md) — 10,000 BTC for two pizzas (P2PKH)
  - [First BRC-20 Deploy](examples/first-brc20.md) — The `ordi` token (Taproot inscription)
  - [Large Image Inscription](examples/ordinal-image.md) — Witness data "abuse" in practice

## Philosophy

Every claim in this guide can be verified with a Bitcoin node and `bitcoin-cli`. The goal is to understand *why* Bitcoin works the way it does, not just *what* it does. Each chapter builds on the last:

```
UTXOs → Script → SegWit → Taproot → Ordinals → BIP-110 → Node Setup → History
    → Self-Custody → Mining → Network → Future
```

Chapters 1-8 build linearly. Chapters 9-12 expand the picture: how to hold your keys, how mining works, how the network operates, and what's being built next.

## Prerequisites

- Basic understanding of public/private key cryptography
- A Bitcoin node (Core or Knots) — see [Chapter 07](07-node-setup.md) for setup
- Willingness to read hex and JSON

## How to Use This Guide

1. **Read sequentially** — each chapter builds on the previous
2. **Run the CLI commands** — verify everything yourself
3. **Decode the example transactions** — seeing real data makes it click
4. **Use mempool.space** as a visual companion — paste any txid to see it rendered

## Tools

Query your own node with **[bitcoinlib-rpc](https://github.com/Bortlesboat/bitcoinlib-rpc)** — a typed Python wrapper for Bitcoin Core RPC:

```bash
pip install bitcoinlib-rpc
bitcoin-mempool    # Fee buckets, congestion analysis
bitcoin-block 939290  # Pool ID, SegWit/Taproot adoption
bitcoin-tx <txid>  # Full decode + inscription detection
```

See the [Tools section](tools/overview.md) for details.

## Contributing

Found an error? Open an issue or PR. This is a living document.

## License

[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) — Share and adapt with attribution.
