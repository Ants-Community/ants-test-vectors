# vectors/

Test vector files. One subdirectory per component / primitive.

**Status:** six packs shipped — `blake3/`, `ed25519/`, `bls12-381/`,
`ecvrf-ell2/`, `cbor-canonical/`, `pouh-blocks/`. Vectors land as components in
[`Ants-Community/ants-client`](https://github.com/Ants-Community/ants-client)
become feature-complete. `pouh-blocks/` is the first pack emitted by a
committed generator tool from the compiled reference library
(`reputation/chain/tools/chain_vectors.c`), per RFC-0008 §8.

## Layout (planned)

```
vectors/
├── blake3/                          # from ants-client/foundation/crypto
├── ed25519/                         # from ants-client/foundation/crypto
├── bls12-381/                       # from ants-client/foundation/crypto
├── ecvrf-ell2/                      # from ants-client/foundation/crypto
├── cbor-canonical/                  # from ants-client/foundation/cbor
├── tee-attestation/                 # from ants-client/foundation/tee  (per vendor)
├── fault-proofs/                    # from ants-client/reputation/crdt
├── pouh-blocks/                     # from ants-client/reputation/chain
├── identity-bond/                   # from ants-client/reputation/identity
├── cache-entries/                   # from ants-client/cache/semantic
├── canonical-kernels/               # from ants-client/inference/kernels
├── commit-at-send/                  # from ants-client/inference/orchestration
├── beacon-derived/                  # from ants-client/reputation/chain
├── e-process/                       # from ants-client/inference/orchestration
└── bond-formulas/                   # from ants-client/economy/bond
```

Each subdirectory contains:

- One CBOR file per vector (deterministic encoding per RFC-0008 §1.1)
- A `README.md` describing the input space and any
  vector-generation script
- A `generation/` subdir with the script (or reference to the
  ants-client commit that produced the vectors)

## Vector schema reminder

Per RFC-0008 §8:

```
TestVector {
  primitive:    string,           // canonical name, e.g. "blake3.derive_key"
  context:      string,           // domain separation, where applicable
  input:        bytes / structured,
  output:       bytes / structured,
  notes:        text,
}
```
