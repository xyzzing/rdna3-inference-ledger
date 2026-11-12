# Methodology — the benchmark protocol and the ratio-definition ledger

This page is the reason the corpus in [rejected/](rejected/) is trustworthy. Two parts: the
protocol every experiment ran under, and a correction ledger that pins the exact definition of
every number quoted across this repo (because ratio wording is where optimization claims go to die).

## The protocol

1. **Paired interleaved arms, one process, one window.** Baseline and candidate alternate
   (ABBA) within the same engine instance. Cross-process sweeps drift — a lead measured as
   "+5–17%" across processes reversed to "−17 to −22%" under paired re-measurement
   (the upstream Q8_0 layout case). Within-window ratios are the only valid currency;
   absolute tok/s is texture, not evidence.
2. **Predeclared gates, never relaxed.** Example promotion bar: median paired ratio ≥ 1.020
   with a bootstrap CI excluding 1.00 (fixed seed recorded per experiment). If correctness
   cannot be established under the predeclared contract, the item stops — the gate is not
   widened to manufacture acceptance. Every rejection in this repo cites its gate.
3. **Keep every raw row.** Failed runs, VOID windows and exclusion reasons are records, not
   garbage. Two of the corpus's most-cited findings came from VOID windows whose *diagnostic
   payload* was the real result (0046a: the a=0 window rate 4.9–7.5%).
4. **Uninstrumented wall time is the performance authority.** Instrumented traces
   (rocprof, phase taps, per-node tables) attribute and explain, and must disclose their own
   perturbation. No performance claim is promoted from a trace alone.
5. **Correctness gates come first and are bitwise where achievable.** The strongest negatives
   here are bitwise-perfect prototypes that still lost on speed (grouped multi-shape GEMV:
   9/9 outputs exactly equal, 0.90× latency ratio (≈1.11× throughput — faster, but the gate needed ≥2×) — closed at the bench gate without spending
   a serve window).
