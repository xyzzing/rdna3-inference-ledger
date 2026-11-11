# Rejected — memory / system

Context: 24 GiB VRAM, a 63+ GiB expert pack, a host running the engine. Most of these closes
are arithmetic or a single decisive measurement, which is exactly why they were cheap.

| Experiment | Plain description | Result |
|---|---|---|
| All-VRAM KV at ≥64K | Keep the whole KV cache on card | Engine exits — dense weights starve; KV streaming (`--kv int8 --kv-resident`) is required at depth |
| kv-resident cliff | What happens past the resident window | Prompt read throughput 468→**98 tok/s** — know where your cliff is before sizing the window |
| Experts on slow `/mnt` volume | mmap the expert pack off hard storage | 5.6–8 vs 25–33 tok/s → **rule: the pack must live on NVMe** |
| 24 GB full-expert residency | Fit all experts on card | Dead arithmetically — even the smaller-format pack is 63 GiB. Ceiling-check beats build |
| Driver memory leak (found Sep 30) | A 123-engine benchmark campaign leaked 36–40 GB of kernel-pinned memory | Everything ran **2.4× slower** and looked like a code regression; only a reboot fixed it. **A campaign-wide slowdown with no code change is a host/driver smell — check it before profiling the kernel** |
| vm.swappiness 60 + zram | OOM valve proposal | Parked — zram held only 50–230 MB at kill time; the cgroup MemoryHigh valve ([adopted.md](../adopted.md)) was the fix that worked |

## The one adopted idea this file shadows

File-backed expert mmap replaced 47.9 GiB of pinned RAM (three OOM kills before it), trading
cold-decode 14.3→7.6 tok/s for stability, then the cgroup valve made the trade survivable.
The memory story (pinned → mmap → cgroup) is one budget with three fixes — see
[../adopted.md](../adopted.md).
