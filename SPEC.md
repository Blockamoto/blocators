# Bitcoin Positional System — Core Specification

Status: Experimental draft  
Version: 0.1

## 1. Purpose

Bitcoin Positional System (BPS) defines a deterministic textual coordinate system for confirmed Bitcoin structure.

BPS describes positions that already exist in Bitcoin. It does not alter consensus or introduce new blockchain objects.

The core hierarchy is:

```text
block
transaction.block
output.transaction.block
offset.output.transaction.block
```

Inputs are a typed branch of the transaction level:

```text
i<input>.transaction.block
```

## 2. Terminology

- **block reference** — a block height or block hash.
- **transaction reference** — a transaction index within a block or a transaction ID.
- **output index** — the zero-based `vout` position within a transaction.
- **input index** — the zero-based `vin` position within a transaction.
- **offset** — a zero-based satoshi-denominated offset within an output.
- **satpoint** — an outpoint plus an offset. BPS can express a satpoint position without assigning a global ordinal number to the satoshi at that position.
- **coordinate** — a BPS string without a namespace qualifier.
- **qualified name** — a BPS coordinate with an implementation-defined namespace qualifier such as `.bitcoin`.

## 3. Canonical lexical rules

### 3.1 Decimal integers

Indexes, heights, and offsets:

- MUST be non-negative decimal integers.
- MUST NOT contain a sign.
- MUST NOT contain leading zeroes except for the single value `0`.
- are interpreted in base 10.

Examples:

```text
0       valid
17      valid
840000  valid
00      invalid
017     invalid
+17     invalid
-1      invalid
```

### 3.2 Hashes

Transaction IDs and block hashes:

- MUST be exactly 64 hexadecimal characters.
- MUST be serialized in lowercase in canonical form.
- are parsed as hashes before decimal integers when a token is exactly 64 hexadecimal characters.

This precedence prevents an all-numeric 64-character hash from being interpreted as a decimal position.

### 3.3 Separator

The period character `.` separates hierarchy levels.

No whitespace is permitted inside a canonical coordinate.

## 4. Reference tokens

```text
decimal      = "0" / nonzero-digit *digit
hash256      = 64HEXDIG
block-ref    = decimal / hash256
tx-ref       = decimal / hash256
input-ref    = "i" decimal
```

A resolver determines whether `block-ref` is a height or block hash, and whether `tx-ref` is a transaction index or txid, using the lexical rules above.

## 5. Core coordinate forms

### 5.1 Block

```text
<block-ref>
```

Examples:

```text
0
840000
0000000000000000000320283a032748cef8227873ff4872689bf23f1cda83a5
```

A decimal block reference is a block height on the resolver's selected chain.

A hash block reference identifies the block with that exact hash.

### 5.2 Transaction

```text
<tx-ref>.<block-ref>
```

Examples:

```text
0.0
17.840000
<txid>.840000
17.<blockhash>
<txid>.<blockhash>
```

If `tx-ref` is decimal, it is the zero-based position of the transaction in the referenced block.

If `tx-ref` is a txid, the resolver MUST verify that the exact transaction occurs in the referenced block. It MUST NOT silently fall back to a global txid lookup if the transaction is absent from that block.

### 5.3 Output

```text
<output-index>.<tx-ref>.<block-ref>
```

Examples:

```text
0.0.0
2.17.840000
2.<txid>.<blockhash>
```

The output index MUST be less than the number of outputs in the resolved transaction.

### 5.4 Input

```text
i<input-index>.<tx-ref>.<block-ref>
```

Examples:

```text
i0.0.0
i2.17.840000
i2.<txid>.<blockhash>
```

The input index MUST be less than the number of inputs in the resolved transaction.

An input coordinate identifies the input's structural position in the spending transaction. It is distinct from the previous output that the input references.

Coinbase inputs are valid transaction inputs even though they do not reference a normal previous outpoint.

### 5.5 Satpoint / output offset

```text
<offset>.<output-index>.<tx-ref>.<block-ref>
```

Examples:

```text
0.0.0.0
100.2.17.840000
100.2.<txid>.<blockhash>
```

The offset MUST be less than the output value measured in satoshis.

BPS uses this as a local position within an output. It does not assign or require a global ordinal number and does not, by itself, track that satoshi through later spends.

## 6. Resolution

Resolution proceeds from the outermost Bitcoin container toward the requested inner object, even though the textual coordinate is written leaf-first.

For:

```text
100.2.17.840000
```

