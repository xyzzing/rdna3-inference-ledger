# Lessons — toolchain (HIP / amdclang / WMMA / data layout)

Gotchas that cost real debugging time, each verified by a reproduction or a bitwise test. If you
write device code on ROCm/RDNA3, read this before trusting a parity result.

## Compiler

- **amdclang contracts explicitly-rounded intrinsics into FMA** ([ROCm/llvm-project#5330](https://github.com/ROCm/llvm-project/issues/5330), open).
  `__fadd_rn(__fmul_rn(a,c),b)` compiles as an FMA whose exact value sits off the round-to-nearest
  midpoint, flipping tie cases (measured a byte flip on a fixed input; `-ffp-contract=off`
  restores host agreement). **Workaround: volatile barriers between explicitly-rounded ops** in
  any bitwise device code. Related genus: LLVM 24 VOPD pairing nondeterminism observed
  independently by the [Radiance](https://codeberg.org/StillDeadcode/radiance) project.
  Audit rule: inventory explicitly-rounded patterns repo-wide before trusting bitwise claims.
- **`__gfx1100__` is device-pass-only.** Host-side dispatch must gate on `hip_wmma_arch` (or
  equivalent runtime query). A host `#ifdef __gfx1100__` silently selects nothing.

## HIP runtime facts (measured, version-pinned: HIP 7.17-era nightly)

- **Conditional graph nodes are absent.** No conditional nodes, no device graph launch, no
  queue/doorbell bypass — design around one captured graph per window with all nodes static.
- **Captured graph args freeze pointers at capture time.** Re-capture or stage writes through
  stable slots; pinned-memcpy nodes re-read their source per launch (which is the one dynamic
  input path).
- **Never spin an unbounded persistent kernel waiting for an unscheduled producer**, and don't
  assume CUDA's grid-wide barrier semantics port. Measured: in-graph flag-polling costs
  ~1.8 ms/window (~5% of a decode burst) and legacy per-step host spins already pipeline
  against device execution — the "obvious" graph optimizations (launch-ahead, self-commit)
  measured neutral-to-negative or parity-breaking.
- **`.cu` compiled as CUDA-mode C++ with the HIP compat header does NOT define `__HIPCC__`.**
  Gate HIP-only sections on `__HIP_PLATFORM_AMD__` / a build define — and note that
  `!defined(X)` with `X` defined-to-0 is always false; test the value (`!X`), not the definedness.

## Data layout traps

- **This ggml's `block_q8_1` is `{fp16 d @0; fp16 s @2; int8 qs[32] @4}` — the REVERSE of the
  classic layout** (classic is `{d; qs}`). `Q8_0` *is* classic `{d; qs}`, correct as expected.
  The failure mode that made this expensive: the kernel and the reference implementation shared
  the same wrong offsets and **agreed with each other while both wrong**. Parity must anchor to
  an independent reference (host-produced golden bytes), never to two implementations agreeing.
- **Verify operand sign/unsignedness per WMMA fragment form.** iu8 WMMA at full s8 on both
  operands works in the natural fragment form on gfx1100; a deployed kernel's unsigned-code-pack
  form is for its specific pack, not universally.
- **A launch-signature bug class worth knowing**: passing a row-tile count as the BLOCK dim gives
  4-thread blocks → only lanes 0–3 compute; signature is "rows 0–3 finite, odd columns
  zero-paged, row 4+ garbage". Read the launch site before debugging the kernel body.

## RDNA3 ≠ NVIDIA (transfer failures with numbers)

| Transferred "good idea" | Measured result |
|---|---|
| Upstream Q8_0 multi-column layout | **−17 to −22%** (see [rejected/kernels-layouts-quantization.md](../rejected/kernels-layouts-quantization.md)) |
| WY-form parallel GDN recurrence (WMMA folds) | prefill **+40% slower** |
| k8v4 KV format (22.7% fewer bytes) | **−24% prefill** |
| CUDA-only dispatch refinements (e.g. PcieAuto) | the *one* clean transfer win — but only after removing an unmeasured guard, then maintainer-ported |

## Provenance discipline

- **Build vintage changes numbers**: worktree builds measured **11–12% slower** than clean repo
  builds. Every published number should carry its build provenance; never compare across vintages.
- **Type-verify tensors from the manifest before pricing kernel work**: a whole gap analysis was
  voided because it priced Q8_0 kernels for tensors that are actually Q6_K/Q5_K/Q4_K per layer.
- **Env-gated branch ladders are baseline selectors.** Pin the production dispatch configuration
  (which branches are live) before choosing what you're measuring against — one 1.39× "win" was
  real but measured against a path production never runs (the production branch was ~4.5× faster).
