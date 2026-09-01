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

## Draft window tuning (measured 2026-09-01, one 5090)

Same two prompts per config: a greedy LRU-cache code prompt (easy) and a
temp-0.8 surreal story (hard). Decode tok/s from the stats line, with
acceptance and total drafted tokens in parentheses.

| window config | code | story |
|---|---|---|
| fixed 7 (engine default) | 168.3 (0.553, 562 drafted) | 86.1 (0.171, 1,248 drafted) |
| dynamic, skip_ema 0.3 | 68.4 (0.680) | 86.2 (0.452) |
| dynamic, skip_ema 0 (server default) | 159.9 (0.731, 417 drafted) | 87.2 (0.422, 474 drafted) |

- The engine's skip_ema 0.3 default parks fresh jobs in skip mode (probe
  every 16 rounds), so easy traffic decodes mostly undrafted: avoid.
- Fixed vs adaptive is a wash shallow, but the fixed window drafts 2.6x
  more tokens on hard output, and each wasted verification round costs
  attention over the whole context. On real 91K-deep agent traffic the
  fixed window measured 28-32 tok/s at acceptance 0.17-0.32.