a resolver:

1. resolves block height `840000`;
2. selects transaction index `17`;
3. selects output index `2`;
4. validates offset `100` against that output's value.

This right-to-left containment model makes truncation meaningful:

```text
840000              block
17.840000           transaction
2.17.840000         output
100.2.17.840000     satpoint
```

## 7. Equivalent references

Different BPS coordinates may resolve to the same Bitcoin object.

After resolving block height `840000` to `BLOCKHASH` and transaction index `17` to `TXID`, these transaction coordinates are equivalent while their referenced facts remain true:

```text
17.840000
TXID.840000
17.BLOCKHASH
TXID.BLOCKHASH
```

The same substitution applies below the transaction level:

```text
2.17.840000
2.TXID.BLOCKHASH

100.2.17.840000
100.2.TXID.BLOCKHASH
```

BPS does not define a separate hash for an input or output. Their identities remain positions within the containing transaction.

## 8. Chain context and reorganizations

Height-based references are chain-relative.

```text
17.840000
```

means transaction index 17 in the block occupying height 840000 on the resolver's selected chain at resolution time.

A reorganization can therefore cause a purely positional height-based coordinate to:

- resolve to a different object;
- cease to resolve; or
- later resolve again to another object.

Hash-anchored block references do not rebind in this way:

```text
17.<blockhash>
```

always refers to transaction position 17 within that exact block, if such a transaction exists.

A txid combined with a height is safer against accidental transaction rebinding because the resolver MUST verify membership:

```text
<txid>.840000
```

If a reorganization replaces the block at 840000 and that txid is absent, resolution fails rather than selecting the new transaction at the old position.

Applications MUST choose confirmation and anchoring policy appropriate to their security model.

## 9. Confirmed-data scope

BPS core coordinates are defined over confirmed block structure.

A transaction has a txid before confirmation but does not have a block position until a miner places it in a block.

BPS therefore complements rather than replaces hash identities:

- hashes can identify objects before chain position is known;
- positions identify objects after chain placement is known.

BPS does not define a positional identifier for an unconfirmed transaction.

## 10. Namespace qualification

The BPS coordinate is independent of transport and namespace.

An implementation MAY expose a qualified form:

```text
100.2.17.840000.bitcoin
```

The `.bitcoin` suffix is not part of the core coordinate grammar.

BPS does not allocate a DNS top-level domain. Some valid BPS hash tokens are 64 characters long and therefore exceed the 63-octet DNS label limit.

BPS also does not claim the `bitcoin:` URI scheme.

Transports and APIs MAY wrap a BPS coordinate in their own syntax, for example:

```text
bps.resolve("100.2.17.840000")
god://100.2.17.840000.bitcoin
```

Such wrappers are outside the BPS core specification.

## 11. Error behavior

A conforming resolver MUST reject a coordinate when:

- a referenced block cannot be resolved;
- a transaction index is outside the block's transaction list;
- a txid is not present in the explicitly referenced block;
- an input or output index is out of range;
- an offset is greater than or equal to the output value;
- the coordinate violates canonical lexical rules;
- a token is ambiguous under an implementation-specific extension and cannot be resolved without guessing.

Resolvers MUST NOT silently substitute another object when an explicit hash identity fails membership verification.

## 12. Non-goals

BPS core does not define:

- Bitcoin consensus changes;
- transaction serialization changes;
- a replacement for txids or block hashes;
- a global satoshi numbering scheme;
- satoshi lineage across spends;
- inscription semantics;
- wallet ownership;
- address history;
- peer-to-peer retrieval;
- a DNS allocation.

Those systems may use BPS coordinates without becoming part of BPS.

## 13. Extensions

Extensions MAY add typed branches to BPS if they:

1. preserve unambiguous parsing;
2. define exact containment and resolution semantics;
3. do not change the meaning of existing core coordinates;
4. specify failure behavior;
5. state whether the referenced object is immutable, chain-relative, or state-relative.

The first proposed extension is inscription addressing, documented in [extensions/ordinals.md](extensions/ordinals.md).

## 14. Open item: input-relative offsets

A possible extension has the shape:

```text
<offset>.i<input-index>.<tx-ref>.<block-ref>
```

This could identify a satoshi-denominated position entering a transaction through a particular input.

It is intentionally **not** normative in BPS 0.1. Unlike an output satpoint, input-relative offset semantics must account for the value and ordering of previous outputs and special coinbase behavior. The rules should be specified before the syntax is standardized.
