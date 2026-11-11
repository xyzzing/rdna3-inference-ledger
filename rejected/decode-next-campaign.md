# The decode-next campaign (2026-11-10/11) — twelve experiments, zero promotions

A final campaign reopened the board with five tracks (leads from a screen, host-side plans,
grouped GEMV batching, compact speculative state, agent-session reuse, retention, plus the
parked Sequoia item). **Result: every track closed honestly — no promotions, deployed stack
untouched, production restored after each measurement window.** This campaign is published
because it is the corpus's cleanest demonstration of the method: each close cost one
predeclared gate or less, and two tracks closed *without building anything*.

| Experiment | Plain description | Result |
|---|---|---|
| CONVAB | Fuse GDN conv + a/b projections into one launch inside the captured verify graph | **Engaged** (traced census: −21,496 kernel dispatches), outputs bit-identical (5/5 text parity) — but decode **0.9185** CI [0.894, 0.986] @94K, 0.844 @32K guard. Matches upstream's own −0.3% note on a different gfx target. Supported negative; stays default-off |
| GDN_NOY | Skip a dead FP32 norm store (prefill-only flag, off on gfx1100 by default) | **Inert on this stack** — the arm byte-exact to baseline *including prefill* (prompt_ms identical to the millisecond). A prior "+4.0%" lead does not exist on this engine vintage. Not reproduced |
| ONE_TOKEN_COMMIT | Skip the separate commit graph after one-token windows (CUDA default-on, HIP opt-in) | Decode rate **1.1250** CI [1.095, 1.207] — but **outputs change** (text 0/5 identical, acceptance signature 47/50 vs ~33/37): the HIP self-commit route alters state vs the commit-graph route. Parity FAIL → BLOCKED; **the +12.5% is unvalidated and must never be quoted** (see [../methodology.md](../methodology.md) ratio ledger). Divergence later located at the first decode window, recurrence half only |
| Grouped multi-shape GEMV | One launch over the co-ready dense-projection families via a descriptor table | Prototype **bitwise-perfect 9/9** (incl. odd-tail member) — but family latency ratio **0.90× median, 0.72× best** (bar ≤0.5×): launch count is NOT the mid-shape gap, the K-quant dequant-bound kernel efficiency is. Closed at the bench gate, no serve window spent. **Premise corrected first**: the projections are Q6_K/Q5_K/Q4_K per layer, not Q8_0 — an older gap analysis had priced the wrong type's kernels |
| Host plan separation | Hoist/reuse the per-window host routing/planning work | Measured *before* patching (phase taps extended): the serial plan/stage slice = **1.62% of decode wall** — below the predeclared 2% stop rule. Closed without a patch |
| Compact speculative state | Commit bounded update data instead of per-position recurrent state | **DUPLICATE** — audit showed the deployed commit is already selective (accepted-rows-only replay, −1 = keep-old, tail snapshots; state 3.12 MiB per layer, never per-position). Closed by audit, no build |
| Exact session reuse | Audit + probe two alternating agent sessions at depth | **Already works** — turns resume from 20-token suffixes (31,320 cached / 4,695 computed) while alternating at 31K depth; turn wall 11.45 s → ~0.9 s (**13×**); evictions 0. Closed by audit + probe |
| Recovery-unit retention | Bounded checkpoint retention under memory pressure | Gated on demonstrated pressure — **none exists** (evictions 0); not run |
| Sequoia tree verification | Draft a tree, verify all paths in one window | Hedge rescue **0/10** chain breaks; acceptance model: every topology 1.0000× linear at S=8, robust to the 26% CI bound. Structural audit: feasible but a 3-item multi-session port — priced, not scheduled. Closed measured-negative |
| Vehicle finding | Fresh-instance interleaved A/B sensitivity check (the mandated A–A pilot) | **Cannot resolve 2%**: the same frozen fixture yields 110–222 completion tokens across instances (cache-state numerics); M-pair CI halfwidth 10.9%. Any future 2%-class claim needs token-matched or within-process measurement — bar never lowered |
| Attribution correction | Single-request vs multi-request decode anatomy | Single-request 94K decode burst = **72.7% device busy / 27.2% exposed** (19.0% host-side + 8.3% API-blocking, 86% of it spin-bracketed); prefill is 92.8% busy. The older "65% host-fed" figure belongs to the **multi-request serve regime** — both stand, do not mix |
| Combined-baseline campaign | Confirm surviving leads against the original baseline | **Not run — correctly**: no lead survived its gate, so the combined arm's precondition never triggered |

## Durable findings of the campaign

1. The **K-quant type correction**: the dense projections are Q6_K/Q5_K/Q4_K per layer — which
   also voids the older Q8_0-framed gap numbers for these shapes. *Type-verify your tensors
   from the manifest before pricing a kernel idea.*
2. The exact verify-window node census from graph records: window graph ≈ 2,200 nodes,
   drafter 58–79, commit 5.
3. The fresh-instance completion-length spread (110–222 tokens) as the quantified reason
   cross-instance byte gates fail.
4. The ONE_TOKEN_COMMIT state-route divergence with ranked causes and a state-hash instrument.
5. Session-reuse machinery is *already strong* — no product gap at agent depths; measure the
   existing thing before building the new thing.
