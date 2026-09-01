# Field notes: 32 GB Blackwell lane

The math below is derived from upstream's published per-token costs.

## Memory math at a 30 GB budget (32 GB card minus headroom)

Upstream's costs: target weights 14.2 GiB, MTP head ~0.05 GiB, DFlash2
draft weights 1.4 GiB; KV ~18 KB/token at nvfp4 (+~1.2 KB/token MTP head,
+~5 KB/token DFlash2 draft KV; only the 16 full-attention layers hold KV;
fp16 is ~64 KB/token).

- DRAFT=dflash2 + nvfp4, 262,144 context: 15.6 + 262,144 x 23 KB
  = ~21.6 GiB. Fits with ~8 GiB spare, which is why this build defaults
  to dflash2 where the 24 GB recipe defaults to mtp.
- fp16 KV at 262,144: 15.6 + 16.8 = ~32.4 GiB. Does not fit; KV quant is
  required for full native context even at 32 GB.
- The YaRN 1M config: ~19 GiB of nvfp4 KV + weights = ~34.6 GiB. Does not
  fit on one card, same conclusion as upstream's 24 GB notes.

## Operational notes carried from upstream

- The server is batch-1: one generation at a time, requests queue. Do not
  put multi-agent traffic on it expecting concurrency.
- `CPU_CACHE_GB` is accepted but inert (the spill tier is planned, not
  implemented).
- Context past 262,144 needs the YaRN config variant; the launcher swaps
  it in automatically when `CONTEXT_SIZE` asks for more.
- First run compiles CUDA kernels for sm120 (5 to 20 minutes) and
  downloads ~15.6 GB of weights.
