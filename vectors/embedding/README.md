# vectors/embedding — canonical embedding model (`ants-embed-v1`)

Reference vectors for Component #11 (the canonical embedding service), per
[RFC-0008 §5 / §5.1](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0008-wire-formats.md)
and [RFC-0002 §The canonical embedding model](https://github.com/Ants-Community/ants/blob/main/spec/RFC-0002-semantic-cache.md).

`embedding.json` is emitted by the compiled reference client
(`cache/embedding/tools/embed_vectors --pack`), never transcribed by hand.

## What is pinned vs. what is reference

Unlike the other packs in this repo, the embedding pack is **not** a bit-exact
cross-platform conformance suite — and it says so honestly:

- **Bit-exact, cross-platform (the real pin):** `model_id`, `model_arch`, and
  the two BLAKE3 hashes. Every peer claiming `ants-embed-v1` MUST load a
  weights GGUF and a `tokenizer.json` that hash to these exact values, or
  refuse to start. This is the protocol invariant.
- **Reference, with tolerance:** the per-input `embedding` arrays. The forward
  pass is F32 and its reduction order is not yet canonicalised across
  platforms (the open problem in RFC-0009), so two conformant peers agree on
  these vectors to **cosine ≥ 0.999**, not bit-for-bit. They are a regression
  fixture and a sanity reference, not a hash target.

## Canonical artifact

- **Weights:** BGE-M3 **F32 GGUF** (BERT-family / XLM-RoBERTa encoder, 24
  layers, hidden 1024, 16 heads, FFN 4096, 8192 ctx, vocab 250 002). Source:
  official `BAAI/bge-m3`. Obtain `CompendiumLabs/bge-m3-gguf` → `bge-m3-f32.gguf`
  or reproduce via `llama.cpp` `convert_hf_to_gguf.py --outtype f32`.
- **Tokenizer:** the official `BAAI/bge-m3` `tokenizer.json` (HF Unigram).

## Embedding procedure (what produced each vector)

Tokenize → wrap as `<s> … </s>` (XLM-R ids 0 and 2; the dense vector is the
`<s>` hidden state read by CLS pooling) → BGE-M3 forward → L2-normalise. The
reference vectors were checked against `llama.cpp`'s `llama-embedding` on the
same GGUF at cosine ≥ 0.999999.
