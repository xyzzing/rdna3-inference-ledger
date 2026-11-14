# Where we are — the plain-language guide

*Everything below is measured on one machine: a Ryzen 9 7900X + **RX 7900 XTX** (24 GB), serving
**Qwen3.8 Flash-Next** (a Mixture-of-Experts model) from a mixed 3–6-bit quantized
pack (IQ3_S experts; Q6_K/Q5_K/Q4_K dense projections) on NVMe. Numbers are wall-clock, measured, and reproducible per [methodology.md](methodology.md).*

---

## How local AI generation actually works (60 seconds)

When you send a long prompt, two very different things happen:

```mermaid
flowchart LR
    A["📄 YOUR PROMPT<br/>(up to 94,000 tokens)"] --> B["**PREFILL**<br/>read the whole document<br/>~3,400 tok/s<br/><i>compute-bound</i>"]
    B --> C["**DECODE**<br/>write the answer<br/>one token at a time<br/>~100 tok/s @1K depth; 43–47 @94K<br/>(distinct workloads)<br/><i>memory-bound</i>"]
    C --> D["✍️ YOUR ANSWER"]
```

> **A note on decode numbers**: the matched-depth protocol (Nov 12/13) measures engine
> decode at **79–94 tok/s @70–98K** — a different token-counting definition and engine vintage
> than the 43–47 @94K figure quoted elsewhere in this ledger. Both are recorded with a
> provenance-gap note; do not mix them in one comparison.

**Prefill** is reading — every prompt token flows through the whole network once. Big matrix
multiplies, loves tensor cores. **Decode** is writing — for every single output token, the
hardware must essentially re-read the model's weights. Decode speed is almost entirely
determined by *how fast you can move weights from memory*, which is why a 7900 XTX's ~960 GB/s
of memory bandwidth is the wall everything else fights against.

Two tricks make decode survivable:

- **Speculative drafting (MTP):** a small draft head guesses the next ~5 tokens; the big model
  verifies all 5 in one pass and keeps the ones it agrees with. Accepted tokens are nearly free.
- **Mixture-of-Experts:** most of the "thinking" happens in a library of 60+ GiB of specialist
  mini-networks ("experts") — only a few fire per token. They can't all fit in 24 GB of VRAM,
  so the engine keeps a hot cache and streams the rest from NVMe/RAM.

---

## The scoreboard

| | count |
|---|---|
| Recorded experiments | **~129** |
| Adopted into the deployed stack | ~20 |
| Rejected with paired measurements | **~85** |
| Timing windows run | one at a time, always with a pre-registered gate |
| Production deployments | the ~20 adopted changes only |

## What we made faster (the deployed stack, measured individually)

