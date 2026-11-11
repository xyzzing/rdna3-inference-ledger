# Open / parked / upstream (as of 2026-11-11)

The local board is fully dispositioned — the final campaign (12 experiments) produced **zero
promotions** and the deployed stack is untouched. Everything remaining waits on upstream.

| Item | State |
|---|---|
| Grouped multi-shape batching GEMV (the last kernel idea with credible headroom) | **CLOSED** — bench-gate negative: bitwise-perfect prototype but 0.90× median family latency ratio (bar ≤0.5×); premise corrected first: the dense projections are Q6_K/Q5_K/Q4_K per layer, not Q8_0. Committed as the bench-only recorded arm |
| Host routing/plan split (19–27% of the decode span was the older coarse number) | **CLOSED** — the finer split measured: per-window serial plan/stage = **1.62% of decode wall** — below the predeclared 2% stop rule; closed without a patch |
| Env-flag screen leads (CONVAB / GDN_NOY / ONE_TOKEN_COMMIT) | **CLOSED** — CONVAB engaged + bit-identical outputs but −8.2% @94K; GDN_NOY inert on this stack (a prior "+4.0%" lead not reproducible); ONE_TOKEN_COMMIT +12.5% rate but **outputs change** (the HIP self-commit route alters state vs the commit-graph route) → parity FAIL, BLOCKED; the +12.5% is unvalidated and must never be quoted |
| Sequoia tree verification ([2402.12374](https://arxiv.org/abs/2402.12374)) | **CLOSED** — measured hedge rescue rate **0/10** chain breaks (the runner-up never matches the target; 95% CI bound ≈ 26%); every tree topology 1.0000× linear at S=8; the 3-item port priced and not scheduled |
| Chunked prefill device fault | **45% faster when it works**, deterministic device memory access fault at the 3×32K chunk handoff (3/3 reproductions, single-chunk control clean) → filed upstream [issue #5539](https://github.com/Niko1221/Strata/issues/5539), parked there |
| Explicitly-rounded-intrinsic contraction (compiler) | 371 hits inventoried on deployed-path device code; upstream [ROCm/llvm-project#5330](https://github.com/ROCm/llvm-project/issues/5330) open; deployed-path items pending the fix decision |
| 262K context | Works: decode decays only 75.6→69.3 tok/s (128K→262K) — believed the only community row at that depth |
| Upstream records | Open: [PR #5458](https://github.com/Niko1221/Strata/pull/5458) (bench report), [#5459](https://github.com/Niko1221/Strata/issues/5459) (defaults row), [PR #5535](https://github.com/Niko1221/Strata/pull/5535) (i8 report), [issue #5539](https://github.com/Niko1221/Strata/issues/5539) (KVA fault), [PR #3919](https://github.com/Niko1221/Strata/pull/3919) (KV ring, merge-ready), [PR #2224](https://github.com/Niko1221/Strata/pull/2224) (PcieAuto, maintainer-ported); [ROCm/llvm-project#5330](https://github.com/ROCm/llvm-project/issues/5330); [hk-dsl#367](https://github.com/vivienfanghuagood/hk-dsl/issues/367). No local action pending; any maintainer response triggers the staged follow-ups |

See [links.md](links.md) for the full annotated link list.
