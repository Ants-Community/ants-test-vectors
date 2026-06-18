# ants-test-vectors

Test-vector pack for the ANTS protocol's reference primitives.

**Status:** eleven packs shipped (BLAKE3, Ed25519, BLS12-381, ECDSA P-256,
ECDSA P-384, ECVRF-ELL2, canonical CBOR, L2 chain objects, receipt-bag
selective disclosure, canonical embedding, AMD SEV-SNP report verification).
Vectors are generated as components in
[`Ants-Community/ants-client`](https://github.com/Ants-Community/ants-client)
become feature-complete.

## What this is

Per [RFC-0008 §8](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0008-wire-formats.md),
every protocol-level primitive ships at least one
fixed-input/fixed-output test vector. A client implementation is
**conformant** iff its outputs match every vector for every input.
This is the cleanest possible interoperability test: the bytes match,
or they do not.

The vectors live here, in a sibling repository to the reference
client. The reference client is the *first* implementation; its
outputs (once feature-complete and audited) are captured as the
authoritative vectors. Future client implementations validate
against these vectors. The chicken-and-egg is acknowledged in
IMPLEMENTATION.md §"Test vector pack".

## Vector format

Each vector is a CBOR file with the structure specified in RFC-0008
§8:

```
TestVector {
  primitive:    string,           // e.g. "blake3.derive_key", "ed25519.sign"
  context:      string,           // domain separation, where applicable
  input:        bytes / structured,
  output:       bytes / structured,
  notes:        text,
}
```

Categories per RFC-0008 §8 (each will have multiple vector instances
as components arrive):

- BLAKE3 plain hash and `derive_key` per reserved context — first
  pack landed at [`vectors/blake3/blake3.json`](./vectors/blake3/blake3.json) (2026-05-20, alongside ants-client PR #7)
- Ed25519 signature generation and verification — first pack at [`vectors/ed25519/ed25519.json`](./vectors/ed25519/ed25519.json) (2026-05-20, alongside ants-client PR #8). RFC 8032 §7.1 vectors verified.
- BLS12-381 signature aggregation and verification — first pack at [`vectors/bls12-381/bls.json`](./vectors/bls12-381/bls.json) (2026-05-21, alongside ants-client PR #9)
- ECVRF-EDWARDS25519-SHA512-ELL2 prove and verify — first pack at [`vectors/ecvrf-ell2/ecvrf.json`](./vectors/ecvrf-ell2/ecvrf.json) (2026-05-21; RFC 9381 §B.4 Examples 19/20/21 byte-exact)
- CBOR canonical encoding of every protocol object type — feature-complete pack at [`vectors/cbor-canonical/`](./vectors/cbor-canonical/)
- L2 chain objects (RFC-0008 §11.6): confirmed-proofs Merkle, EpochSummary, Block, VRF seeds, proposer rule — first pack at [`vectors/pouh-blocks/chain.json`](./vectors/pouh-blocks/chain.json) (2026-06-12, emitted by ants-client's committed `chain_vectors` tool)
- Receipt bodies, receipt-bag inclusion proofs, the `A ≥ b` proof, and compact summaries (RFC-0008 §11.9) — first pack at [`vectors/receipt-bag/identity.json`](./vectors/receipt-bag/identity.json) (2026-06-12, emitted by ants-client's committed `identity_vectors` tool)
- Canonical embedding model `ants-embed-v1` (RFC-0008 §5/§5.1): pinned model hashes + reference embeddings (BGE-M3 dense, multilingual) — first pack at [`vectors/embedding/embedding.json`](./vectors/embedding/embedding.json) (2026-06-15, emitted by ants-client's `embed_vectors --pack`). The hashes are the bit-exact pin; the embeddings are cosine ≥ 0.999 reference, pending cross-platform F32 numerics (RFC-0009).
- ECDSA P-256 signature verification (RFC-0005 TEE attestation chains; Intel TDX signs ECDSA P-256): valid + adversarial signatures from Project Wycheproof — first pack at [`vectors/ecdsa-p256/ecdsa-p256.json`](./vectors/ecdsa-p256/ecdsa-p256.json) (2026-06-15, emitted by ants-client's committed `ecdsa_p256_vectors` tool)
- ECDSA P-384 signature verification (RFC-0005 TEE attestation chains; AMD SEV-SNP signs ECDSA P-384): valid + adversarial signatures from Project Wycheproof — first pack at [`vectors/ecdsa-p384/ecdsa-p384.json`](./vectors/ecdsa-p384/ecdsa-p384.json) (2026-06-16, emitted by ants-client's committed `ecdsa_p384_vectors` tool)
- AMD SEV-SNP attestation report signature verification (RFC-0005 TEE attestation): a genuine report from real AMD Milan silicon (`go-sev-guest` testdata, OpenSSL-confirmed) + single-byte mutations — first pack at [`vectors/snp-report/snp-report.json`](./vectors/snp-report/snp-report.json) (2026-06-18, emitted by ants-client's committed `snp_vectors` tool)
- DHT RPC and gossip frame round-trips (RFC-0008 §11.2–§11.3)
- Merkle tree construction over logit traces (RFC-0003 §commit-at-send)
- Fault proof verification (RFC-0004)
- Beacon-derived selection (audit-flag, position-set, auditor) for given seeds
- Cache entry signing and verification
- Per-position discrepancy score `X_j` computation
- Betting e-process `M_n` computation
- Bond formula application per act class

## How vectors are added

Each `ants-client` component, when its primary contributor reports
feature-complete, generates the vectors for its primitives by running
its own implementation against a canonical input set. The vectors are
committed to this repo via PR. The component then validates against
those vectors in its CI as the conformance test.

A PR adding vectors must include:

- The vector files (CBOR-encoded per RFC-0008 §1.1 deterministic
  encoding).
- A `vectors/<component>/README.md` describing the input space and
  any input-generation script.
- A reference to the spec section the vectors cover.

Vectors are **immutable once merged**. Errors in vectors are fixed by
adding a corrected vector with a higher version suffix and deprecating
the old one. The deprecation is recorded in
[`CHANGELOG.md`](./CHANGELOG.md) (to be created at the first
deprecation — none has happened yet).

## License

Dual-licensed under **Apache-2.0 OR MIT**. See
[LICENSE-APACHE](./LICENSE-APACHE) and [LICENSE-MIT](./LICENSE-MIT).
Vector data is fact-of-computation and not copyrightable in most
jurisdictions; the dual-license is for the surrounding tooling and
documentation.

## Protocol specification

The protocol itself lives at
[Ants-Community/ants](https://github.com/Ants-Community/ants).
The reference client (which generates these vectors) lives at
[Ants-Community/ants-client](https://github.com/Ants-Community/ants-client).
This repository is purely the conformance surface between the two.
