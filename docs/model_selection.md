# VLM Model Selection — Raspberry Pi 5

**Goal:** OCR + scene description for audio output, 15+ t/s, llama.cpp, CPU-only.

## Hardware constraints

- Cortex-A76 × 4 @ 2.4 GHz, 8 GB LPDDR4X
- Real memory bandwidth: ~15–20 GB/s sustained
- Token generation is bandwidth-bound → model weight size determines speed
- Active cooling required (throttles to 1.8 GHz without it)

## OCR benchmarks

| Model | Params | TextVQA | DocVQA | OCRBench | Est. t/s (Pi 5, Q4_K_M) |
|---|---|---|---|---|---|
| Qwen2-VL-2B | 2B | 79.7% | 90.1 | 794 | 10–15 ⚠️ |
| InternVL2-2B | 2.2B | 73.4% | 86.9 | 784 | 12–18 |
| moondream2 | 1.8B | 76.3% | 79.3 | 612 | 20–25 ✅ |
| MiniCPM-V 2.0 | 2.4B | 74.1% | 71.9 | — | 12–18 |
| SmolVLM-2B | 2B | 72.7% | 81.6 | — | 15–20 |
| SmolVLM-500M | 0.5B | weak | weak | — | 40–60 ✅ |
| MiniCPM-V 2.6 | 8B | ~74% | ~84 | 852 | 4–7 ❌ |

## Recommendation

**Start with Qwen2-VL-2B Q4_K_M.** Best OCR at this scale; may clear 15 t/s — benchmark first.
If it falls short, use the two-model split below.

| Use case | Model | GGUF |
|---|---|---|
| Scene description (always-on) | moondream2 Q4_K_M | `ggml-org/moondream2-GGUF` |
| OCR (on demand) | Qwen2-VL-2B Q4_K_M | `bartowski/Qwen2-VL-2B-Instruct-GGUF` (~986 MB) |
| Speed only, weak OCR | SmolVLM-500M Q4_K_M | `HuggingFaceTB/SmolVLM-500M-Instruct-GGUF` |
| Native TTS pipeline (slow) | MiniCPM-o 2.6 Q4 | `openbmb/MiniCPM-o-2_6-gguf` (4.46 GB) |

## llama.cpp build

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release \
  -DGGML_NATIVE=ON -DGGML_NEON=ON -DGGML_OPENMP=ON
cmake --build build -j4
```

Run:
```bash
./llama-cli -t 4 -c 512 --cache-type-k q8_0 \
  --mmproj <vision_encoder.gguf> -m <model.gguf> \
  --image <photo.jpg> -p "Describe this image."
```

- `-t 4` (try `-t 3` if throttling)
- `-c 512` to limit KV cache RAM
- Reduce `min_pixels` on Qwen2-VL to cut image tokens and gain speed

## TTS

Pair with **Piper TTS** (`en_US-lessac-medium`) — ARM-optimized, low latency, offline.

Pipeline: `camera → llama.cpp VLM → text → piper → audio`
