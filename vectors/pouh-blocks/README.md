# pouh-blocks — L2 chain protocol objects

Test vectors for the reputation/chain (Component #8) protocol objects, per
[RFC-0008 v0.6 §11.4 + §11.6 + §4.2](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0008-wire-formats.md):

- confirmed-proofs Merkle roots (including the empty root `BLAKE3(0x02)` and
  the promote-lone shapes) and inclusion paths;
- the EpochSummary canonical-CBOR encoding;
- the Block canonical-CBOR encoding and block hash, with the `degraded_seed`
  flag flipped both ways to pin its effect inside the signed bytes;
- the VRF seed derivations (`ants-v1-vrf-seed`, `ants-v1-vrf-seed-degraded`);
- the proposer rule (`height mod k`);
- a pattern-scan worked example.

## Generation

Every value in `chain.json` is emitted by the **compiled** `ants_chain`
library — never transcribed by hand (RFC-0008 §8). The generator is
[`reputation/chain/tools/chain_vectors.c`](https://github.com/Ants-Community/ants-client/blob/main/reputation/chain/tools/chain_vectors.c)
in the reference client:

```
cmake --build build --target chain_vectors
./build/reputation/chain/chain_vectors > vectors/pouh-blocks/chain.json
```

The tool is deterministic by construction (fixed repeated-byte inputs, no
clock, no randomness, no floats): two runs yield identical bytes. Inclusion
paths are verified against their roots before emission.

The producing ants-client commit is recorded in the commit message that
lands or updates `chain.json` in this repository.

## Deliberately not in this pack

drand beacon rounds: they verify against the live drand chain itself; the
reference client pins real rounds (with dated provenance) in its
`test_chain.c`.

## Caveats

The pattern-rule thresholds and windows, k values, and every numeric
tunable are **DRAFT b2-class values** (RFC-0008 §7) — the encodings and
derivations are the pinned deliverable; the tunables will be recalibrated
on testnet evidence.
