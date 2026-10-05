# Reference Implementation — Gateway

Gateway is the first implementation of the ideas that became BPS.

Repository:

https://github.com/Blockamoto/gateway

Implementation snapshot reviewed for this document:

```text
6f4cfa24e8eaee5022dd4d8eafb8241f55a23fe1
```

This document describes what Gateway currently implements and where the BPS 0.1 specification intentionally goes beyond today's parser.

## 1. Current positional grammar

Gateway's `coordinates.go` currently implements these bare coordinate forms.

### Transaction

```text
<tx-index>.<block-height>
```

Example:

```text
0.0
```

### Output

```text
<vout>.<tx-index>.<block-height>
```

Example:

```text
0.0.0
```

### Input

```text
i<vin>.<tx-index>.<block-height>
```

Example:

```text
i0.0.0
```

### Satpoint

```text
<offset>.<vout>.<tx-index>.<block-height>
```

Example:

```text
0.0.0.0
```

Gateway validates that the offset lies inside the selected output's satoshi value.

## 2. Current inscription extension

Gateway currently accepts a positional reveal coordinate in either of these forms:

```text
<tx-index>i<inscription-index>.<block-height>
<tx-index>.i<inscription-index>.<block-height>
```

Examples:

```text
12i0.792435
12.i0.792435
```

The compact first form is the preferred candidate for a future canonical Ordinals extension. The dotted form remains useful as a compatibility alias in Gateway.

Gateway resolves the referenced transaction inside the authenticated block and extracts the requested inscription envelope from Bitcoin witness data. It does not require an Ord server for that resolution path.

## 3. Current hash-located forms

Gateway's `hash_coordinates.go` currently accepts an exact transaction identity within an explicit block height:

```text
<txid>.<block-height>
```

and inscription forms:

```text
<txid>i<inscription-index>.<block-height>
<txid>.i<inscription-index>.<block-height>
```

The resolver verifies that the exact txid occurs in the explicitly fetched block. It does not fall back to another transaction if the txid is absent.

A 64-character hexadecimal transaction token takes precedence over decimal-position parsing, including hashes made entirely of decimal digits.

## 4. Namespace form

Gateway accepts `.bitcoin` qualified resources, for example:

```text
0.0.bitcoin
i1.12.792435.bitcoin
12i0.792435.bitcoin
<txid>.792435.bitcoin
```

Gateway also accepts native activation forms such as:

```text
god://<resource>.bitcoin
```

BPS treats the namespace and transport wrapper as separate from the bare coordinate.

## 5. DNS bridge behavior

A valid BPS resource is not necessarily a legal DNS hostname.

A txid is 64 hexadecimal characters, while a DNS label is limited to 63 octets.

Gateway therefore does not force hash-located BPS resources through its DNS-style bridge. Hash resources are carried through native activation or a local resolver URL instead.

This behavior supports the BPS rule that namespace qualification is not equivalent to DNS compliance.

## 6. Block retrieval and verification

Gateway's coordinate resolution path:

1. identifies the requested block from the outer coordinate;
2. fetches and decodes that block;
3. requires integrity-checked Bitcoin evidence;
4. selects the requested transaction by index or exact txid;
5. selects the requested input/output/offset;
6. returns provenance describing the block source and verification state.

Gateway can obtain block bytes from ordinary Bitcoin peers using normal Bitcoin P2P block transfer after locating the block.

This is an implementation of BPS resolution over Bitcoin on Demand, not part of the BPS grammar itself.

## 7. Differences between Gateway today and BPS 0.1 target

BPS 0.1 intentionally specifies a cleaner target grammar than Gateway currently implements.

### Not yet implemented in Gateway

- block-hash references as the outer `block-ref`;
- txid substitution below output/input/satpoint forms;
- combinations such as `2.<txid>.<blockhash>`;
- strict no-leading-zero canonicalization for all decimal positional forms;
- a standalone library/API named BPS independent of Gateway.

### Implemented in Gateway but outside BPS core

- inscription occurrence coordinates;
- `.bitcoin` namespace routing;
- `god://` activation;
- Bitcoin on Demand peer retrieval;
- Gateway-specific provenance and verification-state metadata.

## 8. Conformance direction

The intended direction is:

1. BPS becomes the implementation-independent grammar.
2. Gateway keeps backward-compatible parsing where practical.
3. Gateway emits canonical BPS forms when presenting new coordinates.
4. Compatibility aliases remain documented as Gateway behavior, not automatically promoted into BPS.
5. BPS tests are eventually factored into reusable test vectors that Gateway and other implementations can share.

## 9. Relevant Gateway files

At the reviewed snapshot:

- `coordinates.go` — positional coordinate types, constructors, parser
- `coordinates_test.go` — inscription positional parsing
- `hash_coordinates.go` — txid-scoped forms and strict hash parsing
- `hash_coordinates_test.go` — hash grammar and namespace tests
- `hash_coordinate_resolution_test.go` — ordinary Bitcoin peer resolution tests
- `txresolve.go` — block-backed coordinate resolution
- `spec/bod-protocol.md` — Bitcoin on Demand retrieval protocol

