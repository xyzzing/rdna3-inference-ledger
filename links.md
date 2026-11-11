# Links — upstream records, filed bugs, papers, related efforts

## Upstream: Niko1221/Strata (the engine this work ran on)

| Record | What it is | State (2026-11-11) |
|---|---|---|
| [PR #755](https://github.com/Niko1221/Strata/pull/755) | hipBLASLt tuning table for gfx1100 (our 32-row tuning, +82% @128K prefill) | merged |
| [PR #3919](https://github.com/Niko1221/Strata/pull/3919) | Adaptive KV ring (request-sized, not fixed 32K; +9.4–10.6% decode at depth) | open, merge-ready |
| [PR #2224](https://github.com/Niko1221/Strata/pull/2224) | PcieAuto on HIP (maintainer ported our unmeasured-guard removal) | upstream |
| [PR #5458](https://github.com/Niko1221/Strata/pull/5458) | Benchmark report (prefill stack results) | open, no maintainer response yet |
| [Issue #5459](https://github.com/Niko1221/Strata/issues/5459) | Defaults-row report (missing form field fixed post-filing) | open |
| [PR #5535](https://github.com/Niko1221/Strata/pull/5535) | i8-WMMA attention benchmark report (parity-green, 3.8% slower than f16; carries the RDNA4/R9700 angle — gfx12 may price it differently) | open |
| [Issue #5539](https://github.com/Niko1221/Strata/issues/5539) | Chunked-prefill device memory access fault at the 3×32K chunk handoff (3/3 deterministic; 45% faster when it works) | open, bot-accepted |

Channel norms learned on this tracker: results-only PRs are docs-only and never merge (evidence
records); defaults changes need an issue first, opt-in kernels don't; benchmark-report PRs are
the pre-file — code PRs only on maintainer signal.

## Filed compiler bug

| Record | What it is | State |
|---|---|---|
| [ROCm/llvm-project#5330](https://github.com/ROCm/llvm-project/issues/5330) | amdclang contracts explicitly-rounded intrinsics (`__fadd_rn(__fmul_rn(a,c),b)`) into an FMA whose exact value flips round-to-nearest tie cases. Full repro included (default build vs `-ffp-contract=off` vs host). Workaround: volatile barriers between explicitly-rounded ops in bitwise device code | open |
| (related) LLVM 24 VOPD pairing nondeterminism | Second same-genus compiler finding, observed by the [Radiance](https://codeberg.org/StillDeadcode/radiance) project — bitwise-device-code people should treat compiler pairing nondeterminism as a live risk class | upstream |

## External proposal records

| Record | What it is |
|---|---|
| [hk-dsl#367](https://github.com/vivienfanghuagood/hk-dsl/issues/367) | Evidence + catalog-op proposal: block-scale (GGUF Q8_0) W8A8 GEMM, with the gate-1 numbers and a follow-up disclosure comment noting our production padded fp16-WMMA tiles are ~4.5× faster than the i8 kernel (the 1.39× is specific to the naive dequant+GEMM baseline) |

## Papers behind the drafter experiments

| Paper | Used in | Outcome |
|---|---|---|
| [AdaEDL (2410.18351)](https://arxiv.org/abs/2410.18351) | entropy-gated early stop, training-free | implemented + swept 2.0→8.0 nats: no operating point on this drafter (entropy >8 nats everywhere despite 94% top-1) |
| [Cascade (2506.20675)](https://arxiv.org/abs/2506.20675) | MoE utility gating | never built — moot at 94% acceptance / window pricing |
| [Sequoia (2402.12374)](https://arxiv.org/abs/2402.12374) | tree verification | measured-first audit: hedge rescue 0/10, all topologies 1.000× linear at S=8 — port priced, not scheduled |

## Related efforts (other engines we calibrate against)

| Project | What it is | Relevance |
|---|---|---|
| [Radiance](https://codeberg.org/StillDeadcode/radiance) (Apache-2.0) | AMD-only C++/HIP engine, full gfx11 family incl. gfx1100 | independent confirmation of our i8-WMMA fragment facts in production; their MoE-GEMM expert-schedule spread (41.67→21.16 ms, same bytes) matches our attribution findings |
| llama.cpp | the lineage ancestor | same-GPU/weights reference point: 18.7 tok/s decode vs our 38–43 |
