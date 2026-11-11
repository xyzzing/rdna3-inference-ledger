# Rejected — speculation / drafter

Context: production runs speculative decoding (MTP-style drafter + verify windows) at **94%
top-1 acceptance**, depth-3. That single fact closes most drafter-side ideas before they're
built — and the ones we built anyway confirm it. Raw window data for the recent experiments is
at the bottom of this file.

| Experiment | Plain description | Result |
|---|---|---|
| DFlash2 drafter | External drafter claiming +8–11% over MTP | **WASH** — median 0.95; *better* acceptance (0.87 vs 0.80) but shorter chains + heavier drafter. Acceptance rate is not the objective; tok/s is |
| min-p 0.99 chain collapse | Cut speculative chains to near-certainty only | **−15% slower** — the chains earn ~+17% at this shape |
| AdaEDL entropy gate | Stop drafting when drafter entropy is high ([2410.18351](https://arxiv.org/abs/2410.18351)) | Dead — drafter entropy **>8 nats on every step** despite 94% top-1 accuracy (top-1 match ≠ distributional sharpness); calibrated sweep 2.0→8.0 nats all collapse chains identically; no operating point exists |
| Deeper chains (spec 8) | Draft longer chains | Wash (0.996) — depth 5 finally tested and accepts at 94%, yet tok/s flat: the bigger verify-window graph prices out the extras. Chain depth is not the lever |
| T=1 skip | Skip verify plumbing when nothing accepted | Dead — a=0 windows are only **4.9–7.5%** of windows |
| Launch-ahead | Issue draft steps device-gated ahead of verification | **Slower** (median 0.9699, CI excluding 1.0) — legacy per-step spins already pipeline against device execution; the unconditional extra drafter steps cost more than the saved round-trips |
| spec 6 | Raw depth beyond 5 | −6.0% @128K |
| Custom 41K draft vocab | 5.4× cheaper draft GEMV (0.615→0.114 ms) | Still net **−1.37%** tok/s — the renormalization tax cut acceptance 3.6 points. Cheaper drafter ≠ cheaper draft |
| YaRN 512K context | Serve past native 262K | Decode −25% vs floor — MTP acceptance collapses 0.80→0.06 |
| MTP_TOP2 / hnorm / mtp-window / BATCH_F16 | Four drafter tweaks | All rejected — every one lowered acceptance |
| STRATA_SPEC_COST | Cost-aware per-width draft table | −13.45% @128K (noisy self-measured table over-drafted) |
| VERIFY_DEFER_CPU | Defer light CPU rows to the next layer | −3.6 to −11.4% (changed outputs → acceptance cost) |
| Cascade utility gating ([2506.20675](https://arxiv.org/abs/2506.20675)) | Gate draft steps by MoE utility | Never built — utility < 1 cannot occur at 94% acceptance |
| Sequoia tree verification ([2402.12374](https://arxiv.org/abs/2402.12374)) | Draft a tree, verify all paths in one window | **CLOSED measured-negative without building**: hedge rescue rate **0/10** chain breaks (the drafter's runner-up never matches the target; 95% CI bound ≈ 26%), acceptance model shows every topology 1.0000× linear at S=8 — robust even to the CI bound. The 3-item multi-session port priced and not scheduled |

## Raw window data (interleaved paired arms; only within-window ratios are valid)

### Deeper chains: lookup-chain 0 + spec 8

Arms L1–L5 = deep-chain config; M0–M5 = baseline. Depth accepted/tested per level (d1..d5):

| Arm | tok/s | d1 | d2 | d3 | d4 | d5 |
|---|---|---|---|---|---|---|
| L1 | 89.5 | 63/69 | 46/49 | 37/38 | 35/37 | **31/33** |
| L2 | 94.2 | 68/74 | 50/54 | 40/41 | 34/36 | **27/29** |
| L3 | 100.7 | 63/69 | 46/49 | 37/38 | 35/37 | **31/33** |
| L4 | 105.4 | 63/69 | 46/49 | 37/38 | 35/37 | **31/33** |
| L5 | 105.2 | 63/69 | 46/49 | 37/38 | 35/37 | **31/33** |
| M0–M5 | 88.1–101.4 (median 96.3) | — | — | — | — | depth 5 never reached |

**Verdict:** WASH — median ratio 0.9958, CI [0.879, 1.067]. Depth 5 accepts at **94%** (31/33)
yet tok/s is flat. The verify window is the pricing mechanism.

### AdaEDL entropy-gate calibration sweep

Arms at H_max = 2.0 / 3.5 / 5.0 / 6.5 / 8.0 nats vs baseline. The gate ENGAGED in every arm —
and collapsed chains to depth 1 at *every* threshold:

| Arm | H_max (nats) | tok/s | chain shape |
|---|---|---|---|
| E1a/E1c | 2.0 | 93.8 / 89.6 | d1 ~82/86, then 1/1 stubs |
| E2b/E2c | 3.5 | 94.9 / 94.7 | d1 73/77, then stubs |
| H1 | 5.0 | 93.3 | d1 87/91, then stubs |
| H2 | 6.5 | 93.7 | d1 87/91, then stubs |
| H3 | 8.0 | 87.9 | d1 85/88, then stubs |
| M0–M5 | — | median 89.0 all arms, 96.4 excl. 2 outlier arms | normal chain shape |

**Verdict:** scoped negative — the distribution entropy is non-discriminative; the feature is
all-or-nothing. Mechanism stays default-off as the recorded arm.

### DFlash2 one-shot A/B + min-p 0.99 (3 interleaved whole-process rounds, predeclared gates)

| Round | mtp decode_ms | df2 decode_ms | mtpminp decode_ms | D ratio (mtp/df2) | M ratio (mtp/mtpminp) |
|---|---|---|---|---|---|
| 1 | 3946.6 | 4140.2 | 4614.3 | 0.953 | 0.855 |
| 2 | 3962.7 | 3683.0 | 4329.9 | 1.076 | 0.915 |
| 3 | 3887.2 | 4708.4 | 4613.0 | 0.826 | 0.843 |
| **median** | **3946.6** | **4140.2** | **4613.0** | **0.9532** | **0.8552** |

Acceptance/tokens-per-round: mtp 0.80–0.82, 3.15–3.22 tok/round; df2 0.86–0.93, 2.84–3.25;
mtpminp 2.08–2.59. df2's window-size histogram variance across rounds *is* its round noise.

**Verdicts:** Gate D **WASH/underpowered** (CI [0.826, 1.076] straddles 1.00) — DFlash2 closed
as no-win. Gate M **supported negative** (every round < 0.92) — chains earn ~+17% here; min-p
stays 0.70.

### Historical drafter knob A/Bs (for the comparison shape)

| Knob / arm | Setting tried | Outcome |
|---|---|---|
| `--spec 3` | deeper draft window | **+9.5%** (adopted) |
| `--spec 5` + SH_STREAM | deeper + stream | **+11.3%** @1K (adopted) |
| `--spec 6` | raw depth beyond 5 | −6.0% @128K |
| ROUTE_TAIL_SKIP=7 | skip dead experts in verify | **+14–19%** (adopted; biggest decode win) |
| `--lookup-chain 2` | wider lookup drafts | −5.6% before TAIL_SKIP → **+6.3/+19.9%** after (adopted) |
| MTP_TOP2 / mtp-hnorm / mtp-window | drafter tweaks | all rejected (acceptance down) |
| Custom 41K draft vocab | cheaper draft head | −1.37% net |
| STRATA_SPEC_COST | cost-aware draft table | −13.45% @128K |
| YaRN 512K | long-context serving | −16.7% decode |
| VERIFY_DEFER_CPU | defer CPU rows | −3.6 to −11.4% |

(Adopted rows repeated here only to show the rejected/adopted boundary; see [../adopted.md](../adopted.md).)
