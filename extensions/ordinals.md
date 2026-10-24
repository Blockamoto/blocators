# blocators Ordinals Extension

Status: Experimental extension draft  
Core dependency: blocators 0.1

This document is not part of the Bitcoin-native blocators core proposal.

## 1. Purpose

blocators core describes positions in confirmed Bitcoin structure. Ordinal theory can reuse those positions while adding inscription and sat-tracking semantics.

## 2. Inscription reveal coordinate

Canonical candidate:

```text
<tx-ref>i<inscription-index>.<block-ref>
```

Examples:

```text
12i0.792435
<txid>i0.792435
12i0.<blockhash>
<txid>i0.<blockhash>
```

The inscription index is zero-based according to the selected Ord inscription ordering rules.

A positional transaction reference is resolved within the referenced block. A txid reference MUST be verified as a member of the referenced block.

Gateway currently also accepts this compatibility alias:

```text
<tx-ref>.i<inscription-index>.<block-ref>
```

The compact suffix form is the preferred canonical candidate.

## 3. Reveal identity vs location

The reveal coordinate identifies immutable historical provenance.

It does not mean "where is this inscription now?"

Later output or satpoint locations are state derived and may require sat lineage plus Ord indexing rules. They should be specified separately.

## 4. Candidate output locator

A future extension may define:

```text
<output-index>i<inscription-index>.<tx-ref>.<block-ref>
```

Example:

```text
2i0.17.840000
```

Before standardization, the extension must define exactly how inscriptions within one output are ordered.

## 5. Candidate satpoint locator

A future extension may define a disambiguator for multiple inscriptions associated with one historical satpoint:

```text
<offset>i<inscription-index>.<output-index>.<tx-ref>.<block-ref>
```

Example:

```text
100i1.2.17.840000
```

This is not normative in the current draft. Resolving it may require historical Ordinal state rather than only the referenced Bitcoin block.

## 6. Separation from blocators core

Keeping inscriptions outside the base proposal means blocators can be evaluated without requiring adoption of:

- ordinal numbering;
- sat lineage;
- inscription parsing rules;
- inscription numbering;
- historical Ord rule profiles.

An implementation may still derive inscription reveal data directly from authenticated Bitcoin witness data. Gateway currently does so.

## 7. Design rule

Expose a pointer only when its referent and ordering are deterministic.

Syntax should follow a defined object, not create one.
