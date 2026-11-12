# Lessons — measurement and attribution

The rules that made ~80 negatives cheap to close. Each rule here was learned by paying for its
violation — usually one wasted experiment window. The protocol that operationalizes them is in
[../methodology.md](../methodology.md).

## Rules with their stories

1. **Paired in-process alternation is the only truth; cross-process sweeps drift.**
   The upstream Q8_0 layout showed "+5–17%" in a cross-process sweep and "−17 to −22%" under
   paired ABBA re-measurement. Same code, same card — the measurement discipline *chose the sign*.

2. **Grep the annunciator before judging an A/B.**
   An entire window was recorded VOID because the candidate-drafter arms silently ran the
   baseline path (the engine rejected the selector; the harness didn't check). The tell was in
   the data (D ≈ M) but the rule is cheaper: verify engagement in the arm's own log first.
   Related: a KL=0 parity tell is only meaningful when the route touches the prefill path —
   draft-config changes leave row 0 identical by construction.

3. **Pin the production dispatch configuration before choosing any baseline.**
   Env-gated branch ladders mean "the baseline" is a selector choice. One i8-GEMM candidate
   measured a decisive 1.39× against the naive dequant+GEMM path — and production runs a padded
   fp16 tensor-core branch ~4.5× faster than that. Component wins vs. naive baselines are not
   production candidates.

4. **Cross-instance byte-parity gates are invalid; fresh instances are not run-to-run stable.**
   The same frozen prompt yields 110–222 completion tokens across fresh instances at temp 0
   (cache-state numerics). Anything at the 2% scale must be measured token-matched or
   within-process. Corollary: the *first* traced launch after an engine restart is invalid.

5. **Regime separation: never mix figures across serving regimes.**
   "72.7% device busy" is the single-request regime; "65% host-fed" is multi-request serving.
   Both are real; mixing them produced a month of wrong priorities (the megakernel program
   targeted a gap that mostly didn't exist in the objective regime). See the
   [ratio ledger](../methodology.md#ratio-definitions-ledger).

6. **Ceiling-analyze before building.**
   A tensor-core decode scorer cost a build to learn it addresses ~0.1% of decode. One trace
   window with a per-node/per-kernel time table prices the ceiling for free and kills the idea
   (or licenses the build) before any code exists.

7. **Kernel wins don't survive the window; measure end-to-end before promoting.**
   EXPDEC: −21–34% kernel time, e2e −1.75 to −3.34%. The verify window re-prices everything
   that touches its graph. Promotion bars (≥2% e2e, predeclared) exist because sub-window
   wins are systematically illusory.

8. **Bitwise-perfect does not imply faster — but keep demanding bitwise anyway.**
   Two prototypes were output-identical and still lost (expert modes ~3× slower; grouped GEMV
   0.90×). Exactness is the entry ticket: it's what makes a speed rejection *final* instead of
   "maybe the quality loss ate it".

9. **Measure before patching.**
   The host-plan hoist looked worth building (a coarse attribution said 19–27% of the decode
   span). The finer pre-built measurement said the serial slice is **1.62% of wall** — below the
   2% stop rule. Closed without a patch. The cheap version of most experiments is a better
   instrument, not more code.

10. **Keep every raw row; VOID windows have payloads.**
    A VOID A/B (silent-baseline arms) yielded the a=0 window rate (4.9–7.5%) that killed the
    T=1-skip idea for free. Two low-outlier arms in the entropy sweep seeded the min-p question
    that closed the chain-collapse item.

11. **Attribution must segment by stream and kernel-name namespace.**
    Kernel-name namespaces identify the subsystem (prefill kernels interleaving with decode on
    adjacent streams inflate "decode" launch counts); a whole megakernel program dissolved once
    its "unfused seams" were traced to concurrent admission prefill, not decode work.

12. **Rule-of-three your zeros.**
    0/10 rescue events bounds the true rate at ≈26% (95%) — not zero. Design the gate to be
    robust to the bound (the Sequoia close was), and report the basis (n=10) next to the zero.

## The meta-lesson

Every rule above cost one window to learn and saved at least one afterwards. The campaign that
ran *after* all twelve rules were in force closed twelve experiments with zero wasted windows —
including two closes (audit-only, no build) and one bench-gate close that never touched
production. **The instrument compounds faster than the kernels.**

## Added Nov 12 (the 0056–0062 campaigns)

13. **The slot-1 freshness artifact**: the first request after an instance start runs ~12%
    faster than every later slot. Any baseline in slot 1 is systematically wrong — one
    candidate "won every block, +13–15%, CI above zero" and the drift-matched rerun reversed
    it to −10–13%. **The structural fix is a trailing twin**: a stock re-run of the identical
    token stream in the LAST slot measures position drift directly; judge the candidate
    against its own slot. The drift floor measured ±0.5% — tight enough to resolve 2%.
14. **Within-process beats cross-instance, and the floors are not interchangeable**:
    cross-instance A–A floors measured 12–60% per block on this fixture class; the
    within-process floor measured 8–33% (free-running roles) and ±0.5% (forced-identical
    token streams). Any 2%-class claim needs the forced-identical form — the engine must
    replay the baseline's committed ids (a follow pipe), not merely run the same config.
15. **A gate can be technically satisfied and still measure the confound**: the freshness
    gate (dump mtime must move past a baseline) was correctly refusing every run because the
    baseline was captured AFTER the dump was written (the engine writes before the serve
    layer transmits the response). The refusal message with the raw
    `mtime_ns`/`started_ns` values pinpointed it in one look — **print the raw numbers in
    gate refusals**, not just the verdict.
16. **A diagnostic write must never fail the request**: the first follow-screen lost its
    blocks because the engine `return`-ed on a failure to write a *diagnostic* file (a
    relative path, unresolvable from the engine's cwd). Diagnostic paths fail → log and
    continue; and absolutize every path you hand to another process.
17. **Executor disclosure under quota loss**: when the local-model runner's upstream account
    is quota-blocked, disclose the actual executor per packet (frontier vs local model) —
    never silently move a coding campaign to a different executor. (The local path itself
    works: the runner over the minder proxy serves from the local model; only the raw CLI
    hits the external billing account.)
