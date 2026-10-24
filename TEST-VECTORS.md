# blocators Test Vectors

Status: Initial draft  
Network for chain-backed examples: Bitcoin mainnet

These vectors are intended to become shared conformance tests.

## 1. Mainnet genesis fixtures

Mainnet genesis block:

```text
height:
0

block hash:
000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f

coinbase txid:
4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b
```

The genesis block contains one transaction at index 0.

That transaction contains one output at index 0 with value 5,000,000,000 satoshis.

## 2. Valid positional coordinates

| Coordinate | Expected type | Expected result |
|---|---|---|
| `0` | block | mainnet genesis block |
| `0.0` | transaction | genesis coinbase transaction |
| `0.0.0` | output | output 0 of genesis coinbase |
| `i0.0.0` | input | coinbase input 0 |
| `0.0.0.0` | satpoint | offset 0 of genesis output 0 |
| `4999999999.0.0.0` | satpoint | final valid offset in genesis output 0 |

## 3. Equivalent genesis transaction coordinates

These MUST resolve to the same transaction on mainnet:

```text
0.0

4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b.0

0.000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f

4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b.000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
```

## 4. Equivalent genesis output coordinates

These MUST resolve to the same output:

```text
0.0.0

0.4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b.0

0.0.000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f

0.4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b.000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
```

## 5. Bounds failures

These MUST fail resolution on mainnet:

| Coordinate | Reason |
|---|---|
| `1.0` | transaction index 1 does not exist in genesis block |
| `1.0.0` | output index 1 does not exist in genesis coinbase |
| `i1.0.0` | input index 1 does not exist in genesis coinbase |
| `5000000000.0.0.0` | offset equals output value and is therefore out of range |

## 6. Lexical failures

These MUST be rejected before chain resolution:

```text
00
00.0
0.00
01.0.0
i00.0.0
+1.0
-1.0
1..0
.1.0
1.0.
i.0.0
```

## 7. Hash membership failure

A txid-scoped coordinate MUST NOT fall back to another transaction.

For example, if a valid txid that is not present in the mainnet genesis block is supplied as:

```text
<TXID_NOT_IN_GENESIS>.0
```

resolution MUST fail with a membership error.

The resolver MUST NOT perform a global txid lookup and return that transaction from another block.

## 8. Case handling

Canonical blocators hashes are lowercase.

Implementations MAY accept uppercase hexadecimal input as a user convenience, but if they do, canonical output MUST be lowercase.

A final BIP draft should decide whether uppercase input acceptance is normative or implementation-specific.

## 9. Qualified-name parsing

Where an implementation supports the `.bitcoin` qualifier:

```text
0.bitcoin
0.0.bitcoin
0.0.0.bitcoin
i0.0.0.bitcoin
0.0.0.0.bitcoin
```

the parsed blocator MUST be identical to the corresponding bare coordinate.

The namespace qualifier MUST NOT alter blocators object semantics.

## 10. Network context

These chain-backed vectors apply only when the resolver context selects Bitcoin mainnet.

The bare coordinate:

```text
0
```

does not globally mean the mainnet genesis block. It means block height 0 on the selected chain.

