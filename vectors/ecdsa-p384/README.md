# ecdsa-p384 — NIST P-384 ECDSA signature verification

Test vectors for `ants_ecdsa_p384_verify` (foundation/crypto), the ECDSA
P-384 (secp384r1) verification primitive used to check vendor signature
chains in TEE attestation quotes per
[RFC-0005](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0005-identity.md)
(AMD SEV-SNP signs its attestation report with ECDSA P-384; Intel TDX uses
P-256, see [`ecdsa-p256`](../ecdsa-p256/)). Verify-only — ANTS never produces
P-384 signatures (peer identity is Ed25519).

Each vector carries a 96-byte uncompressed public key (X‖Y, no `0x04` prefix,
shared by the group), a message, its SHA-512 digest (`msg_sha512_hex`, the
input our verify consumes), a 96-byte IEEE-P1363 signature (raw r‖s), and a
`result`: `valid` MUST verify (`ANTS_OK`), `invalid` MUST be rejected
(`ANTS_ERROR_MALFORMED`). The pack covers valid signatures plus adversarial
special-value cases — `r, s ∈ {0, 1, n, n−1, n+1, p}` — i.e. the range and
special-value checks a verifier must enforce.

Only 96-byte signatures are included: this primitive is fixed-width, so
over/under-length P1363 and DER/SEC1 encodings are the TEE layer's parsing
concern, not the primitive's.

## Source

The vectors are drawn from **Project Wycheproof**
(`ecdsa_secp384r1_sha512_p1363_test.json`, `testGroups[0]`), an independent,
adversarial vector set — the cross-check that catches a verifier which is
self-consistent but wrong. `tc_id` is the Wycheproof test id.

The digest is SHA-512 (Wycheproof has no `sha256_p1363` pack for P-384). The
`ants_ecdsa_p384_verify` primitive is hash-agnostic — it consumes the digest
the caller supplies — so SHA-512 here exercises the same curve-verify code
path that SHA-384 will in AMD SEV-SNP report verification.

## Generation

`ecdsa-p384.json` is emitted by the **compiled** `ants_crypto` library — the
`result` of every case is confirmed against `ants_ecdsa_p384_verify` and each
`msg_sha512_hex` is computed by `ants_sha512`, never transcribed (RFC-0008
§8). The generator is
[`foundation/crypto/tools/ecdsa_p384_vectors.c`](https://github.com/Ants-Community/ants-client/blob/main/foundation/crypto/tools/ecdsa_p384_vectors.c)
in the reference client:

```
cmake --build build --target ecdsa_p384_vectors
./build/foundation/crypto/ecdsa_p384_vectors > vectors/ecdsa-p384/ecdsa-p384.json
```

Deterministic by construction (fixed inputs, no clock, no randomness): two
runs yield identical bytes. The emitter aborts rather than print a vector
whose verdict its own verify does not reproduce.

The producing ants-client commit is recorded in the commit message that lands
or updates `ecdsa-p384.json`.
