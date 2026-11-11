# Rejected — the negative corpus

Every entry is a *measured* negative, not a guess or a judgment call. Each closed under the
[protocol](../methodology.md) with a predeclared gate. Roughly 80 named experiments across four
files (the full era produced ~300 verdict records; these tables carry the named, numbered ones):

| File | Domain | Headline |
|---|---|---|
| [drafter.md](drafter.md) | Speculative decoding: drafters, chain depth, gates, trees | DFlash2 wash; Sequoia 0/10 rescue; entropy gate has no operating point; chain depth is not the lever |
| [kernels-layouts-quantization.md](kernels-layouts-quantization.md) | Kernel variants, data layouts, quant formats | NVIDIA's layouts/quant transfers mostly reverse on RDNA3; kernel wins don't survive the verify window |
| [memory-system.md](memory-system.md) | KV placement, residency, storage, OS | Arithmetic kills all-VRAM residency; pack must live on NVMe; a driver leak silently cost 2.4× |
| [decode-next-campaign.md](decode-next-campaign.md) | The final 12-experiment campaign (2026-11-10/11) | Zero promotions; every track closed honestly; session reuse already works (13× turn latency on resume) |

## Why publish negatives

The porting-era pattern that motivated this repo: ideas that are consensus-good on NVIDIA
repeatedly measured *negative* on RDNA3 — the upstream Q8_0 multi-column layout (−17 to −22%),
the k8v4 KV format (−24% prefill), WY-form parallel GDN recurrence (+40% slower prefill).
Anyone optimizing for RDNA3 without paired measurements is likely shipping regressions with
confident commit messages. Copy any of these tables; re-run anything on your own card — the
[protocol](../methodology.md) is short on purpose.
