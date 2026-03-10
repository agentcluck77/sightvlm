# llama.cpp Multimodal CLI

`llama-llava-cli` is deprecated. Use `llama-mtmd-cli` (unified multimodal CLI).

## Build

```bash
sudo apt install -y cmake build-essential libcurl4-openssl-dev
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
cmake -B build -DCMAKE_BUILD_TYPE=Release \
  -DGGML_NATIVE=ON -DGGML_NEON=ON -DGGML_OPENMP=ON
cmake --build build -j4
sudo cmake --install build   # puts binaries in /usr/local/bin
# or add to PATH:
echo 'export PATH="$HOME/batglass/llama.cpp/build/bin:$PATH"' >> ~/.bashrc
```

## Usage

VLMs need two GGUF files: **text model** + **mmproj** (vision encoder).

### Single-turn (one image, one prompt)

```bash
llama-mtmd-cli \
  -m <model.gguf> \
  --mmproj <mmproj.gguf> \
  --image /path/to/image.jpg \
  -p "Describe this image." \
  -t 4 -c 2048 --temp 0.1
```

### Interactive mode (multiple images per session)

```bash
llama-mtmd-cli -m <model.gguf> --mmproj <mmproj.gguf> -t 4
# at the > prompt:
# /image /path/to/image.jpg
# then type your prompt
# /clear  — reset context
# /quit   — exit
```

### Auto-download from HuggingFace

```bash
llama-mtmd-cli -hf <user/repo> --image /path/to/image.jpg -p "..."
# -hf auto-downloads both model and mmproj
```

## Key flags

| Flag | Description |
|---|---|
| `-m` | Text model GGUF |
| `--mmproj` | Vision projector GGUF |
| `-hf user/repo` | Auto-download both files from HuggingFace |
| `--image <path>` | Image file (repeatable for multiple images) |
| `-p <prompt>` | Prompt text |
| `-t 4` | Threads (use 4 on Pi 5) |
| `-c 512` | Context size (keep small on Pi 5) |
| `--temp 0.1` | Temperature (low = more deterministic) |
| `--chat-template <name>` | Override chat template (e.g. `vicuna`) |
| `--cache-type-k q8_0` | Quantize KV cache to save RAM |

## Benchmarking t/s

```bash
llama-bench -m <model.gguf> -t 4 -p 0 -n 50
# tg = token generation t/s (the number that matters)
```
