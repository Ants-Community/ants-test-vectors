# snp-report — AMD SEV-SNP attestation report signature verification

Test vectors for `ants_snp_report_verify_signature` (foundation/tee), the
verifier that checks an AMD SEV-SNP attestation report's signature against a
VCEK public key per
[RFC-0005](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0005-identity.md).
The report is the `0x4A0`-byte `ATTESTATION_REPORT` structure; its signature
covers `report[0, 0x2A0)` and is ECDSA P-384 over a SHA-384 digest (the
P-384 / SHA-384 primitives the verifier composes are in `foundation/crypto`,
see [`ecdsa-p384`](../ecdsa-p384/)).

Each vector carries a full `report_hex` and a `result`: `valid` MUST verify
(`ANTS_OK`), `invalid` MUST be rejected (`ANTS_ERROR_MALFORMED`). The VCEK
leaf public key is shared at the top level as `vcek_pubkey_hex` — the 96-byte
uncompressed point `X‖Y` (big-endian, no `0x04` prefix), exactly what the
verifier consumes. R and S inside the report are stored little-endian in
72-byte slots (low 48 significant).

The pack covers the genuine report (valid) plus single-byte mutations that a
verifier must reject: a tampered body byte inside the signed prefix, a
tampered R or S, an unsupported `SIGNATURE_ALGO`, non-canonical R padding
(the high 24 bytes of the slot must be zero), and a wrong-length report.

**Scope.** These vectors exercise **only** the report-to-VCEK signature — not
VCEK certificate-chain validation (VCEK → ASK → ARK, which needs ASN.1/X.509
+ RSA-4096), freshness, or revocation. Those are separate, composable checks.

## Source

The base report is a **genuine attestation report from real AMD Milan
silicon**, drawn from
[`google/go-sev-guest`](https://github.com/google/go-sev-guest)
`verify/testdata` (`attestation.bin`, the VCEK leaf `vcek.testcer` with
`CN=SEV-VCEK` / issuer `SEV-Milan`), pinned at commit `33e009a4`. Its
signature was independently confirmed with OpenSSL before pinning:

```
openssl x509 -in vcek.testcer -inform DER -pubkey -noout > pub.pem
dd if=attestation.bin bs=1 count=672 of=signed.bin     # report[0:0x2A0]
# R = report[0x2A0:+48] LE, S = report[0x2E8:+48] LE -> DER -> sig.der
openssl dgst -sha384 -verify pub.pem -signature sig.der signed.bin
# => Verified OK
```

`sha256(attestation.bin) = 377e6241d3b373ab1df80c0f96978594e7e21f4797dd6ea95e2957e1c1e26060`

## Generation

`snp-report.json` is emitted by the **compiled** `ants_tee` library — the
`result` of every case is confirmed against `ants_snp_report_verify_signature`
before it is printed, the negative cases being single-byte mutations of the
genuine report. The generator is
[`foundation/tee/tools/snp_vectors.c`](https://github.com/Ants-Community/ants-client/blob/main/foundation/tee/tools/snp_vectors.c)
in the reference client:

```
cmake --build build --target snp_vectors
./build/foundation/tee/snp_vectors > vectors/snp-report/snp-report.json
```

Deterministic by construction (fixed inputs, no clock, no randomness): two
runs yield identical bytes. The emitter aborts rather than print a vector
whose verdict its own verify does not reproduce.

The producing ants-client commit is recorded in the commit message that lands
or updates `snp-report.json`.
