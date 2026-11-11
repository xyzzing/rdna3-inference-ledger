# Adopted — what actually shipped

Hardware: one RX 7900 XTX (gfx1100 / RDNA3, 24 GiB). Model: Qwen3.8 Flash-Next MoE
(IQ3_S, mixed q8_0/bf16 pack). Engine: Strata HIP fork + a later independent core study.
Era: Sep 28 – Nov 7, 2026. Every row is a paired measurement against the then-current stack.

## The arc

| | First light (Sep 28) | porting-era peak (Oct 20) | Deployed (Nov 7+) |
|---|---|---|---|
| Prefill | 32.35 tok/s (first inference) | 926 → 2,404 tok/s @128K (**+159%**) | **~3,400 tok/s @94K** |
| Decode @1K | — | 62.3 → 144 tok/s (**+131%** in 17 days) | ~43–47 tok/s @94K depth |

Reference point at the same GPU/weights: llama.cpp measured **18.7 tok/s decode** vs the fork's
38–43 → **2.1–2.3× faster**.

## The production stack

| Change | Plain description | Result | Provenance |
|---|---|---|---|
| MMQ build on | Turn on the compiled-out quantized-GEMM prefill path | Prefill 416→971 tok/s (**+112%**) | fork build flag |
| hipBLASLt tuning table | gfx1100 had no tuned GEMM table; tuned 32 rows | 128K prefill 926→1,687 tok/s (**+82%**) | merged upstream [#755](https://github.com/Niko1221/Strata/pull/755) |
| WMMA select scorer (fp16) | Tensor-core attention block-scorer, opt-in | **10–11×** kernel @128K; e2e +8.7–11.2% | fork, opt-in |
| WMMA prompt-attention | Ported the gfx12-only kernel to gfx1100 | 128K prefill **+22.8%**; attention share 27%→9% | fork |
| fp16 over bf16 select arm | Compared the two select-scorer precision arms | ours **1.64×** faster at 128K select | fork |
| `--prefill auto` chunking | Let the engine chunk deep prompts | **+26.4%** deep prefill (2,365→2,983 tok/s) | deployed flag |
| Prefill stack (PA_FAST+FUSED+RING512) | Three confirmed env knobs, deployed together | +5.43% and +9.08% individually, **+13.82%** combined | deployed env |
| Adaptive KV ring | Resize the KV ring to the request, not a fixed 32K | **+9.4–10.6%** decode at depth | upstream PR [#3919](https://github.com/Niko1221/Strata/pull/3919) (merge-ready) |
| KV-ring 2×-cap growth fix | Old growth rule regressed 64K | 64K went −16.5% → **+23.2%** after fix | fork |
| PcieAuto on HIP | Upstream's CUDA-only PCIe refinement; unmeasured guard removed | Live; maintainer ported it upstream ([#2224](https://github.com/Niko1221/Strata/pull/2224)) | upstream |
| `--spec 5` + SH_STREAM | Deeper draft window + stream flag | +9.5%, then **+11.3%** decode @1K | deployed flag |
| ROUTE_TAIL_SKIP=7 | Skip never-routed experts in verify windows | **+14–19% decode** — the biggest single decode win | deployed env |
| `--lookup-chain 2` | Wider lookup drafts (profitable only *after* TAIL_SKIP) | −5.6% before → +6.3% @1K, **+19.9%** @128K after | deployed flag |
| GDN_CHUNKED + PF_GEMM/PF_PAD | Upstream prefill opt-ins turned on | +5.85% @32K, +4.41% @128K | deployed env |
| File-backed IQ experts (mmap) | Replace 47.9 GiB pinned RAM (was OOM-killed 3×) | OOM fixed; cold-decode trade 14.3→7.6 tok/s | deployed |
| cgroup MemoryHigh=62G | Unit sheds file cache before host OOM | oom_kills 0 since | systemd unit |

## Interaction lessons worth stealing

- **`--lookup-chain 2` was a −5.6% loss until ROUTE_TAIL_SKIP landed, then +6.3/+19.9%.**
  Optimization stacking is not additive or monotone; re-pair every old knob against the new stack.
- **The prefill stack's knobs measured +5.43% and +9.08% individually but +13.82% combined** —
  the rare super-additive case. Measure combinations, not just singles.
- **OOM and performance are the same budget**: 47.9 GiB of pinned expert weights → mmap trade
  (cold decode 14.3→7.6 tok/s) → then the cgroup valve so the kernel sheds page cache before
  the OOM killer acts. Three separate fixes, one memory story.

## Not adopted despite passing gates

- **i8-WMMA attention** (0037): parity/KL all green (rel RMS 1.9e-2, KL 0.0036 ≤ 0.00484 gate),
  long outputs byte-identical at temp 0 — but wall-clock **−32.1% vs the split class and +3.8%
  slower than the deployed f16 arm**. Correct and slower is a rejection. Filed upstream as a
  [benchmark report](https://github.com/Niko1221/Strata/pull/5535) with the RDNA4/R9700 angle
  (the gfx12 successor may price it differently).
- **TSUM butterfly reduction**: real −0.91%, bitwise-equal — under the predeclared 2% promotion
  bar. Documented, not deployed.
