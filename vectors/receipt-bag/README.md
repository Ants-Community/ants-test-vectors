# receipt-bag — selective disclosure of receipts

Test vectors for the reputation/identity (Component #9) selective-disclosure
objects, per
[RFC-0008 v0.7 §11.9 + §4.1](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0008-wire-formats.md)
and RFC-0004 v0.7 §"Selective disclosure of receipts" + §"Compact summaries
for large peers":

- the countersigned receipt body encoding and both Ed25519 signatures
  (RFC 8032 signing is deterministic, so the signatures are stable vectors);
- `bag_root` (including the empty bag: merkle root `BLAKE3(0x02)`, then the
  `ants-v1-receipt-bag-root` derive_key peer binding) and inclusion paths;
- the (A, T) computation over the fixture bag under the DRAFT default
  parameters;
- the `A ≥ b` proof: prover-side most-recent-first subset selection and the
  verifier's acceptance verdict;
- the compact summary: bucketing, the signed map(5) body, the map(6) full
  encoding, and per-bucket audit verdicts (a full disclosure passes; a
  withheld receipt fails).

The fixture bag deliberately includes a receipt crediting a **different**
server: it is committed in the Merkle tree (inclusion-provable) but never
creditable to the subject peer — the case every eligibility filter must
reject.

## Generation

Every value in `identity.json` is emitted by the **compiled**
`ants_reputation` library — never transcribed by hand (RFC-0008 §8). The
generator is
[`reputation/identity/tools/identity_vectors.c`](https://github.com/Ants-Community/ants-client/blob/main/reputation/identity/tools/identity_vectors.c)
in the reference client:

```
cmake --build build --target identity_vectors
./build/reputation/identity/identity_vectors > vectors/receipt-bag/identity.json
```

The tool is deterministic by construction (repeated-byte private keys and
nonces, no clock, no randomness, no floats): two runs yield identical
bytes. Every proof, signature, and verdict is re-verified through the
library's own verifier before emission, and the summary encoding was
additionally eye-decoded byte-for-byte against §11.9 when the pack first
landed.

The producing ants-client commit is recorded in the commit message that
lands or updates `identity.json` in this repository.

## Caveats

The decay ratios, κ, bin width, and the summary bucket width are **DRAFT
b2-class values** (RFC-0004 §835; RFC-0008 §7) — the encodings, the decay
*recipe*, and the audit semantics are the pinned deliverable; the tunables
will be recalibrated on testnet evidence and the pack re-emitted with a
version bump when calibration lands.

Hand-checkable anchors inside the pack: the fixture A is
`2^30>>24 + 2^30>>23 + 12345 = 12537` (whole-bin ages 24/23/0 under the
exact-halving 0.5 ratio), the summary buckets are `[[0, 192], [1, 12345]]`,
and the `A ≥ b` selection walks most-recent-first to `[3, 1]`.
