# ecdsa-p256 — NIST P-256 ECDSA signature verification

Test vectors for `ants_ecdsa_p256_verify` (foundation/crypto), the ECDSA
P-256 (secp256r1, with SHA-256) verification primitive used to check vendor
signature chains in TEE attestation quotes per
[RFC-0005](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0005-identity.md)
(Intel TDX signs ECDSA P-256; AMD SEV-SNP uses P-384, a later sibling).
Verify-only — ANTS never produces P-256 signatures (peer identity is Ed25519).

Each vector carries a 64-byte uncompressed public key (X‖Y, no `0x04` prefix,
shared by the group), a message, its SHA-256 digest (`msg_sha256_hex`, the
input our verify consumes), a 64-byte IEEE-P1363 signature (raw r‖s), and a
`result`: `valid` MUST verify (`ANTS_OK`), `invalid` MUST be rejected
(`ANTS_ERROR_MALFORMED`). The pack covers valid signatures plus adversarial
edge cases — `r=0`/`s=0`, `s ∈ {n, n−1, p}`, `r=n+1`, and `r` replaced by
`n−r` — i.e. the range and special-value checks a verifier must enforce.

Only 64-byte signatures are included: this primitive is fixed-width, so
over/under-length P1363 and DER/SEC1 encodings are the TEE layer's parsing
concern, not the primitive's.

## Source

The vectors are drawn from **Project Wycheproof**
(`ecdsa_secp256r1_sha256_p1363_test.json`, `testGroups[0]`), an independent,
adversarial vector set — the cross-check that catches a verifier which is
self-consistent but wrong. `tc_id` is the Wycheproof test id.

## Generation

`ecdsa-p256.json` is emitted by the **compiled** `ants_crypto` library — the
`result` of every case is confirmed against `ants_ecdsa_p256_verify` and each
`msg_sha256_hex` is computed by `ants_sha256`, never transcribed (RFC-0008
§8). The generator is
[`foundation/crypto/tools/ecdsa_p256_vectors.c`](https://github.com/Ants-Community/ants-client/blob/main/foundation/crypto/tools/ecdsa_p256_vectors.c)
in the reference client:

```
cmake --build build --target ecdsa_p256_vectors
./build/foundation/crypto/ecdsa_p256_vectors > vectors/ecdsa-p256/ecdsa-p256.json
```

Deterministic by construction (fixed inputs, no clock, no randomness): two
runs yield identical bytes. The emitter aborts rather than print a vector
whose verdict its own verify does not reproduce.

The producing ants-client commit is recorded in the commit message that lands
or updates `ecdsa-p256.json`.
