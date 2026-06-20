# rsa-pss — RSA-4096 RSASSA-PSS signature verification

Test vectors for `ants_rsa_pss_verify` (foundation/crypto), the verifier for
the RSA leg of the TEE attestation certificate chains per
[RFC-0005](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0005-identity.md).
**AMD SEV-SNP** signs its ASK/ARK certificate chain with **RSA-4096
RSASSA-PSS** (SHA-384, MGF1-SHA-384, salt 48) — the protocol's only RSA; Intel
TDX (see [`tdx-quote`](../tdx-quote/)) has an all-ECDSA PKI and peer identity is
Ed25519. The verifier is verify-only and takes a pre-computed digest, like the
ECDSA verifiers (see [`ecdsa-p384`](../ecdsa-p384/)).

Each vector is verified by the shared ARK public key (`modulus_hex`,
`exponent_hex`, big-endian). `msg_sha384_hex` is the SHA-384 digest the verifier
consumes — computed over a DER `TBSCertificate` — and `sig_hex` is the raw RSA
signature, the same byte-length as the modulus (512 for RSA-4096). `result`:
`valid` MUST verify (`ANTS_OK`), `invalid` MUST be rejected
(`ANTS_ERROR_MALFORMED`).

The pack covers two genuine signatures — the **ASK** intermediate signed by
**ARK**, and the **ARK** self-signed root — plus single-byte mutations a
verifier must reject: a tampered message byte, a tampered signature byte, a
mismatched signature (the ARK self-signature against the ASK message), and a
one-byte truncation.

**Scope.** These vectors exercise **only** the RSASSA-PSS signature against a
provided RSA public key — not ASK/ARK X.509 chain validation (DER/ASN.1),
freshness, or revocation. Those are separate, composable checks; the chain
validator lands next and will hand the verifier the same key these vectors
carry.

## Source

The signatures are drawn from the **real AMD SEV-Milan certificate chain**,
fetched from AMD's Key Distribution Service:

```
https://kdsintf.amd.com/vcek/v1/Milan/cert_chain   (ASK CN=SEV-Milan, ARK CN=ARK-Milan)
sha256(cert_chain) = 22e62f8d2c21a156470145fc75f7b5a377cb053ced3e97f0bd3f8d8ca5941ce6
fetched: 2026-06-20
```

Both signatures were independently confirmed with the OpenSSL CLI before
pinning — `openssl dgst -sha384 -sigopt rsa_padding_mode:pss -sigopt
rsa_pss_saltlen:48 -sigopt rsa_mgf1_md:sha384 -verify ark_pub.pem …` returns
`Verified OK` for both the ASK and ARK `TBSCertificate`s, while PKCS#1 v1.5
verification of the same is rejected — confirming the PSS scheme.

## Generation

`rsa-pss.json` is emitted by the **compiled** `ants_crypto` library — the
`result` of every case is confirmed against `ants_rsa_pss_verify` before it is
printed, and each `msg_sha384_hex` is computed by the reference `ants_sha384`
(not transcribed). The generator is
[`foundation/crypto/tools/rsa_pss_vectors.c`](https://github.com/Ants-Community/ants-client/blob/main/foundation/crypto/tools/rsa_pss_vectors.c)
in the reference client:

```
cmake --build build --target rsa_pss_vectors
./build/foundation/crypto/rsa_pss_vectors > vectors/rsa-pss/rsa-pss.json
```

Deterministic by construction (fixed inputs, no clock, no randomness): two runs
yield identical bytes. The emitter aborts rather than print a vector whose
verdict its own verify does not reproduce.

The producing ants-client commit is recorded in the commit message that lands
or updates `rsa-pss.json`.
