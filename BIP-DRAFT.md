BIP: ?
Layer: Applications
Title: Bitcoin Positional System
Authors: Bitoshi Blockamoto <TBD>
Status: Draft
Type: Specification
Assigned: ?
License: TBD
Version: 0.1.0

# Abstract

This BIP specifies the Bitcoin Positional System (BPS), a deterministic textual coordinate system for referring to confirmed Bitcoin structure by position.

BPS defines hierarchical references for blocks, transactions, transaction inputs, transaction outputs, and satoshi-denominated offsets within outputs. A basic positional coordinate is written from the most specific object toward its containing Bitcoin structure, for example `100.2.17.840000` for offset 100 of output 2 of transaction 17 in block 840000.

The system also permits block hashes and transaction IDs to replace their corresponding positional references where exact cryptographic identity is preferred.

BPS does not modify Bitcoin consensus, transaction serialization, or peer-to-peer rules. It standardizes a naming and resolution model over data that already exists in confirmed Bitcoin blocks.

# Copyright

A BIP-compatible license must be selected before submission.

The current BIP process requires an acceptable SPDX license expression. This draft intentionally leaves the legal choice to the author rather than selecting one implicitly.

# Motivation

Bitcoin objects have canonical cryptographic identifiers, but confirmed chain data also has deterministic position.

A confirmed transaction has:

- a containing block;
- an index within that block.

An output has:

- a containing transaction;
- an index within that transaction.

A position inside an output can be expressed as an offset.

Applications that want to refer to these objects currently use txids, block hashes, application-specific indexes, explorer URLs, or ad hoc positional grammars.

A standardized positional system has several useful properties:

1. positions can be shorter and more human-readable than 256-bit hashes;
2. positions naturally express containment;
3. each level can be independently addressable;
4. sparse or light clients can use a coordinate to request only the historical Bitcoin data required for resolution;
5. protocols can share references without inventing incompatible positional syntax.

BPS complements rather than replaces cryptographic hashes. Hashes identify content before confirmation; positions become available after confirmed placement in chain history.

# Specification

## Canonical integers

Numeric positions MUST be non-negative base-10 integers.

They MUST NOT include a sign or leading zeroes, except for the value `0`.

All indexes are zero-based.

## Hash tokens

A block hash or transaction ID MUST be represented as exactly 64 lowercase hexadecimal characters.

An exactly 64-character hexadecimal token is interpreted as a hash before it is interpreted as a decimal number.

## Block reference

```text
<block-ref>
```

where `block-ref` is either:

- a decimal block height; or
- a block hash.

Examples:

```text
840000
<blockhash>
```

A height is relative to the resolver's selected chain. A block hash identifies one exact block.

## Transaction reference

```text
<tx-ref>.<block-ref>
```

where `tx-ref` is either:

- the zero-based transaction index within the referenced block; or
- a txid.

Examples:

```text
17.840000
<txid>.840000
17.<blockhash>
<txid>.<blockhash>
```

If a txid is supplied, the resolver MUST verify that the exact transaction is present in the referenced block. It MUST NOT silently resolve the txid in a different block.

## Output reference

```text
<output-index>.<tx-ref>.<block-ref>
```

Example:

```text
2.17.840000
```

The referenced output MUST exist in the resolved transaction.

## Input reference

```text
i<input-index>.<tx-ref>.<block-ref>
```

Example:

```text
i2.17.840000
```

The `i` prefix distinguishes an input position from the otherwise bare numeric output branch.

The referenced input MUST exist in the resolved transaction.

## Satpoint reference

```text
<offset>.<output-index>.<tx-ref>.<block-ref>
```

Example:

```text
100.2.17.840000
```

The offset is measured in satoshis from the beginning of the output and MUST be less than the output value.

This coordinate describes a local position within an output. BPS does not assign a global ordinal number to the satoshi at that position and does not track it through later spends.

## Resolution order

Coordinates are serialized leaf-first but resolved container-first.

For:

```text
100.2.17.840000
```

the resolver:

1. resolves block 840000;
2. selects transaction 17;
3. selects output 2;
4. validates offset 100.

This structure makes truncation meaningful:

```text
840000
17.840000
2.17.840000
100.2.17.840000
```

## Equivalent forms

Once a height and transaction position have been resolved to `BLOCKHASH` and `TXID`, all of the following may refer to the same transaction:

```text
17.840000
TXID.840000
17.BLOCKHASH
TXID.BLOCKHASH
```

