# Moondream2

Small VLM (1.8B params) optimised for speed on CPU. Best choice for always-on scene
description on Pi 5. OCR is adequate for signs and printed text, not documents.

- TextVQA: 76.3% | DocVQA: 79.3 | OCRBench: 612
- Est. ~20–25 t/s on Pi 5 at f16 (no Q4 available yet)

## Quick Start (Moondream station)
https://docs.moondream.ai/station

## Download

```bash
# ggml-org repo — recommended (has vicuna template embedded)
hf download ggml-org/moondream2-20250414-GGUF \
  moondream2-text-model-f16_ct-vicuna.gguf \
  moondream2-mmproj-f16-20250414.gguf \
  --local-dir ./models/moondream2

# OR official moondream org (clean filenames, same files)
hf download moondream/moondream2-gguf \
  moondream2-text-model-f16.gguf \
  moondream2-mmproj-f16.gguf \
  --local-dir ./models/moondream2
```

## Run

```bash
# Auto-download shortcut (simplest)
llama-mtmd-cli -hf ggml-org/moondream2-20250414-GGUF \
  --image /tmp/test.jpg \
  -p "Describe this image." \
  -t 4 --temp 0.1

# With local files
llama-mtmd-cli \
  -m ./models/moondream2/moondream2-text-model-f16_ct-vicuna.gguf \
  --mmproj ./models/moondream2/moondream2-mmproj-f16-20250414.gguf \
  --chat-template vicuna \
  --image /tmp/test.jpg \
  -p "Describe this image." \
  -t 4 --temp 0.1
```

> **Note:** `--chat-template vicuna` is required. Moondream2 does not embed a chat
> template in its GGUF. The `ggml-org` repo patches this in; other sources need the flag.

## Useful prompts

```
Describe this image.
Read any text visible in this image.
Describe this scene as if explaining it to someone who cannot see.
What objects are in this image?
```

## TTS pipeline

```bash
llama-mtmd-cli -hf ggml-org/moondream2-20250414-GGUF \
  --image /tmp/test.jpg \
  -p "Describe this scene for a visually impaired person." \
  -t 4 --temp 0.1 --no-display-prompt -n 150 \
  | piper --model en_US-lessac-medium --output-raw \
  | aplay -r 22050 -f S16_LE -t raw -
```
