# Sabatech V100-32GB serving line (branch `sabatech/v100-32g`)

Mission: production-grade serving on **2× Tesla V100-SXM2 32GB, TP2**, targeting

- **concurrency ≥ 5** streams (validated, not assumed),
- **context ≥ 190K tokens** (256-aligned; upstream geometry kernel requires
  `k % 256 == 0` — see iamwavecut/ninfer-all#384),
- **maximum committed decode tok/s** at those settings,
- uncensored artifact support as a first-class option.

## Baseline

Branch point: `d0a1212` (QUASAR v0.2.x line, TP2/TP4 packed projections,
retained prefix branches, long-context concurrency). Apache-2.0.

## Improvement plan

1. **Context alignment guard**: clamp/round `--max-context` up to a 256-multiple at
   option parsing; a non-aligned value reaches the GGUF MMQ bridge and kills the
   engine engine-wide (evidence: upstream issue #384, 100K-token request at 200000
   → `gguf matrix product: unsupported geometry` → exit 2). Aligned values
   (131072, 160000, 200192) are known-good on this artifact family.
2. **Validated serving defaults for 2×V100-32GB**: kv headroom must be explicit at
   large contexts (`kv-capacity auto` otherwise starves the prefill workspace);
   publish a stress matrix (concurrency 1/5/8 × 191K-token distinct prompts,
   VRAM sampled at 1 Hz) as the acceptance gate for any context claim.
3. **Benchmark harness donation**: single/multi-stream O2 probe, multi-turn
   prefix-reuse probe (real RAG pattern), concurrency stress harness — the three
   tools behind the measurements above.
4. **Artifact guidance**: GGUF-derived artifacts pay the MMQ bridge tax
   (~12% prefill, lower draft acceptance); document native-profile alternatives
   for the same tunes.

## Non-goals (this branch)

Changing numerical contracts of the QUASAR TP2/TP4 work; supporting <2 cards.
