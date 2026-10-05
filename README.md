# Bitcoin Positional System (BPS)

**A positional addressing system for Bitcoin.**

Bitcoin already gives blocks, transactions, inputs, outputs, and positions within outputs an ordered structure. BPS gives that structure a compact, deterministic textual coordinate system.

The core idea is simple:

```text
block
transaction.block
output.transaction.block
offset.output.transaction.block
```

For example:

```text
840000
17.840000
2.17.840000
100.2.17.840000
```

Each additional component moves inward through already-existing Bitcoin structure. All numeric indexes are zero-based.

BPS is a naming and resolution system. It does **not** change Bitcoin consensus, replace transaction IDs, define global satoshi identity, or require a new transaction format.

## Why

Bitcoin's canonical cryptographic identifiers are excellent content identities, but they are not always the most convenient locators for confirmed chain data.

A block already has a height. A transaction already has a position in that block. An output already has a position in that transaction. A position within an output can already be expressed as an offset.

BPS standardizes those relationships as a hierarchy.

## Address as retrieval path

A transaction ID identifies transaction content, but it does not tell a sparse client which block contains that transaction. Without a transaction-to-block index, the client cannot derive the block location from the txid alone.

A positional address carries that missing retrieval context.

For example:

```text
17.840000
```

tells a resolver to:

1. resolve block height `840000` to its block hash using the selected chain/header context;
2. request that block from a peer able to serve it;
3. select transaction index `17` from the returned block.

The same idea extends naturally to outputs and offsets.

This means a BPS address can function as both a human-shareable reference and a deterministic retrieval plan. A user can copy, paste, publish, or send a coordinate without requiring the recipient to already maintain a full transaction-location index.

This is useful for:

- human-readable references to confirmed Bitcoin structure;
- deterministic navigation and linking;
- light or sparse clients that retrieve historical data on demand;
- APIs, explorers, developer tooling, and protocol references;
- protocols that need to point into Bitcoin without inventing their own positional grammar.

## Coordinate vs qualified name

The BPS coordinate is the bare coordinate:

```text
100.2.17.840000
```

A surrounding namespace may qualify it:

```text
100.2.17.840000.bitcoin
```

The `.bitcoin` suffix is a namespace qualifier used by implementations such as Gateway. It is not required when the surrounding API or protocol has already established BPS context, and BPS does not claim a DNS TLD.

BPS also does not claim the `bitcoin:` URI scheme, which is already used by Bitcoin payment URI standards.

## Documents

- [SPEC.md](SPEC.md) — normative BPS core grammar and semantics
- [BIP-DRAFT.md](BIP-DRAFT.md) — BIP-shaped draft for the Bitcoin-native core
- [PRIOR-ART.md](PRIOR-ART.md) — TxRef, early Ordinal slot notation, compressed transactions, and related work
- [IMPLEMENTATION.md](IMPLEMENTATION.md) — current Gateway implementation and conformance notes
- [TEST-VECTORS.md](TEST-VECTORS.md) — initial shared conformance vectors
- [extensions/ordinals.md](extensions/ordinals.md) — non-core Ordinals/inscription extension
- [OPEN-QUESTIONS.md](OPEN-QUESTIONS.md) — unresolved design questions

## Scope

The core BPS proposal addresses confirmed Bitcoin structure:

```text
Block
└── Transaction
    ├── Input
    └── Output
        └── Offset
```

Ordinals and inscriptions are intentionally outside the core BPS proposal. They can reuse BPS coordinates as an extension without requiring BPS users or implementers to adopt ordinal theory.

## Status

**Experimental draft.**

Gateway is the first implementation and currently supports a substantial subset of the grammar. The specification is expected to change before any formal BIP submission.

## One-sentence definition

> BPS names positions in Bitcoin's confirmed structure.

