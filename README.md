# Qwen3.8-27B DFlash2 EXL3 on consumer Blackwell

Blackwell-only (sm120, RTX 5090 class) build of the Qwen3.8-27B EXL3
deployment kit by [Mia'a AI Lab](https://x.com/MiaAI_lab): her
[exllamav3 fork](https://github.com/MiaAI-Lab/exllamav3) (DFlash2/MTP
drafting, NVFP4/FP8 KV cache), her EXL3 quants, her server. This repo
strips the kit to the discrete-Blackwell lane and pins the build to sm120;
everything that makes it work is her engineering. Upstream:
[MiaAI-Lab/Qwen3.8-27B-DFlash2-EXL3-5.0bpw](https://github.com/MiaAI-Lab/Qwen3.8-27B-DFlash2-EXL3-5.0bpw), MIT.

## Contents

- `launch.sh`: self-bootstrapping launcher (venv, sm120 engine compile,
  weight download from HF, serve); upstream's `start.sh` pinned to
  Blackwell
- `stop.sh`: graceful shutdown
- `tools/serve_openai.py`: her OpenAI-compatible server (chat completions,
  streaming, tool calling), unmodified
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

## Weights (hers, on Hugging Face)

- [`Mia-AiLab/Qwen3.8-27B-EXL3-3.5bpw`](https://huggingface.co/Mia-AiLab/Qwen3.8-27B-EXL3-3.5bpw):
  target, workload-calibrated, 14.2 GB
- [`Mia-AiLab/Qwen3.8-27B-DFlash2-EXL3-5.0bpw`](https://huggingface.co/Mia-AiLab/Qwen3.8-27B-DFlash2-EXL3-5.0bpw):
  DFlash2 draft, 1.4 GB, +33% decode throughput at acceptance parity
