# Rejected — kernels, layouts, quantization

The dominant theme: **RDNA3 is not NVIDIA**. Layouts and quant formats that are consensus-good
on CUDA measured negative here, repeatedly, under paired measurement. The second theme: a kernel
that wins in isolation routinely loses end-to-end — the verify window, not the kernel, is the
pricing mechanism.

| Experiment | Plain description | Result |
|---|---|---|
| GSQ execution layouts | Repack IQ3_S expert weights into tile-major / predecoded planes | **+71% to 4.8× slower** — field separation always costs on this access pattern |
| Expert kernel modes 2/4/8 | Alternative expert-decode kernels | ~**3× slower** than the deployed mode, all bitwise-equal |
| EXPDEC (gate/up and down) | A kernel that wins in isolation | Kernel −21–34% faster but e2e **−1.75 to −3.34%** — kernel wins don't survive the window |
| Fused gate/up front half | Fuse SwiGLU into the grouped dot | 29–36% slower |
| Upstream Q8_0 multi-column layout | NVIDIA's good layout, ported faithfully | **−17 to −22% on RDNA3** — a cross-process sweep had shown "+5–17%"; paired in-process re-measurement reversed it |
| MoE fused rocWMMA i8 | Tensor-core expert path | Parity perfect, e2e −0.8% (the MMQ path was already 4.1× faster than BLAS) |
| i8-WMMA prompt attention | Int8 Q·K with exact i32 accumulate | Parity/KL all green (KL 0.0036 ≤ 0.00484) but **3.8% slower than deployed f16** — correct and slower is a rejection ([filed as a benchmark report](https://github.com/Niko1221/Strata/pull/5535), RDNA4 may price it differently) |
| i8-WMMA dense GEMM | Beat the naive dequant+GEMM baseline | 1.39× vs the *naive* baseline — but production's padded fp16 tensor-core path is **~4.5× faster**; a component win vs a baseline nobody runs is not a candidate. (Filed as a disclosure comment on [hk-dsl#367](https://github.com/vivienfanghuagood/hk-dsl/issues/367)) |
| INT8-Q QSA attention | Quantize Q for score math | Fails the precision gate: softmax Δ 0.095 vs the 0.02 bound |
| WY-form parallel GDN recurrence | Break the serial linear-attention walk with WMMA folds | Prefill **40% slower**; upstream rejected their own PR citing these numbers |
| k8v4 KV format | 22.7% fewer KV bytes | **−24% prefill**, −4.9% @128K — rejected twice (bytes moved ≠ bytes worth moving) |
| FUSE_WAIT staging fold | Remove 10.7pp of dispatch waits | Wall clock **−0.35% = wash** — stream overlap had already absorbed the waits |
| PF_FUSED (early variant) | Fused expert path | **1 GPU hang at 128K** — disqualifying regardless of speed |
| TSUM butterfly reduction | Bitwise-equal micro-win | Real −0.91% but under the predeclared 2% promotion bar — documented, not deployed |
| ROUTER_SKIP_TOPK / small-GEMV batch / HC-up / Tile-J MMQ | Four sub-2% kernel ideas | Each measured 1.2–1.9% or ceiling ≤2–3% — under the predeclared bar |
| rocWMMA QSA decode scorer | Tensor-core decode scorer | 2× slower (the scorer is ~0.1% of decode anyway — ceiling analysis should have caught this pre-build) |
| IQ2_XS anomaly hunt | Why is the smaller pack 30% slower? | Kernels exonerated — per-tensor **"format lottery"** in the pack itself: same format class, wildly different per-tensor performance |

## The pattern worth internalizing

1. **Ceiling-analyze before building.** The QSA decode scorer cost a build to learn it was 0.1%
   of decode. A per-node time table (one trace window) prices the ceiling for free.
2. **Bitwise-perfect does not imply faster.** Two prototypes here were output-identical and
   still lost (expert modes ~3× slower; grouped GEMV 0.90×). Exactness is the entry ticket,
   not the win.
3. **Paired measurement is not optional on this hardware.** The Q8_0 layout case flipped sign
   *twice* depending on measurement discipline: folklore said good, cross-process sweep said
   +5–17%, paired in-process said −17 to −22%. See [../lessons/measurement.md](../lessons/measurement.md).
