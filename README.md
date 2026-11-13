# qwen-rdna3-optimization-ledger

Measured results from a two-month optimization campaign (Sep 28 – Nov 12, 2026) serving a
**Qwen3.8 Flash-Next MoE** checkpoint (IQ3_S, mixed q8_0/bf16 pack) on a single
**RX 7900 XTX** (gfx1100 / RDNA3, 24 GiB), with a HIP inference engine — a fork of
[Strata](https://github.com/Niko1221/Strata) (llama.cpp-derived lineage) plus a later
independent core study.

**~125 recorded experiments: ~20 adopted, ~85 rejected with paired measurements.** The rejected corpus is the asset. RDNA3 optimization advice is dominated by CUDA
folklore transferred by analogy, and most of it does not survive contact with this hardware:
NVIDIA's own Q8_0 multi-column layout measured **−17 to −22% on RDNA3**; a 22.7%-fewer-bytes KV
format measured **−24% on prefill**; "kernel that wins in isolation" measured **−1.75 to −3.34%
end-to-end**. None of that was guessable. All of it was cheap to close once measured under a
fixed protocol.

## Start here

- **[Where we are](where-we-are.md)** — the plain-language guide: how generation works, what we
  made faster, which doors are measured shut, and **what transfers to Qwen 4** (three tiers).
- **[Campaign update Nov 12/13](campaign-2026-11-13.md)** — the newest arc: the first matched
  deep-vs-short depth table, the 94K penalty attributed to the proposal side (zero engine
  code — the production binary already carried the diagnostics), and the head-repack screen
  that closed the last bitwise candidate.
- **[Campaign update Nov 11/12](campaign-2026-11-12.md)** — the newest arc: a correctness
  mystery settled by a noise floor, a performance question closed by arithmetic, and the
  session-lookup idea chased from paper claim to drift-matched refutation.

## Contributing

**Community contributions are welcome — humans and agents.** Start with
**[CONTRIBUTING.md](CONTRIBUTING.md)**: how to propose and record experiments as GitHub issues,
the evidence-packet format (`experiments/issue-<number>/`), the correctness and timing gates,
the one-scope verdict vocabulary (PASS / NEGATIVE / INCONCLUSIVE / VOID / BLOCKED), and the
review checklist. Bottleneck measurements, independent reproductions, corrections, minimal
bug reproducers, and well-supported negatives are all first-class contributions — hardware
owners and analysis-only contributors alike.

## The arc in three numbers

| | First light (Sep 28) | porting-era peak (Oct 20) | Deployed (Nov 7+) |
|---|---|---|---|
| Prefill | 32.35 tok/s (first inference) | 926 → 2,404 tok/s @128K (**+159%**) | **~3,400 tok/s @94K** |
| Decode @1K | — | 62.3 → 144 tok/s (**+131%** in 17 days) | ~43–47 tok/s @94K depth |

Same GPU, same weights, the engine measured **2.1–2.3× faster decode than llama.cpp**.

## What shipped (headline — full table in [adopted.md](adopted.md))

- **MMQ quantized-GEMM prefill path**: +112% prefill
- **hipBLASLt tuning table for gfx1100** (was absent): +82% @128K prefill — merged upstream [#755](https://github.com/Niko1221/Strata/pull/755)
- **WMMA tensor-core attention on gfx1100** (select scorer + prompt attention): 10–11× kernel, +22.8% @128K prefill
- **ROUTE_TAIL_SKIP**: skip never-routed experts in verify windows — **+14–19% decode**, the biggest single decode win
- **Adaptive KV ring** sized to the request: +9.4–10.6% decode at depth — upstream [#3919](https://github.com/Niko1221/Strata/pull/3919)
- **`--prefill auto` chunking** (+26.4%) and the PA_FAST+FUSED+RING512 stack (+13.82%)
- File-backed expert mmap + cgroup valve: fixed 3× OOM kills at 47.9 GiB pinned

## What didn't (samples — full corpus in [rejected/](rejected/))

- **DFlash2 drafter** (claims +8–11%): WASH, median −5%, ~3× the variance of MTP
- **Sequoia tree verification**: built as a measure-first audit first — hedge rescue rate **0/10** chain breaks, every tree topology 1.000× linear; closed without the port
- **Launch-ahead / megakernel**: decode is serialization-floor-bound; the "unfused seams" the megakernel targets turned out not to exist
- **AdaEDL entropy gate**: drafter distribution entropy exceeds **8 nats on every step** despite 94% top-1 acceptance — no operating point exists
- **i8-WMMA dense GEMM**: 1.39× vs the naive baseline — but production's padded fp16 path is **4.5× faster**; a component win vs a baseline nobody runs is not a candidate

## Repository layout

| Path | Content |
|---|---|
| [adopted.md](adopted.md) | Everything that shipped, with numbers and provenance |
| [rejected/](rejected/) | The negative corpus: drafter, kernels/layouts/quantization, memory/system, the final campaign |
| [lessons/toolchain.md](lessons/toolchain.md) | HIP/amdclang/WMMA/data-layout gotchas (incl. a filed compiler bug) |
| [lessons/measurement.md](lessons/measurement.md) | Measurement rules that made each close cheap — the most reusable file here |
| [methodology.md](methodology.md) | The benchmark protocol + a ratio-definitions ledger pinning every ambiguous number |
| [open-upstream.md](open-upstream.md) | What remains (all upstream waits, incl. a device-fault bug report) |
| [links.md](links.md) | Upstream PRs/issues, filed bugs, papers, related efforts |

## Method in one paragraph

Paired interleaved baseline/candidate arms inside one process and one window; cross-process
sweeps drift and have reversed a "+5–17%" win into "−17 to −22%" on re-measurement. Predeclared
gates (e.g. promote at median ratio ≥ 1.020 with bootstrap CI excluding 1.00) — never relaxed
after the fact. Keep every raw row, failed run and exclusion reason. Uninstrumented wall time is
the only performance authority; instrument traces must disclose their perturbation. Full protocol
and the ratio-definition ledger: [methodology.md](methodology.md).

## Status (2026-11-11)

The board is fully dispositioned: **zero promotions across the final ~12-experiment campaign**;
the deployed stack stands on the adopted set; every remaining item is an upstream wait
([open-upstream.md](open-upstream.md)). The campaign's most valuable export is the negative
knowledge plus the measurement rules that made each close cheap.

## License

[CC-BY-4.0](LICENSE). Hardware/software provenance per file; upstream PR and issue numbers link
to the public records. The working repos behind this ledger are private; this repository is the
curated export.
