# Qwen3.8-27B DFlash2 EXL3 on consumer Blackwell

Blackwell-only (sm120, RTX 5090 class) build of the Qwen3.8-27B EXL3
deployment kit by [Mia'a AI Lab](https://x.com/MiaAI_lab): the lab's own
[exllamav3 fork](https://github.com/MiaAI-Lab/exllamav3) (DFlash2/MTP
drafting, NVFP4/FP8 KV cache), EXL3 quants and server. This repo strips
the kit to the discrete-Blackwell lane and pins the build to sm120;
everything that makes it work is Mia'a AI Lab's engineering. Upstream:
[MiaAI-Lab/Qwen3.8-27B-DFlash2-EXL3-5.0bpw](https://github.com/MiaAI-Lab/Qwen3.8-27B-DFlash2-EXL3-5.0bpw), MIT.

## Measured on one RTX 5090

Defaults as shipped (DFlash2 draft + nvfp4 KV, 262,144 context), 25.4 GiB
resident, numbers from the server's own `[stats]` line, 2026-09-01:

| metric | result |
|---|---|
| decode, 500-token code generations | **193 tok/s** (decode-only) |
| same runs, wall-clock incl. prefill | 132-161 tok/s |
| draft acceptance (DFlash2) | 0.70 |
| prefill, cache-cold 16K prompt | **2,244 tok/s** |
| prefill @48K / @96K | 1,671 / 1,191 tok/s (~81 s TTFT at 96K) |
| first-run bootstrap (compile + weights) | under 25 minutes, unattended |

## Contents

- `launch.sh`: self-bootstrapping launcher (venv, sm120 engine compile,
  weight download from HF, serve); upstream's `start.sh` pinned to
  Blackwell
- `stop.sh`: graceful shutdown
- `tools/serve_openai.py`: Mia'a AI Lab's OpenAI-compatible server (chat
  completions, streaming, tool calling), plus one addition here: a
  per-request `[stats]` log line (cached tokens, prefill time, decode
  tok/s, draft acceptance)
- `.env.example`: documented knobs, defaults set for a 32 GB card
  (DRAFT=dflash2, CACHE_QUANT=nvfp4, 262,144 context)
- `NOTES.md`: the 32 GB memory math and operational notes

## Quick start

```bash
cp .env.example .env      # edit if the defaults do not fit
./launch.sh               # first run: builds .venv + compiles the engine
                          # for sm120, downloads weights, serves
```

Serving on `http://localhost:8888/v1`. The server generates one request
at a time (batch-1 speculative decoding); concurrent requests queue. The
weight repos may need a HF token (`HF_TOKEN` in `.env` or `hf auth login`).

## Weights (Mia'a AI Lab's, on Hugging Face)

- [`Mia-AiLab/Qwen3.8-27B-EXL3-3.5bpw`](https://huggingface.co/Mia-AiLab/Qwen3.8-27B-EXL3-3.5bpw):
  target, workload-calibrated, 14.2 GB
- [`Mia-AiLab/Qwen3.8-27B-DFlash2-EXL3-5.0bpw`](https://huggingface.co/Mia-AiLab/Qwen3.8-27B-DFlash2-EXL3-5.0bpw):
  DFlash2 draft, 1.4 GB, +33% decode throughput at acceptance parity