| change | what it does | measured |
|---|---|---|
| Quantized-GEMM prefill path (MMQ) | prefill reads the prompt with integer math | **+112% prefill** |
| hipBLASLt tuning table for gfx1100 | the GPU's GEMM library had no tuned settings for this card; we measured 32 | **+82% prefill @128K** (merged upstream) |
| Tensor-core attention, select scorer (WMMA) | block-scoring attention candidates on matrix cores | **10–11× kernel**, +8.7–11.2% prefill @128K end-to-end |
| Prompt-attention port (WMMA) | the newer-card full-attention kernel, back-ported to RDNA3 | **+22.8% prefill @128K** (attention's share of prefill: 27% → 9%) |
| ROUTE_TAIL_SKIP | skip loading experts that no token in the window routed to | **+14–19% decode** — the biggest single decode win |
| Adaptive KV ring | the conversation memory window resizes to the request | **+9–11% decode at depth** |
| `--prefill auto` + PA_FAST/FUSED/RING512 | chunk long prompts; three prefill knobs | **+26%**, then **+13.8%** combined |
| mmap'd expert weights | stop pinning 48 GiB in RAM (3 OOM kills fixed) | stability; cold-start trade |
| Deeper drafting (`--spec 5`, `--lookup-chain 2`) | draft more tokens per verify | **+6–20% decode** |

Net: **2.1–2.3× faster decode than llama.cpp on the same GPU and weights**; prefill went
32 → ~3,400 tok/s over the campaign.

---

## Where we hit the ceiling (all measured, not assumed)

These are the findings that matter most to future work — each one is a door we measured shut:

| area | what we tried | why it's a ceiling |
|---|---|---|
| **The drafter's accuracy** | deeper draft chains (8 tokens), entropy-based early stop, cost-aware drafting, tree verification, an alternative drafter (DFlash2), near-certain-only drafting | the draft head already agrees with the big model **94% of the time**; every extension drafts more, pays more in verification, and comes out flat or negative |
| **Skipping the commit step** (ONECOMMIT) | don't launch a commit pass for 1-token windows | **≤0.49%** of decode wall — the 2% bar needs 3× that. Closed by arithmetic; never timed |
| **Q6_K weight decoding** (dense projections) | hoist scale loads to registers; decode each weight block once for all verification rows | bitwise-correct, **1.015–1.017× family speedup vs the 1.30× needed** — the decode work hides under memory traffic |
| **Host readback** | is the CPU waiting on logits it doesn't need? | the engine already reads **only token IDs** — confirmed from device traces |
| **Expert branch overlap** | run shared and routed experts concurrently | they're **already on separate streams**; what's left is serial by real data dependencies |
| **Window re-shaping / pre-draft skipping** for a session-lookup drafter | replace neural drafts with session-history drafts on repetitive text | drafts **convert** (24% accepted) but every implemented form measured **−4 to −13% slower** drift-matched — accepted drafts ≠ cheaper windows |
| **94K depth penalty** | attribute the deep-context slowdown; screen the one bitwise-preserving head candidate | **attributed — proposal-side** (offered width shrinks at depth; acceptance holds at matched width); the repack screened **−7% to −41%** at every width — closed on this stack, reopening = a proposal-width mechanism |
| **Cross-run comparisons** | compare separate engine instances | the first request after a start runs **~12% faster** (freshness artifact) and identical prompts produce different token counts across instances (110–222!) — only within-process, token-matched comparisons are valid |

The pattern across all of these: the device is busy **~73% of the decode burst** (single
request, one measured pass) and every scheduling/overlap lever attempted so far measured flat
or negative — the remaining gaps are structural hand-offs, not idle waste. Two honest limits
on that statement: the busy 73% is one measured pass, not a proof of optimality (a busy GPU
can still run an inefficient schedule — the trace localizes where to look, it does not bound
how much faster the busy part could run); and the busy figure is regime-specific (single
request; multi-request serving measured differently). That's why ~85 ideas measured negative:
the easy wins were taken first, and the rest have so far resisted measurement.

## What's genuinely left open

| area | status |
|---|---|
| A device fault in chunked prefill (bug filed upstream, [#5539](https://github.com/Niko1221/Strata/issues/5539)) — the fix would be ~45% faster long prompts | waiting on upstream |
| A compiler bug that silently fuses rounding steps ([ROCm #5330](https://github.com/ROCm/llvm-project/issues/5330)) — matters for anyone needing bit-exact GPU math | waiting on upstream |
| Multi-request serving (two jobs at once) | headroom measured: a *short* second job fits; two full 94K sessions don't |
| Two pricing gaps, one capture from resolution | the IQ3_S→Direct4 integer-dot cache (**format proven**: the codebook is 16 signed levels, exact mapping exhibited; exposure `f×h` unpriced) and window-shape arm gating from an EXL3 donor audit — the capture's instrumentation is now built and review-gated (**re-request READY**, operator grant pending), and block verification was measured **INAPPLICABLE** for greedy production on the way ([window-1 arc](campaign-2026-11-14-window1.md)) |
| Time-to-first-token as its own product | separate objective, separate scorecard |
| The next model generation | see below |

---

## Does any of this apply to Qwen 4?

Honest answer, in three tiers:

### 🟢 Transfers directly (engine + hardware level — most of the value)

Everything that lives at the **RDNA3/HIP/hardware** layer is model-agnostic and carries over
to any model served on this stack, including a future Qwen 4 MoE:

- the whole **deployed prefill stack** (MMQ, the hipBLASLt tuning table, WMMA attention, chunking) —
  these accelerate matrix math and attention, which every transformer uses;
- **ROUTE_TAIL_SKIP, the adaptive KV ring, expert mmap/cache, the cgroup valve** — MoE serving
  mechanics that any MoE needs;
- the **measurement protocol itself** — paired arms, pre-registered gates, the drift probe,
  the freshness artifact, engagement checks. This is arguably the most valuable export:
  ~85 of ~110 ideas failed, and the protocol is why each failure was cheap;
- the **toolchain findings** — the compiler's silent FMA contraction
  ([ROCm #5330](https://github.com/ROCm/llvm-project/issues/5330)), the AMD hip-header build
  matrix, the build-isolation gate. Bugs don't care which model you serve.

### 🟡 Transfers as *method*, not as numbers

The **drafter-side conclusions** (chain depth is not the lever, entropy gates have no
operating point, tree verification doesn't pay, session lookup loses to a 94%-accurate
neural drafter) were measured against *this* draft head's 94% top-1 acceptance. A Qwen 4
drafter with different acceptance behavior re-opens the questions — but you now have the
exact harness and decision rules to re-answer each one in one window instead of a month.
**The percentages won't transfer; the decision procedure will.**

### 🔴 Reopens from scratch

- **Quant-format performance**: the Q6_K/Q5_K/Q4_K dense-path findings are properties of *this
  pack's* formats. If Qwen 4 ships new quantization (or you pick different formats), the
  format-vs-kernel questions reopen — check the [rejected list](rejected/) first, because
  several "obvious" ideas (NVIDIA-style layouts, fewer-bytes KV) already lost here once.
- **Session lookup**: only pays where model outputs literally repeat session history (code
  editing, tool output). That's a workload property, not a model property — but it also has
  to *beat the neural drafter* on that workload, which it didn't here even at its best.
- **Expert-routing mechanics** (ROUTE_TAIL_SKIP-style skips, cache sizing): the numbers depend
  on the new model's expert count/size — the *mechanisms* stay, the tuning re-measures.

### The one-sentence version

> **Everything we learned about the GPU, the toolchain, and how to measure transfers to Qwen 4;
> everything we learned about this model's drafter and quant formats is a template for how to
> re-measure Qwen 4 fast — and the ~85 negative results are the checklist of what NOT to
> re-try without new evidence.**

---

*Questions, corrections, or "did you try X?" — check [rejected/](rejected/) first; there's a
decent chance the answer is yes, with a confidence interval.*