The corresponding substitution applies to output and satpoint coordinates.

## Reorganizations

A coordinate containing a block height is chain-relative.

A reorganization may cause a purely positional height-based coordinate to resolve to a different object or no object.

A block-hash coordinate is anchored to one exact block and therefore does not rebind under an active-chain reorganization.

Applications MUST choose confirmation-depth and block-anchoring policies appropriate to their use case.

## Namespace and transport

The BPS coordinate is independent of URI schemes, DNS, or application namespaces.

Implementations MAY expose a qualified name such as:

```text
100.2.17.840000.bitcoin
```

The `.bitcoin` suffix is not part of the core coordinate and this BIP does not allocate a DNS top-level domain.

This BIP also does not claim the `bitcoin:` URI scheme.

# Rationale

## Leaf-first order

Earlier positional proposals have commonly serialized the outer container first.

BPS uses leaf-first order so the outer Bitcoin context remains on the right:

```text
offset.output.transaction.block
```

This is convenient for hierarchical namespace systems and makes progressively shorter suffixes valid higher-level references.

## Input prefix

Bare numeric levels follow the natural block → transaction → output → offset hierarchy.

Inputs are a sibling branch of outputs, so they require an explicit type marker. The `i` prefix preserves compactness while preventing ambiguity with an output index.

## Hash substitution

BPS does not force applications to choose between human positional references and cryptographic identities.

Where an exact block or transaction identity is known, the corresponding hash may replace its positional token while the containment structure remains unchanged.

## No unconfirmed positional reference

A transaction has a txid before it is mined but has no block position until confirmation.

BPS therefore intentionally does not define a positional reference for unconfirmed transactions.

This reflects a useful division:

- hash identity is available prospectively;
- positional identity is available retrospectively.

# Backward Compatibility

BPS changes no Bitcoin consensus or transaction rules.

Existing Bitcoin software may ignore BPS entirely.

An implementation can resolve BPS coordinates using existing block data and ordinary transaction ordering.

Optional future API/RPC or peer-service support would be separate implementation work and is not required by this BIP.

# Security Considerations

## Weak positional binding

A pure height/index reference is not cryptographically bound to transaction contents.

This is an inherent property of positional references and is shared with earlier work such as BIP 136.

Applications that require immutable historical identity SHOULD use block-hash anchoring and MAY use txid substitution.

## Reorganizations

Height-based references can rebind after a chain reorganization.

Applications MUST NOT treat a chain-relative coordinate as equivalent to a cryptographic commitment without additional anchoring.

## Explicit hash membership

When a txid is used with explicit block context, a resolver MUST fail if the transaction is absent from that block.

It MUST NOT search elsewhere and silently substitute another location.

## Bounds

Resolvers MUST reject input indexes, output indexes, transaction indexes, and output offsets that are outside the referenced container.

# Reference Implementation

Gateway is the first BPS implementation:

https://github.com/Blockamoto/gateway

The current Gateway parser implements positional transaction, input, output and satpoint coordinates, plus selected txid-located forms and an experimental inscription extension.

Implementation notes and current conformance gaps are maintained in:

https://github.com/Blockamoto/bitcoin-positional-system/blob/main/IMPLEMENTATION.md

# Prior Art

## BIP 136 — TxRef

BIP 136 standardizes a Bech32m encoding of block height plus transaction index and optional output index.

https://github.com/bitcoin/bips/blob/master/bip-0136.mediawiki

BPS generalizes the underlying positional idea into an explicit recursive hierarchy instead of an opaque encoded identifier.

## Ordinal Numbers draft — slot

Casey Rodarmor's 2022 Ordinal Numbers draft defines a slot as:

```text
block-height.transaction-index.output-index.offset
```

https://www.mail-archive.com/bitcoin-dev%40lists.linuxfoundation.org/msg10989.html

BPS contains the same underlying positional dimensions but reverses the textual order for hierarchical resolution and does not include ordinal numbering in the core proposal.

## BIP 337 — Compressed Transactions

BIP 337 uses block height and an index within the block as an optional compression representation for transaction outpoints.

https://github.com/bitcoin/bips/blob/master/bip-0337.mediawiki

BPS does not propose transaction compression or consensus changes. BIP 337 is evidence that positional references are independently useful as a machine-level primitive.

# Extensions

Non-Bitcoin-native concepts such as inscription addressing are outside this BIP.

An experimental BPS Ordinals extension is maintained separately in the project repository.
