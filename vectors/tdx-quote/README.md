# tdx-quote — Intel TDX DCAP attestation quote signature verification

Test vectors for `ants_tdx_quote_verify_signature` (foundation/tee), the
verifier that checks an Intel TDX DCAP v4 quote's internal signature chain
against a PCK leaf public key per
[RFC-0005](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0005-identity.md).
A quote is a 48-byte header, a 584-byte TD report body, and a variable-length
signature section. Where AMD SEV-SNP (see [`snp-report`](../snp-report/))
carries a single report-to-VCEK signature, a TDX quote is a **three-link
chain**, all of it ECDSA P-256 over SHA-256 (the curve / digest primitives
the verifier composes are in `foundation/crypto`, see
[`ecdsa-p256`](../ecdsa-p256/)):

1. the platform **PCK leaf** signs `SHA-256(QE report)`;
2. the **QE report**'s `report_data` binds the per-quote **attestation key**
   (`== SHA-256(att_key ‖ qe_auth_data)`);
3. the **attestation key** signs `SHA-256(header ‖ TD body)`.

Together: PCK → QE → attestation key → TD body.

Each vector carries a full `quote_hex` and a `result`: `valid` MUST verify
(`ANTS_OK`), `invalid` MUST be rejected (`ANTS_ERROR_MALFORMED`). The PCK
leaf public key is shared at the top level as `pck_leaf_pubkey_hex` — the
64-byte uncompressed point `X‖Y` (big-endian, no `0x04` prefix), exactly what
the verifier consumes. All ECDSA `r‖s` and `X‖Y` values inside the quote are
big-endian (passed straight through, unlike SNP's little-endian R/S).

`quote_hex` is the **structured** quote (the first `0x27C + sig_data_len`
bytes). The upstream `.dat` appends a 39-byte test-only ASCII trailer beyond
`sig_data_len` that is not part of the quote; it is stripped here.

The pack covers the genuine quote (valid) plus single-byte mutations that a
verifier must reject: a tampered TD body byte, a tampered attestation
signature, a tampered attestation key, a tampered QE report, a tampered QE
report signature, tampered QE auth data, an unsupported quote version, a wrong
certification-data type, and a truncated quote.

**Scope.** These vectors exercise **only** the quote's internal signatures
against a provided PCK leaf — not PCK certificate-chain validation to the
Intel root (which needs ASN.1/X.509), freshness, or revocation. Those are
separate, composable checks; the chain validator lands next and will hand the
verifier the same leaf key these vectors carry.

## Source

The base quote is a **genuine production Sapphire Rapids Intel TDX quote**,
drawn from
[`google/go-tdx-guest`](https://github.com/google/go-tdx-guest)
`testing/testdata/tdx_prod_quote_SPR_E4.dat` (Apache-2.0), pinned at commit
`4612d58`. Its three signature links were independently confirmed with Python
`cryptography` and the OpenSSL CLI before pinning — the self-contained
attestation-key signature over `header ‖ TD body` verifies, the QE report's
`report_data` matches `SHA-256(att_key ‖ qe_auth_data)`, and the QE report
signature verifies against the PCK leaf extracted from the embedded cert chain
(`CN=Intel SGX PCK Certificate` → `Intel SGX PCK Platform CA` → `Intel SGX
Root CA`).

```
sha256(tdx_prod_quote_SPR_E4.dat) = 6dde5548bec99147fef832643301f113df99931547be26df8ac376c4eaa5b5a7
sha256(structured quote)          = 3507b5f7e6124e17210ffb4d5caf25a5d289a64fb19068ae90cd4cb25828db9f
```

## Generation

`tdx-quote.json` is emitted by the **compiled** `ants_tee` library — the
`result` of every case is confirmed against `ants_tdx_quote_verify_signature`
before it is printed, the negative cases being single-byte mutations of the
genuine quote. The generator is
[`foundation/tee/tools/tdx_vectors.c`](https://github.com/Ants-Community/ants-client/blob/main/foundation/tee/tools/tdx_vectors.c)
in the reference client:

```
cmake --build build --target tdx_vectors
./build/foundation/tee/tdx_vectors > vectors/tdx-quote/tdx-quote.json
```

Deterministic by construction (fixed inputs, no clock, no randomness): two
runs yield identical bytes. The emitter aborts rather than print a vector
whose verdict its own verify does not reproduce.

The producing ants-client commit is recorded in the commit message that lands
or updates `tdx-quote.json`.
