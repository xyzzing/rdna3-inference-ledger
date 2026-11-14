# Campaign update — Nov 14, 2026 (window-1 arc): preparing the one deciding window, and a block-verification idea measured INAPPLICABLE

Continues [the no-GPU round](campaign-2026-11-14.md). One deciding correctness-only capture
(window 1) prices the two remaining IQ3_S kernel ideas at once: the **Direct4 integer-dot cache**
(format proven, exposure BLOCKED on two named counters) and the **donor's window-shape arm gating**
(the one transferable EXL3 mechanism). This arc prepared that capture end to end on CPU — and
closed a whole idea class on the way.

Verdicts (scoped): preflight **BLOCKED→prepared** · block verification **INAPPLICABLE** ·
instrumentation **implemented, review-gated, re-request READY** · the GPU window **requested, not
granted by us — it runs when the operator grants it**.

## The preflight: a worker hit a hard context ceiling, and the frontier finished the audit

The first two packets of this arc died inside the local coding worker: its conversation window is
**131,072 tokens, and four runs across two rounds died at that ceiling** (the last 428 tokens
over, ~10 minutes into each attempt, ~12.5M worker tokens with zero output) — a context-bounded
packet body still does not fit once the worker's fixed scaffolding takes its share. The recorded
protocol covers this: the frontier finished the audit itself and **disclosed the executor change**
in the record. Two lessons now live in [toolchain lessons](lessons/toolchain.md): the salvage
pattern (verify the worker's partial output, reject skeletons, finish from pinned evidence) and
frontier-direct execution with disclosure when the worker is functionally unavailable.

The preflight's answer: **go=false for the capture as originally imagined** — three required
fields had no existing zero-code tap (the per-window `(n_groups, n_ent, widest)` triplet, the
executed arm/width/layer linkage, and any stable request/window key at all — the engine's
per-request lines are positional, which is exactly how an earlier round's client-field staleness
happened), and the verifier's expert stage is **inclusive**: it wraps gate/up + SwiGLU + q8_1 +
down in one stamp, so from stock rows the gate/up-only share is a **14.0% upper bound** on the
one captured request, nothing tighter. Everything else needed (stream readiness, residency
hits/denominator) already joins per request.

## Block verification is INAPPLICABLE for greedy production — measured, not guessed

The donor ecosystem shows block verification committing more tokens per window. On our frozen
production cells it cannot help, and the enumeration says why precisely:

- Production's frozen cells run **greedy**: target and draft laws are point masses, so a proposed
  prefix survives exactly until its first token mismatch — **a block verifier commits the same
  prefix as the existing equality scan on every path**. Zero gain is forced, not empirical.
- Exact enumeration over adversarial synthetic cases (unequal p/q, disjoint support, full
  acceptance, partial rejection, close-drafter) confirms both laws emit **target-exact sequences
  (TV ≤ 1.1e-16)** while the *accepted-length* laws differ: the block law commits +0.40
  tokens/window where the drafter is nearly as confident as the target — a real gain, under
  sampling laws production does not run.
- Naively applying the donor rule to the gated path's masked proposal measure is **not**
  target-exact (TV 0.0667). Reopening requires a real stochastic workload, its actual proposal
  law, and a new target-exact proof for that law.

Donor pinned (commit + license recorded). No HIP port, no engine fork, no prototype requested —
the idea class is closed **for frozen greedy production** with its exact reopening conditions.

## The three default-off taps — and a design premise that did not survive contact with the source

The preflight specified three minimal instrumentation changes; the implementation (frontier-direct,
disclosed) kept them exact and corrected one premise on the way:

1. `STRATA_WINDOW_SHAPE=1` — one stderr row per window per expert layer:
   `strata winshape: req=R win=W T=t L=l kind=k arm=a ng=g ne=e nw=w` (n_groups, entries, widest
   group, executed arm native/nibble/s2). The preflight assumed the triplet was known host-side at
   plan publish — **the source says otherwise**: the common all-resident path builds its plan *on
   the device*, and the device plan block is a per-group scratch every layer reuses. The triplet is
   therefore computed by the same device kernel that builds the plan and collected at the per-window
   sync; the CPU-pool path republishes asynchronously and emits `-1` rows (disclosed; that path is
   outside the resident share the pricing needs anyway).
2. `STRATA_REQUEST_STAMP=1` — an in-process request ordinal stamped onto the three per-request
   lines (`; req N`) and into every winshape row, giving the parser its stable join key.
3. `STRATA_VERIFY_FINE=1` — a two-stamp pair inside the grouped-expert call bracketing the gate/up
   launches, so the gate/up-only share finally separates from the inclusive stage row (reported as
   "fine gate/up X of Y ms/window" per layer kind).

All default off; with the envs unset the hot path gains nothing (no allocation, no launch, no copy,
no formatting, and the request counter does not even tick). Built in an isolated worktree off the
pinned commit with the production build recipe recovered from the recorded build cache (one more
environment lesson: a desktop-app image exporting `LD_LIBRARY_PATH` silently breaks the system
cmake — clear it).

## The parser fails closed, and the fresh-eyes review earned its keep

The field parser joins rows by (request, window, layer) and **fails closed on ambiguity**:
duplicate keys, orphan rows, zero ordinals, duplicate request anchors — exit 3, never averaged
away. Missing rows are reported as coverage, never synthesized. Stock (unstamped) lines are
skipped as legacy, never joined by position.

An independent review with fresh eyes then **rejected the first build on two real defects**, both
fixed before any GPU request:

- the widest-group value was read across threads with no barrier — silently stale numbers from the
  reused plan scratch would have polluted the shape table undetectably;
- the env combination `WINDOW_SHAPE` without `REQUEST_STAMP` orphaned every row (anchor lines now
  stamp whenever the ordinal ticks, and the runbook pins the full four-env set).

Three lesser findings were disclosed rather than coded around. This is the second time this
campaign that a reviewer-only context caught what the author could not — the first (a verdict
label correction) changed a decision row; this one protected a GPU window.

## What one window decides, and the NOT_RUN ledger

The re-request package is complete: the instrumented binary (its own hash recorded), stock-equivalent
config, the frozen cells at actual occupancy, a clean control (same binary, tracing off), the
watchdog restore plan, and the pinned four-env set (no full-vocab copies). One diagnostic-only
overnight run decides: the gate/up-only share f and eligible resident coverage h (the Direct4
exposure arithmetic — close if even the optimistic bound misses the 2% bar), and the shape
frequencies that would switch arms (the donor gating candidate — priced or closed). Device-side tap
emission, the joins on real output, and correctness against the clean control are all still
NOT_RUN; the window itself is **requested, not granted** — it runs when the operator grants it.

Closed negatives untouched. The block-verification closure and the preflight's upper-bound figure
are recorded in [rejected/drafter.md](rejected/drafter.md) and this page respectively; the
toolchain lessons gained four entries ([lessons/toolchain.md](lessons/toolchain.md)).

## Addendum (same day): the window RAN — taps emit, keys clean, and the pool path turned out to be the only path

The operator granted the re-request. Five diagnostic passes ran against the instrumented binary
(one control + two instrumented + three debug/minimal; every pass restored production and verified
its fingerprint; ~35 min GPU total):

- **All three taps emit in serve.** ~170k keyed per-(request, window, layer) shape rows per pass;
  per-request anchor stamping works; the parser joins clean on the real capture after two recorded
  rule refinements (req=0-only slices are the batch-warmup bucket; mixed slices are the slicer's
  region spanning a warmup and the next keyed request — true anomalies stay hard-fail).
- **Correctness vs the clean control holds**: committed-token counts differ on only the
  known-nondeterministic rows (sacrificial warmups + the p-reg EOS-knife-edge cells); every 94K
  cell and every p-code/p-zh cell matches.
- **Two real instrumentation defects were found and fixed in-flight** (a stamp-slot index bug that
  silently zeroed the gate/up denominator; an unsound cross-buffer span comparison — the per-window
  copy races the pipelined next window's execution), both by a one-pass debug print and both fixed
  before the definitive pass.
- **The honest outcome**: the deployment's windows turn out to run the CPU-pool plan path on every
  window (the expert cache is a hot set — whole-model residency never holds), so the device-side
  triplet route covers a path that never executes and the shape rows printed `-1`; and the fine
  gate/up spans are internally inconsistent with the established stage rows by ~4.7×, which the
  session could not attribute. Per the labeling rules: **B1 stays upper-bound-only** (inclusive
  expert-stage bound ≈ 12-13% of the decode wall; with h ≈ 0.985 the optimistic bound clears the
  2% bar at s ≥ ~1.16 — but f itself is unpinned), and **D0's shape table stays BLOCKED** on one
  precisely-named missing piece: the pool-path host stamp at the planner publish, where the
  (groups, entries, widest) triplet is already known host-side.

The arc closes the day with the deciding capture half-won: the instrumented harness works end to
end, the joins hold on real data, and the remaining gap is one small host-side stamp plus a
resolved stamp-span semantics question — CPU-daytime work, one more window pass away.

## Addendum 2 (same day): the shape table is IN — the donor arm-gating candidate is quantified

One minimal minder packet (exact anchors, the size class that fits the worker — passed on the
first attempt at 1.8M tokens after the oversized packets had burned 12.5M) added the pool-path
host stamp: the CPU planner writes the (groups, entries, widest) triplet at publish into per-layer
host slots, and the collector reads them. The definitive pass:

- **143,232 keyed layer-windows, zero missing triplets**; joins clean; correctness vs the clean
  control unchanged (only the known-nondeterministic warmup + p-reg knife-edge rows differ).
- **43.5% of layer-windows are all-singleton** (ne == ng) — dominated by T=1 windows (43.6% of all
  rows; a T=1 window's 10 routed entries are mostly distinct experts, so only 0.5% of T=1 windows
  pass the donor's reuse gate).
- **64.1% of the multi-token (T≥2) windows pass the donor width-4 gate** (widest group ≤ 4 AND
  entry reuse ≥ 1.2) — 36.4% of all layer-windows (width-3: 25.3%, width-2: 12.4%).

**The window-shape arm-gating candidate graduates from BLOCKED to quantified and bench-ready**: a
width-4 shape rule engages the multirow arm on ~36% of layer-windows, so the +2% end-to-end bar
needs ≈ **5.4% per-shape saving on the passing windows** (unweighted share; per-shape time
weighting is still NOT_RUN). That is a real number the isolated paired bench can now be judged
against — competing for the one bench slot with the Direct4 kernel (B2), whose exposure question
(f, the gate/up-only share) remains the one open measurement-semantics question: the fine-stamp
spans and the established stage rows disagree ~4.7× for the same code span, and B1 stays
upper-bound-only (inclusive bound ≈ 12-13% of the decode wall; the optimistic bound clears the bar
at s ≥ 1.16) until that is attributed.

## Addendum 3 (same day): the discrepancy attributed, f pinned — and the bench slot decided by measurement

- **The 4.7× fine-vs-stage discrepancy is ATTRIBUTED**: the pool path calls the grouped kernel
  twice per layer (VRAM share + PCIe share) and the fine stamps fired on both — the near-empty
  PCIe call overwrote the triple. One line fixed (VRAM-share only); post-fix the fine denominator
  matches the stage rows within ~5-10% — **the stage rows were right all along**, and gate/up is
  ~55% of the expert stage.
- **B1's arithmetic, pinned on the definitive pass** (18 requests, six cells): **f = 7.56% of the
  decode wall** (range 7.0-8.3%, depth-invariant), h = 0.986, **f×h = 7.46% — clears the 2% bar by
  3.8×**. The Direct4 kernel's go/no-go number: **local speedup s ≥ 1.36** on the gate/up dot.
- **The bench slot decided by measurement** (the existing expert-bench harness, real layer-17
  weights from the pinned shard, byte-for-byte parity first, prod stopped for clean timing): the
  donor-class fused arm is **slower** than the incumbent on every passing shape (+10% to +26%
  all-span; every other arm +13% to +85%), bitwise equal throughout. **D1 closes as a scoped
  negative** — production's default arm is already the best arm for the passing shapes (reopening:
  a new kernel ≥1.60× on those shapes, bitwise-equal). **The bench slot goes to B2 (the Direct4
  symbol-dot kernel)** with its concrete bar: beat the grid/sign-reconstruction dot by ≥1.36×,
  or close on arithmetic.