6. **Engagement verification before verdicts.** Grep the arm's engine log for the route's
   annunciator before trusting any A/B result. One entire window (0046a) was VOID because the
   D arms silently ran the baseline path. And pin the *production* dispatch configuration
   before choosing any baseline — env-gated branch ladders mean "the baseline" is a selector
   choice (the i8 dense screen's 1.39× was real but measured against a path production never runs).

## Ratio-definitions ledger

Every ambiguous figure in this repo, with numerator/denominator/scope:

| Figure | Actual definition | Direction / reading |
|---|---|---|
| Grouped-GEMV "0.90× median, 0.72× best" | **grouped_time ÷ baseline_time** per family×ncols config; baseline = 8 back-to-back per-tensor GEMV launches timed as one event span; grouped = one descriptor-table launch; interleaved ABBA per rep, **min-of-12** per arm; K-quant synthetic weights, fixed seed | **Latency ratio, lower is better**; the predeclared bar was grouped ≤ 0.5× baseline (i.e. ≥2× faster, the one-launch-over-N hypothesis's whole point). On this definition 0.90× = 10% less latency (≈1.11× throughput) and 0.72× = 28% less (≈1.39×) — the prototype WAS faster in every config (raw: e.g. 0.1520→0.1094 ms), just far short of ≥2×. The close: real-but-insufficient, not slower |
| "parity bitwise 9/9 (max_rel 0)" | max_rel over ALL member outputs, grouped vs per-tensor baseline, identical weights/activation | exactness evidence, not performance |
| 72.7% device busy (single request) | rocprof busy-union ÷ decode-burst span (one pass, 94K, burst 2,723 ms) | **regime-specific**: single-request decode. NOT comparable to the multi-request figure below |
| "65% host-fed" (older figure) | 1 − busy over a multi-request serve burst | different serving regime; both figures stand, do not mix them |
| 1.62% serial plan/stage | (build→staged + staged→launch phase segments) ÷ (sum of all four phase segments), decode windows, verified build | share of the decode-loop wall occupied by pre-launch host work |
| 0/10 rescue rate | top2 runner-up hits ÷ recorded chain breaks (depths 0–3, one window, diagnostic flag) | sampling basis 10 breaks; rule-of-three CI bound ≈ 26% — used as the robustness bound in the Sequoia close |
| "+12.5%" ONECOMMIT rate | lead-arm predicted rate ÷ baseline-arm median, fresh-instance window | **INVALID as performance evidence**: the arm's outputs differ (parity fail) → not equivalent work. Retained only as motivation for the follow-up investigation |
| "17.1% of decode kernel time" (multi-shape GEMV) | Σ kernel durations of the multi-shape kernels ÷ total kernel time in the decode burst | kernel-time share, NOT wall share; wall conversion requires the busy share |

## Known instrument traps (each cost at least one window to learn)

- **Fresh-instance temp-0 outputs are not run-to-run stable**: the same frozen fixture yields
  110–222 completion tokens across fresh instances (cache-state numerics). Cross-instance
  byte-parity gates are invalid; 2%-class claims need token-matched or within-process
  measurement.
- **The first traced launch after an engine restart is invalid.**
- **A KL=0 parity tell is only meaningful when the route touches the prefill path** — draft-config
  changes leave row 0 identical by construction; verify engagement via depth stats/annunciators instead.
- **Build vintage matters**: worktree builds measured 11–12% slower than repo builds. Every number
  here carries its build provenance; don't compare numbers across vintages.
- **Attribution must segment by stream and by kernel-name namespace** — launch counts and busy
  time mislead when admission-prefill kernels interleave with decode on adjacent streams.

See [lessons/measurement.md](lessons/measurement.md) for the fuller rule list with the stories behind them.

## Corrections log

**Nov 12, 2026 — after an external review of commit 49aa1d9.** The reviewer distinguished
checkable arithmetic from reported measurements; five corrections applied:

1. **Grouped-GEMV interpretation reversed** (this page's ratio ledger, rejected/, open-upstream/):
   on the published definition (grouped_time ÷ baseline_time, lower is better), the prototype's
   0.90× median / 0.72× best meant 10–28% LESS latency (≈1.11–1.39× throughput) — it was faster
   in every config, not slower. The rejection stands on the real reason: far below the
   predeclared ≥2× (≤0.50×) gate. The raw per-config rows were re-checked before rewording
   (e.g. baseline 0.1520 ms → grouped 0.1094 ms).
2. **The first token-matched window's positive sign withdrawn** (campaign page): the naive
   candidate-vs-free-baseline pairing was inflated — the free-running baseline runs
   systematically slower (~15–25%) than any forced role, so it flatters any candidate. The
   published reconciling block row (slot/arm/follow/committed/wall) is in the campaign page;
   the drift-matched negative stands (+10–13% slower, 14/15 and 10/10 blocks, floor ±0.5%).
3. **WMMA summary split** (where-we-are): the select scorer (10–11× kernel, +8.7–11.2% e2e) and
   prompt attention (+22.8% prefill @128K) were conflated in one summary table.
4. **ONECOMMIT arithmetic tightened** (campaign page): 43 windows × 0.273–0.35 ms = 12.9–15.0 ms
   = 0.42–0.48% of the 3,102 ms decode wall (the 2% bar needs 60.8 ms). The older page's
   output-change attribution now links directly to the superseding controlled result.
5. **"Serialization floor" softened** (where-we-are): the 72.7% device-busy figure is one
   94K single-request pass; busy is not optimality — the trace localizes where to look, it does
   not bound the busy portion's headroom. Decode-rate figures are now labeled by workload
   (@1K peak vs @94K deployment) wherever they appear together.

The reviewer's confirmation is recorded: the lookup-bypass close survives review — conversions
occurred on the intended edit workload and the implemented bypass still lost on matched
committed tokens (14/15 dev, 10/10 held-out). Scope wording uses "all implemented forms".
