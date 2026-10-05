# Prior Art and Historical Lineage

BPS does not claim that positional references to Bitcoin structure are a new idea.

Several earlier proposals independently demonstrate that block position is a useful primitive. BPS differs by making the positional hierarchy itself the primary, recursively addressable object.

## 1. BIP 136 — TxRef

**BIP 136: Bech32 Encoded Tx Position References**

https://github.com/bitcoin/bips/blob/master/bip-0136.mediawiki

BIP 136 defines a confirmed transaction reference using:

- block height;
- transaction index within the block;
- optionally an output index.

It then encodes those values into a Bech32m TxRef.

BIP 136 is the closest formal Bitcoin precedent for the claim that a confirmed transaction can be usefully referenced by chain position instead of only by txid.

### Shared ground

BIP 136 and BPS both recognize:

- block height + transaction index as a valid transaction locator;
- output index as a further positional refinement;
- reorganization risk for chain-relative positional references;
- the distinction between cryptographic identity and positional location.

### Difference

TxRef encodes selected positional data into a compact checksummed identifier.

BPS makes the hierarchy itself explicit and independently addressable:

```text
840000
17.840000
2.17.840000
100.2.17.840000
```

BPS also defines typed input coordinates, a satpoint/output-offset level, hash/position equivalence rules, and resolver semantics.

## 2. BTCR DID usage of TxRef

The BTCR decentralized identifier method adopted BIP 136 TxRef as the transaction-position component of a DID.

Historical specification:

https://github.com/w3c-ccg/didm-btcr

This is useful evidence that positional Bitcoin references have been used outside the original TxRef proposal as application-level identifiers.

It also highlights a property relevant to BPS: transaction position is known only after confirmation, while txid identity exists before confirmation.

## 3. Casey Rodarmor's 2022 Ordinal Numbers draft — "slot"

On 22 February 2022, Casey Rodarmor posted a draft BIP for Ordinal Numbers to the bitcoin-dev mailing list.

Archive:

https://www.mail-archive.com/bitcoin-dev%40lists.linuxfoundation.org/msg10989.html

The draft defines a **slot** as a dotted positional coordinate:

```text
block-height.transaction-index.output-index.offset
```

Example from the draft:

```text
83.0.1.100
```

This is extremely close to the underlying four-dimensional coordinate represented by BPS.

### Shared ground

Both systems identify:

- block position;
- transaction position within the block;
- output position within the transaction;
- an offset within the output.

### Difference

The slot notation is written container-first:

```text
block.transaction.output.offset
```

BPS is written leaf-first:

```text
offset.output.transaction.block
```

That reversal is deliberate. BPS is a hierarchical namespace in which the rightmost component is the outer container and each leftward component narrows the target.

Every truncation is therefore a valid higher-level reference:

```text
83
0.83
1.0.83
100.1.0.83
```

The early slot notation appeared inside ordinal theory and did not survive as the normal user-facing addressing grammar of current Ord. Current Ord commonly uses txid/outpoint-based satpoints and inscription IDs instead.

BPS does not adopt ordinal numbering as part of the core system.

## 4. BIP 337 — Compressed Transactions

**BIP 337: Compressed Transactions**

https://github.com/bitcoin/bips/blob/master/bip-0337.mediawiki

BIP 337 is an API/RPC-layer compression proposal. One of its principal compression techniques replaces an ordinary 36-byte `txid + vout` outpoint representation with a shorter positional representation based on:

- block height;
- an index within that block.

The proposal decompresses back into an ordinary Bitcoin transaction before consensus validation.

### Relevance to BPS

BIP 337 independently demonstrates that Bitcoin position is useful not only for human references but also as a machine compression primitive.

BPS does **not** propose using positional references as consensus outpoints.

The relationship is conceptual:

> If multiple independent systems repeatedly derive block position as a useful reference primitive, standardizing the positional model itself may be useful.

## 5. BIP 321 — Bitcoin URI scheme

**BIP 321: URI Scheme**

https://github.com/bitcoin/bips/blob/master/bip-0321.mediawiki

BIP 321 standardizes the `bitcoin:` URI scheme for payment requests and related wallet interactions.

For that reason, BPS does not claim `bitcoin:` as its URI scheme.

A BPS implementation can carry coordinates through another protocol or API and may use a namespace qualifier such as `.bitcoin` where appropriate.

## 6. What BPS adds

The novelty claim for BPS should be narrow.

BPS does **not** claim invention of positional Bitcoin references.

BPS proposes to standardize the following combination:

1. a recursive hierarchy covering block, transaction, output, input, and output offset;
2. a leaf-first textual form suitable for hierarchical namespace resolution;
3. meaningful truncation at every containment level;
4. exact resolution and failure semantics;
5. explicit equivalence between positional and hash-anchored forms;
6. separation of the coordinate from namespace and transport;
7. an extension model that allows systems such as Ordinals to reuse BPS without entering the Bitcoin-native core.

## 7. Historical framing

A conservative historical description is:

> TxRef standardized positional transaction references. Early ordinal theory described a full block/transaction/output/offset coordinate. Compressed Transactions reused position as a data-compression primitive. BPS treats Bitcoin position itself as the object to standardize and exposes the hierarchy directly as a resolvable namespace.

That framing credits the lineage without overstating novelty.
