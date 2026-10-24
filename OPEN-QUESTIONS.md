# Open Questions

These are intentionally unresolved in blocators 0.1.

The purpose of this file is to keep uncertain design choices visible instead of allowing implementation accidents to become standards.

## 1. Should the base specification define a checksummed encoding?

The raw positional grammar is intentionally transparent:

```text
100.2.17.840000
```

BIP 136 instead wraps position in Bech32m, gaining checksums and explicit network information at the cost of hiding the hierarchy.

Possible direction:

- keep raw blocators as the canonical coordinate;
- optionally define a separate checksummed transport encoding later.

Do not add a checksum merely to make blocators look more protocol-like. It should solve a concrete transcription or transport problem.

## 2. How should network context be serialized when it must travel with the coordinate?

blocators core currently requires the surrounding context to establish mainnet, testnet, signet, or regtest.

Possible carriers include:

- API parameters;
- protocol negotiation;
- a namespace qualifier;
- a future checksummed encoding.

The core coordinate should remain unchanged unless self-contained network identity proves necessary.

## 3. Should `.bitcoin` ever become normative?

Current position:

- `.bitcoin` is a useful Gateway namespace qualifier;
- it is not part of the blocator;
- it is not a DNS allocation.

Questions:

- Should a blocators companion specification standardize the qualified name?
- Should different networks have different suffixes?
- Should blocators remain completely namespace-neutral?

## 4. Block hash syntax and DNS incompatibility

A block hash is a 64-character hexadecimal token.

DNS labels are limited to 63 octets.

Therefore hash-anchored qualified blocators names cannot always be represented as literal DNS hostnames.

Current direction:

- blocators remains transport-neutral;
- DNS-compatible bridges support only forms that fit DNS;
- native resolvers carry full hash forms losslessly.

## 5. Input-relative offsets

Candidate:

```text
<offset>.i<input-index>.<tx-ref>.<block-ref>
```

Possible meaning:

> offset N within the value entering through transaction input I.

Questions:

- Is this useful enough to standardize?
- Is the offset defined over the referenced previous output value?
- How should coinbase be treated?
- Does this create a useful primitive independent of ordinal sat-flow rules?

No syntax should become normative until these semantics are exact.

## 6. Canonical Ordinals reveal syntax

Gateway currently accepts both:

```text
12i0.792435
12.i0.792435
```

Current preference:

```text
12i0.792435
```

because the `iN` suffix is part of the inscription occurrence identity attached to the transaction reference.

Before standardization, confirm that this remains unambiguous alongside typed input syntax.

## 7. Output-scoped inscription ordering

Candidate:

```text
2i0.17.840000
```

This is only useful if "inscription 0 in output 2" has one deterministic ordering.

Possible orderings include:

- reveal order;
- satpoint order;
- inscription ID order;
- canonical inscription-number order;
- protocol-specific traversal order.

The extension must pick one or reject this locator.

## 8. Satpoint-scoped inscription ordering

Candidate:

```text
100i1.2.17.840000
```

This may be needed when multiple inscriptions are associated with one satpoint.

It requires exact historical Ordinal-state semantics and is therefore outside the core blocators proposal.

## 9. Confirmation guidance

BIP 136 recommends restrictions and warnings for shallow positional references because reorgs can rebind them.

blocators must decide whether to:

- prescribe confirmation thresholds;
- give non-normative guidance;
- leave confirmation policy entirely to applications.

Because use cases have different risk models, hard-coded confirmation counts may be inappropriate.

## 10. Integer bounds

The abstract grammar uses non-negative decimal integers.

Implementations have finite integer widths.

Questions:

- Should blocators define explicit maximum values?
- Should bounds follow Bitcoin's actual serialized field limits where available?
- Should parsers accept arbitrary-precision values and fail only during resolution?

The specification should avoid architecture-dependent behavior.

## 11. Canonical display of hashes

blocators should explicitly confirm that txids and block hashes use the conventional Bitcoin user-facing hexadecimal display order used by Bitcoin Core RPCs and block explorers.

This is likely obvious to implementers but should not remain implicit in a final standard.

## 12. Active-chain vs known-block hash resolution

For a block-hash coordinate:

```text
17.<blockhash>
```

should a resolver:

- resolve any known valid block, including stale branches; or
- resolve only blocks on the selected active chain unless explicitly configured otherwise?

Current intuition:

- hash form identifies the exact historical block;
- active-chain membership should be returned as metadata, not silently redefine identity.

This needs explicit consensus before submission.

## 13. blocators and Bitcoin Core RPC

Possible future Core surface:

```text
resolveblocator "100.2.17.840000"
```

This is not required for the Applications-layer BIP.

Questions:

- separate API/RPC BIP or implementation PR?
- return raw bytes, decoded JSON, or both?
- should missing pruned block data trigger retrieval or simply return unavailable?

## 14. blocators and peer retrieval

Bitcoin on Demand makes blocators useful on sparse historical storage.

That is a separate Peer Services problem.

blocators should not grow peer protocol rules merely because Gateway implements both layers.

A later proposal can define how a peer advertises and serves resolvable blocators history.

